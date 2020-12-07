---
title: 用Python爬取了三大相亲软件评论区，结果...
subtitle: 
layout: post
author: "Python小二"
header-style: text
tags:
  - Python
  - Python爬虫
  - 相亲软件
---

> 小三：怎么了小二？一副愁眉苦脸的样子。

![](https://p6-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/ede853b7a8304be0aa9f2d7a31419cf2~tplv-k3u1fbpfcp-watermark.image)

> 小二：唉！这不是快过年了吗，家里又催相亲了 ...

![](https://p9-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/2c8d3fd6eaad43ea88394dbda3cdfbea~tplv-k3u1fbpfcp-watermark.image)

> 小三：现在不是流行网恋吗，你可以试试相亲软件呀。

![](https://p6-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/5cbeebf091284ab78cb82c4df474f075~tplv-k3u1fbpfcp-watermark.image)

> 小二：这玩意靠谱吗？

![](https://p1-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/1e65098a59c7439ea9c7e70f170b29e5~tplv-k3u1fbpfcp-watermark.image)

> 小三：我也没用过，你自己看看软件评论区吧。

> 小二：这 ... 不过也只能先到评论区看看了。

![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/503e489a19c74bdcb92595d53a8d7ef5~tplv-k3u1fbpfcp-watermark.image)

本文以 360 手机助手为例，地址为：`http://zhushou.360.cn/`，相亲软件选择 3 个比较流行的，分别为：世纪佳缘、百合婚恋、有缘网，我们使用 Python 爬取软件评论区，看看用户评价情况。
先来看一下这三款软件的下载量和好中差评占比情况（下图单位为万次）。

![](https://p9-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/763f5dd1a70d45fca4e017394b6bf5be~tplv-k3u1fbpfcp-watermark.image)

![](https://p1-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/6af716dc558743e895bf08adc3a3ce80~tplv-k3u1fbpfcp-watermark.image)

![](https://p9-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/1e86a6d430db4fb0bd5c9e1b47b3e8b6~tplv-k3u1fbpfcp-watermark.image)

![](https://p9-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/8f9550abad354e498d2a3c2470ac9b20~tplv-k3u1fbpfcp-watermark.image)

下面开始爬取评论区，以世纪佳缘为例，首先，在搜索框输入世纪佳缘进行搜索，如图所示：

![](https://p6-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/c3ed59d3fa3a4121b500dc768afee2d0~tplv-k3u1fbpfcp-watermark.image)

接着，点击搜索到的软件进入其详情页，如图所示：

![](https://p6-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/77540993ae894bdc8fc0081b0ade2ad0~tplv-k3u1fbpfcp-watermark.image)

将页面向下拉就可以看到评论区了，如图所示：

![](https://p9-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/7575377785324a53b1421b548afcab67~tplv-k3u1fbpfcp-watermark.image)

此时打开开发者工具并选择`Network`项，点击查看更多评论，然后可以看到`getComments`请求，如图所示：

![](https://p6-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/540783be2c85455ebd7bba4cbdd8816d~tplv-k3u1fbpfcp-watermark.image)

通过这个请求我们就可以动态获取评论区数据了，其中参数`star`为开始的评论索引，参数`count`为每次加载的评论个数，可以通过参数`callback`、`baike`指定不同应用，爬取代码实现如下：

```python
headers = {
    "Accept": "*/*",
    "Accept-Encoding": "gzip, deflate, sdch",
    "Accept-Language": "zh-CN,zh;q=0.8",
    "Connection": "keep-alive",
    "Host": "comment.mobilem.360.cn",
    "User-Agent": "Mozilla/5.0 (Windows NT 6.1; WOW64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/57.0.2987.98 Safari/537.36 LBBROWSER"
}
def comment_spider(param, file_name):
    base_url = "http://comment.mobilem.360.cn/comment/getComments?c=message&a=getmessage&&count=50"
    start = 0
    for i in range(1, 50):
        print("第{}页".format(i))
        url = base_url + param + "&start=" + str(start)
        r = requests.get(url, headers=headers)
        data = re.findall("{\"errno\"(.*)\);}catch\(e\){}", r.text)
        # 转为 Json 格式
        jdata = json.loads("{\"errno\"" + data[0])
        for message in jdata["data"]["messages"]:
            content = message["content"]
            print(content)
            with open(file_name + ".txt", "a", encoding="utf-8") as f:
                f.write(content)
        start = start + 50
        time.sleep(2)
```

我们将爬取的评论数据存到了 txt 文件中。

接着，我们将评论数据进行词云展示，代码实现如下：

```python
with open("yy.txt", "r", encoding="utf-8") as f:
    content = f.read()
    stylecloud.gen_stylecloud(text=content, max_words=600,
                              collocations=False,
                              font_path="SIMLI.TTF",
                              icon_name="fas fa-heart",
                              size=800,
                              output_name="yy.png")
    Image(filename="yy.png")
```

最后，通过词云看一下用户对上述软件的评价情况。

世纪佳缘：

![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/8763972cbf114da2bc5fa973c78ef189~tplv-k3u1fbpfcp-watermark.image)

百合婚恋：

![](https://p1-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/7e2dbd660a304aa7bf66b0acd483cfbd~tplv-k3u1fbpfcp-watermark.image)

有缘网：

![](https://p9-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/b2b1e2eb72b84b72a46da192ccdec6a6~tplv-k3u1fbpfcp-watermark.image)

> 小二：看了有缘网的评论，我感觉自己和相亲软件无缘 ...

> 小三：...

源码在公众号 **Python小二** 后台回复 **201207** 获取。

**声明：本文不构成对上述相亲软件的任何使用建议。**