---
title: 学习笔记(12):netlogo
date: 2023-04-21 13:06:08
tags: netlogo
---

> 百科定义：Netlogo 是一个用来对自然和社会现象进行仿真的可编程建模环境。

## 软件使用

### 模型使用

下载：https://ccl.northwestern.edu/netlogo/



#### 狼羊捕食模型

打开模型

> 文件 -> 模型库 -> Sample Model -> Biology -> Wolf Sheep Predation

运行模型

1. 点击 setup 初始化

2. 点击 go 运行

观察模型和统计数据随时间变化

{% asset_img wolf_sheep.png 捕食模型 %}



<!-- more -->

### 编程模型

整个空间叫 world

world 内的角色

*  turtle: 模型上可以移动的元素 
*  patch: 元素后面背景中的每个二维网格单位
*  link: 元素之间的连接 	 
*  observer: 监控所有事物

refer to the NetLogo Programming Guide. ：https://ccl.northwestern.edu/netlogo/docs/programming.html

### 命令

命令作用

* 控制 world 内的各个角色，修改属性、新建/销毁等等
* 统计数据



observer 全局修改属性

* 修改 turtle 属性

```
ask turtles [ set color red ] ;; 修改所有turtle
ask turtle idx [ set color red ] ;; 修改某个turtle
```
* 修改 patch 属性

```
ask patches [ set pcolor green ] ;; 修改所有patche
ask patch x y [ set pcolor green ] ;; 修改某个坐标的patch
```
* 统计数据

```
count turtles ;; 对所有turtle计数
```

### 代码

代码本质上是一系列命令行的集合

#### 变量定义

* 定义全局变量，声明后要在代码中初始化

```
global [rate-limit]
```

* 定义 turtle 成员变量
```
turtles-own [energy]
```

#### 方法

```
to 方法名
	;; 命令内容
	;; 方法之间可以相互调用
end
```

* turtle 操作

```
to move-turtles
  ask turtles [
    right random 360 ;; 设置转向角度
    forward 1 ;; 向前移动 1 格
    set energy energy - 1 ;; 修改成员变量 
  ]
end
```

* 逻辑时钟操作

```
reset-ticks ;; 重置时钟
tick ;; 时钟 + 1 
```

* 清理元素

```
clear-all
```

### 组件

* 按钮

用于执行方法，点击即执行对应名称的方法代码

* 监视器

展示当前统计值

* 图 plot

用于统计数值随时间变化曲线

```
plot count turtles ;; 展示计数
plot count patches[with pcolor = green]
plot [speed] of turtle 0 ;; 展示成员变量
plot [speed] of sample-car ;; 展示成员变量
plot min[speed] of turtle 0 ;; 展示成员变量
```

* 选择器/滑块...

组件直接定义全局变量，值为组件内选取的值



## 软件架构

> Netlogo 主要由 Scala 语言编写，少量 Java
> 基于 Swing 框架编写 GUI 界面的客户端架构

主要由 GUI 和后台事件处理两部分组成

### GUI 界面

基于 Swing 框架实现

主要入口：org.nlogo.app.App

{% asset_img GUI.png 主要GUI部分 %}

GUI 页面主要收集用户行为

* 前台 Widget 组件的添加、编辑、修改
* 后台 Event 操作事件发送如点击运行命令



### 事件处理

总体事件处理流程

{% asset_img event.png 事件处理流程 %}



以点击 Button 按钮运行方法为例

时序调用

{% asset_img buttonClick.png button点击 %}

调用完成后一个新 Job 会进入 JobTread 线程的处理队列

JobTread 工作线程会不断从队列中取出 job，并执行 job

```
// org.nlogo.job.JobThread
...
override def run() {
  handling(classOf[RuntimeException]) {
    while (!dying) {
      compact(primaryJobs)
      runPrimaryJobs()
      maybeRunSecondaryJobs()
      // Note that we must synchronize on newJobsCondition before calling isEmpty, since
      // otherwise a job could get added after the empty check but before we sleep - ST 8/11/03
      newJobsCondition.synchronized {
        if (primaryJobs.isEmpty)
          ignoring(classOf[InterruptedException]) {
            // only sleep for a short time, since there may still
            // be secondary jobs that need attention - ST 8/10/03
            newJobsCondition.wait(PeriodicUpdateDelay.DelayInMilliseconds)
          } } } }
}
...
```

执行过程即取出 commond，对相关的 Agent 对象进行操作

{% asset_img run.png 线程处理 %}

