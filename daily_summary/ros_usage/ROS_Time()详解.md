# ROS：：Time()详解

[TOC]

介绍ROS中的时刻，时间间隔，定时器的定义和应用

## 一、 Time

### 1、 时间和间隔

ROS中有`time`和`duration`两种类型，相应的有`ros::Time()`和`ros::Duration()`类。

- time表示的是时刻
- duration表示的是时间间隔

其表示形式为：

```c++
int32 sec
int32 nsec
```

ROS可以给节点提供一个模拟时钟。不同于平台时间，你可以利用`roscpp`的时间例程来得到当前的时间，此事件能够和模拟时间，wall-clock时间进行无缝连接。

#### 1.1 获得当前时间

```c++
ros::Time::now()
```

**时间计算的起点**

使用模拟时间时，当`/clock`节点接收到第一条消息时，`now()`返回时刻`0`，此时客户端还不知道时钟时间。

#### 1.2 创建时间和间隔

浮点数形式

```c++
ros::Time time_moment(0.0001);
ros::Duration time_interval(5.0);
```

用两个整数表示

```c++
ros::Time time_moment(0, 1000000);
ros::Duration time_interval(5 ,1000);
```

#### 1.3 时间与间隔的转化

```c++
double secs = ros::Time::now().toSec();
ros::Duration d(0.5);
secs  = d.toSec();
```

#### 1.4 时间与间隔的四则运算

```c++
ros::Duration two_hours = ros::Duration(60 * 60) + ros::Duration(60 * 60);
ros::Duration one_hour = ros::Duration(2 * 60 * 60) - ros::Duration(60 * 60);
ros::Time tomorrow = ros::Time::now() + ros::Duration(24 * 60 * 60);
ros::Duration negative_one_day = ros::Time::now() - tomorrow;
```

### 2、休眠和频率

休眠0.5s

```c++
ros::Duration(0.5).sleep();
```

频率10Hz:

```c++
ros::Rate r(10);
while(ros::ok())
{
    ...
    r.sleep();
}
```

`Rate`和`Timer`的作用一样，最好用`Timer`来定时。

### 3、Wall Time

在模拟时，如果想要进入实际运行wall-clock time,可以用`ros::WallTime()`,`ros::WallDuration`, `ros::WallRate`,类似于`ros::Time`, `ros::Duration`,`ros::Rate`。

##  二、Timer

#### 2.1 、定义定时器

定时器不能代替实时线程/内核，它们仅对没有硬实时要求的事物有用。

方法： `ros:Nodehandle::createTimer()`

```c++
ros::Timer timer = nh.createTimer(ros::Duration(0.1), timerCallback);
```

完整定义：

```c++
ros::Timer ros::NodeHandle::creaTimer(ros::Duration period, callBack, bool oneshot = false);
```

`period`:定时器回调函数之间的时间间隔

`callBack`:定时器回调函数、类方法、函数子对象

`oneshot`:是否只定时一次。`false`是连续定时。

#### 2.2、回调特征

```c++
void callBack(const ros::TimerEvent&);

struct TimerEvent
{
    Time last_expected; //上一回调函数应该发生的时刻
    Time last_real; //上一回调函数实际发生的时刻
    
    Time current_expected; //当前回调函数应该发生的时刻
    Time current_real;//当前回调函数实际发生的时刻
    
    struct
    {
        WallDuration last_duration; //包含上一回调的时间间隔（开始时间-结束时间），在wall-                                         clock time 中
    }
}
```

#### 2.3、回调类型

- Functions

  ```c++
  void callBack(const ros::TimerEvent& event)
  {
      ...
  }
  ros::Timer timer = nh.createTimer(ros::Duration(0.1), callBack);
  ```

- Class Methods

  ```c++
  void Foo::callBack(const ros::TimerEvent& event)
  {
      ...
  }
  Foo foo_object;
  ros::Timer timer = nh.createTimer(ros::Duration(0.1), &Foo::callBack, &foo_object);
  ```

- Functor Objects

  ```c++
  class Foo
  {
      public:
      	void operator()(const ros::TimerEvent& event)
          {
              ...
          }
  };
  ros:Timer timer = nh.createTimer(ros::Duration(0.1), Foo());
  ```

- Wall-clock Timer

  ```c++
  void callBack(const ros::WallTimerEvent& event)
  {
      ...
  }
  ros::WallTimer timer = nh.createWallTimer(ros::WallDuration(0.1), callBack);
  ```

## 三、 Time() vs WallTime()

`ros::Time()`指的是ROS网络中的时间。ROS网络中的时间是指，如果当时在非仿真环境离运行，那它就是当前的时间。但是假设去回放当时的数据，就需要把当时的时间记录下来。Wall Time可以理解为墙上时间，墙上时间没有人可以改变，永远往前走；Time（）可以被认为修改，可以暂停，可以加速，减速，但是Wall Time不可以。

在开启一个Node之前，当把`use_sim_time`设置为`true`时，节点会从`clock topic`获得时间，所以操作这个`clock`的发布者，可以实现一个让Node中得到ROS Time暂停、加速、减速的效果。

### 3.1 三种时间的定义

- 时钟时间：也就是wall clock time,进程从开始运行到结束，时钟走过的时间，这其中包含了进程在阻塞和等待状态的时间。
- 用户CPU时间：就是用户的进程获得CPU资源以后，在用户态执行的时间。
- 系统CPU时间：用户进程获得了CPU资源后,在内核态的执行时间。

### 3.2 三者之间的联系

进程的状态分为阻塞、就绪、运行。

时钟时间 = 阻塞时间 + 就绪时间 +运行时间

用户CPU时间 = 运行状态下用户空间的时间

系统CPU时间 = 运行状态下系统空间的时间

运行时间 = 用户CPU时间 +系统CPU时间

- CPU时间也称为进程时间，用以度量进程使用的中央处理器资源。进程时间以时钟嘀嗒计算，实际时间（Real）,用户CPU时间（User）,系统CPU时间（Sys）。
- 实际时间指的是实际流逝的时间；用户时间和系统时间指特定进程使用的CPU时间。
- Real time是从进程开始执行到完成所经理的墙上时钟时间，包括其他进程使用的时间片和本进程耗费在阻塞上的时间。
- User time是进程执行用户代码（内核外）耗费的CPU时间，仅统计该进程执行时实际使用的CPU时间，而不计入其他进程使用的时间片和本进程阻塞的时间。
- Sys time是该进程在内核态运行所耗费的CPU时间，即内核执行系统调用所使用的CPU时间。

CPU总时间（User+Sys）是CPU执行用户进程操作和内核系统调用所耗时间的总和，即该进程所使用的实际CPU时间。若程序循环遍历数组，则增加用户CPU时间，若程序执行`exec`或`fork`等系统调用，则增加系统CPU时间。

在多核处理器上，如进程含有多个线程或通过`fork`调用创建子进程，则时间时间可能小于CPU总时间，因为不同线程或进程可并行执行，但其时间会计入主进程的CPU总时间。若程序在某段时间处于等待状态而并未执行，则实际时间可能大于CPU总时间：

- real time < CPU time 表明进程为计算密集型（CPU bound），利用多核处理其的并行执行优势
- real time $\approx$ CPU time 表明进程为计算密集型，未并行执行
- real time > CPU time 表明进程为`I/O`密集型（I/O bound），多核并行执行优势不明显

