---
title: CF IO LZX
subtitle: 
layout: post
author: "LZX"
header-style: text
tags:
  - Python
---

# 读取文件信息

## 流程简介

并行读取网格文件，流程分为如下几个步骤：

- 读取CFmesh文件头信息，根据头信息给之后读取信息分配空间

- 读取网格拓扑信息Element
- 并行分区，分区信息重分配
- 构建overlap重叠面信息
- 根据分区信息读取对应本进程的边界Face、Node坐标和State数据
- 构建face信息



从读入一个非CFmesh格式的文件，到构建出对应的单元面信息，涉及到的函数如下：

```
MeshCreator::generateMeshData
|	CFmeshReader::generateMeshDataImpl
|	|	CFmeshReader::convertFormat
|	|	|	DerivedClass::convert				--------1
|	|	ParReadCFmesh<ParCFmeshFileReader>::execute
|	|	|	FileReader::readFromFile			--------2
|	|	|	|	ParCFmeshFileReader::readString
|	|	|	|	|	ParCFmeshFileReader::readElementList
|  	|	|	|	|	ParCFmeshFileReader::readNodeList
|  	|	|	|	|	ParCFmeshFileReader::readStateList ........
|  	|	|	MeshDataBuilder::computeGeoTypeInfo		--------3
|  	|	|	FVMCC_MeshDataBuilder::createTopologicalRegionSets	-------4
|  	|	|	|	MeshDataBuilder::createInnerCells
|  	|	|	|	FVMCC_MeshDataBuilder::renumberCells
|  	|	|	|	MeshDataBuilder::setCoordInCellStates
|  	|	|	|	FVMCC_MeshDataBuilder::createCellFaces
|  	|	|	|	FVMCC_MeshDataBuilder::createInnerFaceTRS
|  	|	|	|	FVMCC_MeshDataBuilder::createBoundaryFacesTRS
|  	|	|	|	FVMCC_MeshDataBuilder::setMapGeoToTrs
```

首先由CF的自注册机制，按照CFcase注册读取模块CFmeshReader，经过接口调用generateMeshData函数进入读取流程，整个流程对应上图的函数，分别为：

1. 转换网格格式（如果需要转化的话）
2. 并行读取CFmesh文件，根据文件中的内容执行绑定的函数，读取node、state、cell和element等信息，执行并行分区，并将对应的内容发送给对应的进程
3. 计算单元类型信息（对于高阶有限元类算法，单元信息比较复杂，包括单元几何和流场变量的形函数等信息）
4. 创建TRS，构建所有的内部面、边界面并创建索引
   1. 创建所有内部单元TRS
   2. 单元重排序
   3. 根据单元类型设置单元状态变量，定义对应的几何坐标
   4. 创建单元和单元的面对接关系
   5. 创建所有内部单元面TRS，找出左右单元
   6. 根据不同的边界类型，创建对应的边界面TRS
   7. 创建几何实体与TRS间的映射关系



## 读取CFmesh文件头

CFmesh文件头包含文件版本、生成该文件的程序版本、Element类型数、每个Element类型中Element的个数、TRS的个数、TR的个数等。文件分为ASCII码明文文件和二进制文件，ASCII码文件读取对应于ParCFmeshFileReader类，二进制文件对应于ParCFmeshBinaryFileReader。

在读取文件之前会先注册函数，在setMapString2Reader函数中注册关键字对应的函数。例如将字符串 `！COOLFLUID_VERSION` 绑定到 `readCFVersion` 函数上。在函数 `readString` 中，调用`readAndTrimString` 扫描文件中的每一行关键字，然后根据关键字调用对应的绑定函数，在绑定的函数中会修改文件指针，致使下一次的扫描关键字会跳过数据部分，又到下一个关键字字段，直到读取到文件终止符 `!END` 停止。



## 读取Element信息

