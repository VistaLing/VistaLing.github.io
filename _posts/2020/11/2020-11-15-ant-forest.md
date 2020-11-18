---
title: 用Python实现定时自动化收取蚂蚁森林能量，再也不用担心忘记收取了
subtitle: 
layout: post
author: "Python小二"
header-style: text
tags:
  - Python
  - Appium
  - 蚂蚁森林
---

## 1. 概述 

提到蚂蚁森林，大家应该都知道，你是否有因忘记收取能量而被好友收取的经历呢？

![](http://pythontalk.cn/img/2020/11/15/1.PNG)

如果你不是蚂蚁森林重度用户，被别人收取了能量可能对你来说没什么。

![](http://pythontalk.cn/img/2020/11/15/2.jpg)

但如果你是蚂蚁森林重度用户，遇到能量被偷 ...

![](http://pythontalk.cn/img/2020/11/15/3.jpg)

本文我们来看一下如何使用 Python + Appium 实现定时自动化收取蚂蚁森林能量。

## 2. 环境

本文主要环境如下：

* Win7
* 小米5s
* Python3.7
* Appium1.5
* 支付宝10.2.6.7010

如果对环境搭建不熟悉的话，可以看一下：[Python + Appium 自动化操作微信入门](https://mp.weixin.qq.com/s/cMdQKerwD-UIX5Xcnb_GIw)和[我用 Python 找出了删除我微信的所有人并将他们自动化删除了](https://mp.weixin.qq.com/s/n9XD5PNIghvGM84ghqvkLA)。

## 3. 实现

功能实现的基本思路为：

* 打开支付宝进入蚂蚁森林，收取自己的能量

* 收取完自己能量后，点击`找能量`进入好友蚂蚁森林，收取好友能量，以此类推

![](http://pythontalk.cn/img/2020/11/15/4.png)

接下来我们看一下主要代码实现。

参数配置代码实现如下：

```python
desired_caps = {
    "platformName": "Android", # 系统
    "platformVersion": "8.0.0", # 系统版本号
    "deviceName": "m5s", # 设备名
    "appPackage": "com.eg.android.AlipayGphone", # 包名
    "appActivity": "AlipayLogin", # app 启动时主 Activity
    'noReset': True # 保留 session 信息，可以避免重新登录
}
```

通常大家都会将蚂蚁森林放在支付宝首页，此时我们打开支付宝后直接点击蚂蚁森林选项即可进入。

![](http://pythontalk.cn/img/2020/11/15/5.jpg)

代码实现如下：

```python
driver.find_elements_by_id('com.alipay.android.phone.openplatform:id/home_app_view')[10].click()
```

进入自己蚂蚁森林之后，开始收取自己的能量，因为新版支付宝不能定位能量球元素了，所以我们需要在能量球可能出现的区域实现点击。收取能量的代码实现如下：

```python
# 收取能量
def collect_energy(driver):
    print('开始收取能量')
    # 获取手机屏幕宽高
    width = int(driver.get_window_size()['width'])
    height = int(driver.get_window_size()['height'])
    # 能量球可能出现的区域坐标
    start_x = 110
    end_x = 940
    start_y = 460
    end_y = 880
    for i in range(start_y, end_y, 80):
        for j in range(start_x, end_x, 80):
            tap_x1 = int((int(j) / width) * width)
            tap_y1 = int((int(i) / height) * height)
            # 点击指定坐标
            driver.tap([(tap_x1, tap_y1), (tap_x1, tap_y1)], 1000)
    print('能量收取完毕')
```

自己能量收取完毕之后，点击`找能量`进入好友蚂蚁森林继续收取能量，代码实现如下：

```python
# 找能量
def search_energy(driver):
    print('找能量，收取好友能量')
    time.sleep(3)
    # 点击找能量
    driver.tap([(1000, 1520), (1080, 1580)], 1000)
    time.sleep(3)
    # 收取好友能量
    collect_energy(driver)
    time.sleep(3)
    # 收取完毕继续找能量
    search_energy(driver)
```

能量收取的功能实现了之后，我们使用定时任务实现定时收取即可，下面看一下定时任务的实现。

定时任务的实现我们使用 `apscheduler` 组件，使用之前需执行 `pip install apscheduler` 装一下。

定时任务的代码实现如下：

```python
scheduler = BlockingScheduler()
# collect_main：定时执行的方法
scheduler.add_job(collect_main, 'cron', hour=20, minute=23, second=20)
try:
    scheduler.start()
except (KeyboardInterrupt, SystemExit):
    pass
```

到此，我们利用 Python + Appium 实现定时自动化收取蚂蚁森林能量的工作就完成了。

源码在公号 **Python小二** 后台回复 **201116** 获取。