先根据头信息获取Element的总数，再根据环境中拥有多少个进程，按进程数分配每个进程的Element个数。这里粗分的策略只是将Element个数用进程数整除，多余的放到0号进程中，只是为了粗略的切分一下。之后按分配的Element数去读取对应的Element，对于二进制文件是直接跳转文件指针，不属于自己的部分不读；而ASCII码无法跳转，所以会扫描所有的行，但只保存属于自己的Element块。

```c++
// 在ParCFmeshFileReader::setElmDistArray函数中实现
// 为每个进程分配读取的大小
vector<int> nbElemPerRank(nbProcessors);
const int ne = totalNbElements / nbProcessors;
// 第一个进程会将不能等分的部分存下来
nbEmenPerRank[0] = nr + totalNbElements % nbProcessors;
for(int i = 1; i < nbProcessors; ++i){
    nbEmenPerRank[i] = ne;
}

// 找到每个Element类型对应的node和stat个数
ParCFmeshFileReader::setSizeElemVec();

// 在ParCFmeshFileReader::readElemListRank函数中实现
// 找到本进程应该读的部分
int firstElementID = 0;
for(int rank = 0; rank < myRank; ++rank){
    firstElementID += nbEmenPerRank[rank];
}
const int lastElementID = firstElementID + nbElemPerRank[p];

// 遍历不同的Element类型（主要为了混合网格）
	// 循环读取elements信息
		// 只存储属于自己的部分，不属于自己的部分丢弃
```

读取的Element信息都存在PartitionerData结构中，该结构也会在之后分区与发送中发挥作用，这个类存储所有进程分区所需的信息，并承担分区结果的临时存储。

```c++
struct PartitionerData
{
  typedef int IndexT;
    
  /// 网格维度
  CFint ndim;
  // 存储每个进程对应的Element起始位置的索引
  std::vector<IndexT> elmdist;
    
  // 每个进程Element中拥有的node个数
  std::vector<IndexT> sizeElemNodeVec;
  // 每个进程Element中拥有的state个数
  std::vector<IndexT> sizeElemStateVec;
    
  // 实际存储的Element中的node数据
  std::vector<IndexT> elemNode;
  // 同上，存储state
  std::vector<IndexT> elemState;
    
  // 作为elemNode的索引，存储每个node在elemNode中的起始位置
  std::vector<IndexT> eptrn;
  // 同上，存储state
  std::vector<IndexT> eptrs;
    
  // 存储下标对应的Element属于哪个进程（为什么要用指针）
  std::vector<IndexT> *part;
};
```



## 并行分区并同步信息

在ParCFmeshFileReader::readElementList()函数的doPartition()中，进入分区需要运行进程数大于1，且维度数大于1。

并行分区分为两步：

- 调用parmetis函数进行并行分区，利用上一步读取到的Element的elmdist、elemNode、eptrn进行分区，每个进程将得到一个一维数组，指明本进程存储的各个Element属于哪个进程，这个一维数组存储在PartitionerData结构的part中。例如：part = { 0、0、1、2 }，表示本进程中的0、1号Element属于0号进程，2号Elem属于1号进程，3号Elem属于2号进程，需要之后通信的时候发送给对应的进程。
- 构建全局信息表，根据分区信息将需要发送的Elem重新组织，放到一个数组中，下标对应进程号，存储的数据为应该发送给对应进程的Elem。
- 将这些分区信息分发到对应的进程
  - 通过MPI_Alltoall函数交互数据的长度，让所有进程都知道自己应得的Elem个数和长度。
  - 将本进程读取的Local Element装进一维数组，通过MPI_Alltoallv发送实际的数据到对应的进程。

发送数据用的结构是 `ElementDataArray` ，由 `_eData` 和 ` _ePtr` 构成，一个存储连续的数组，另一个存储每个Element对应在数组中的下标，这样设计主要是用来适应混合网格，用来扩展Parmetis分区所需要的数据格式。

```c++
class ElementDataArray{
public:
    // 自定义的迭代器
    class Itr;
public:
    class Itr{
    public:
        // 返回Itr对应的data值
        CFuint getEntry(CFuint i) const;
    private:
        /// eData的指针
        CFuint *_dataPtr;
        /// ePtr的指针
        CFuint *_ep;
    }
private:
	// 一维数组存储所有要发送的element块，结构为：
    // GlobalID + localID + type + node个数 + state个数 + node + state
    // 即每个Element块的大小为 5 + nodeSize + steteSize
	std::vector<CFuint> _eData;
   	// 存储单个发送块开始的位置
   	std::vector<CFuint> _ePtr;
};

```

分区信息重分配的详细步骤：

1. 重新组织分区信息。构建全局Element表，先由每个进程的ElementSize，获得本进程的起始ELement的ID，由此逐个填充全局Element表。将parmetis分区得到的信息存储到mapProcToOldElemID中，key为所属进程号，value为globalElementID，将part中的数据重写下标为map中的key，数据为所属进程。（eg：0-256为0号进程，257-596为1号进程，按下标去map中查对应的Element ID）
2. 填充临时的tmpElem（ElementDataArray类型）。按照`GlobalID + localID + type + node个数 + state个数 + node + state`的顺序填入将每个Element信息填入\_eData，并用\_ePtr标记每个Element在\_eData中的位置。【但其实发送的时候只用了\_eData，实际的\_ePtr在通信获取最新的数据后会再更新一遍，这时候更新的_ePtr没用。】
3. 发送重分配分区信息。
   1. 发送分区信息长度。将本进程中的数组的长度信息更新sendCount和sendPtrCount、sendDispl、sendPtrDispl，通过MPI_Alltoall广播给其他节点，同步后的数据存在recvCount里面。再计算出分区后个节点数（重叠区也计算在内），通信获取每个进程的Element个数，存放在m_nbElemPerProc中。
   2. 发送分区信息数据。清空pdata的数据准备之后重写，将tmpElem和tmpElemSize的数据广播出去，tmpElemSize用来重建实际需要的localElem中_ePtr信息。之后清空tmp，重建localElem。由通信获取每个进程的Element的大小，存放在sizeElemArray中。
   3. 统计每个Element块中Node和State个数的最大值。先统计本进程中Node和State的个数的最大值，在通过通信获取全局的最大值。但这些值并没有被使用，用途暂时未知。
   4. 遍历localElem，将其中非重复的Node和State的ID存储到isLocalNode和isLocalState中，且复制一遍到localNodeIDs和localStateIDs中，



## 构建overlap重叠面信息

进入ParCFmeshFileReader::buildOverlapLayers流程，构建重叠面

1. 每个进程遍历自己的 local Element，标记所有的本地node和state的ID为可更新的。通信得到所有进程中最大的node数和state数，通过MPIStructDef::buildMPIStruct构建发送node和state的ID的数据结构，这里面申请的空间大小是全局最大node数，原因是要遍历其它进程的Node表，为了防止数组越界，牺牲部分性能提升安全性。
2. 每个进程广播自己的 local Element给其它所有进程，遍历接收到的来自其它进程的数据。
3. 轮询非本进程的Element，如果该node或state也在另一个进程中存在，那么该点就是重叠点。对于第一层重叠面来说，还需要判定第一层重叠区域的归属问题。CF采用的是lower Rank策略，即两个cell共用一个点时，将该点判定给较低进程作为local，另一个较高进程标记为ghost。如果当前其它进程的Element块中至少有一个node重叠，则该Element判定为重叠区域，将其中所有本进程没有的Node标记为ghost保存下来，之后读取Node的时候也一起读取进来。如果该层不是最后一层重叠区，就暂时将所有的重叠区Element加入到本进程的 local Elem 列表中，用于下一层重叠区的判定。在所有重叠区构建完毕后会将临时加入到 local Elem 的 ghost Elem 删除。
4. 第三步是递归进行的，次数等于用户选择的重叠层数，重叠层数取决于空间离散化算法。例如，如果采用 least square reconstruction(最小二乘重建)，格心形的有限体积法需要2层重叠；但如果选择结构化网格的MUSCL extrapolation方法，则至少需要3层。
5. 创建两个list，一个用来存储ghost的node/state，另一个用来存储local的node/state，并将所有的Node标记到对应的Element上（采用multimap实现）。此外，存储着重叠Element信息的ElementDataArray会被附加存储local Element的上面去。
6. Element数据根据类型重排序，相同类型的Element被连续的存储在各自的进程中。

```c
std::vector<CFuint> m_localNodeIDs;
std::vector<CFuint> m_ghostNodeIDs;
std::vector<CFuint> m_localElemIDs;

// ghostNode的id及其所属的进程号
Common::SharedPtr<Common::CFMultiMap<CFuint, CFuint>> m_gNodeID2DonorRank; 

// globalNode的id对应的本地node的id
Common::CFMap<CFuint, CFuint> m_mapGlobToLocNodeID; 

// node id 对应的element id
Common::CFMultiMap<CFuint, CFuint> m_mapodeElemID; 
```



## 读取TRs边界信息

当重叠区已经构建好后，每个进程就可以根据这些信息去读取对应的TRs边界面信息，不需要再进行进程间通信。

读取策略为：当读取一行信息后，检查该边界Face的第一个Node存在于哪些Element中，遍历这些Element，如果有一个Element持有该Face的所有Node，即说明本边界是本进程持有的，否则说明是别的进程持有。



## 读取Node&State信息

作为读取文件最后的流程，每个进程都会去读取所有的Node和State数据，但只会根据保存分区后的local和ghost中对应的部分，剩下的部分都会被丢弃掉。



# 构建Face信息

构建Face信息分为三步：

- 构建所有的Face信息
- 区分出内部面
- 区分出边界面

需要之前分区后的Elem信息，如果是并行程序的话，这里Elem中的Node应该是读取后本进程分到的 local Node信息，之后可以通过 _mapGloablToLocalNodeID 来查找。

## 构建所有Face

构建Face信息实质上是通过一定的Elem内部排序的规律将属于Elem的面给挑出来，这些面信息是隐藏在Elem的排序规律中的。例如：一个四面体占有的NodeID为1、2、3、4，那么按照规定，第一个面为1、2、3，第二个面为1、4，2，第三个面为2，4，3，第四个面为1、3、4。但按照所有的Elem来找面，会存在很多的面都重合了，需要将这些重复的面标记出来，标记这些重复的面就是CF构建所有Face中最重要的步骤。

1. 根据本进程中Elem的总数计算总的Face数（带重复Face的），初始化mapNodeFace表（一个二维数组），列表示本进程占有的NodeID，行表示拥有该Node的依赖面。
2. 遍历所有Elem，获取当前Elem中Face的个数。遍历每个Face，获取Face中的Node个数
3. 获取当前Face中的NodeID，获取其中第一个Node的ID，获取该Node的依赖面表，挨个遍历这些依赖面，记为currentFace。
4. 遍历当前Face中除第一个Node以外的Node，挨个获取当前Node的依赖面表，遍历这些依赖面，如果这些依赖面和currentFace相同，就说明这个点和起始的Node共面。如果剩下的点都和第一个点共面，那就说明除了初始加入的依赖面外，这些点还有一个面与之重合，即这个面是一个重叠的内部面，就将这个面加入到内部面 innerFace 的表中，并将这个面标记为 faceFound。
5. 如果没有找到共同的依赖面，就说明这个面还没被加入，将该面加入到mapNodeFace依赖面表中，面的总数加一。
6. 重复上述4，5步骤，直到找完所有的face。

## 构造内部面InnerFace

上一步已经将所有的内部面都找到了，这一步主要是找到所有内部面的左右单元找到并标记。

1. 遍

