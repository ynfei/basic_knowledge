# CMakeLists.txt路径包含问题

&emsp; catkin_ros包中包含工程内部的头文件，在cpp文件中include具体文件后，在CMakeLists.txt中include_directories项中include路径需要与cpp文件中的路径相呼应。

比如：

```c++
///cpp文件中
#include "rslvi_mapping/common.h"

///在CMakeLists.txt中对应
include_directories(include)
=======================================
///cpp文件中
#include "common.h"

///在CMakeLists.txt中对应
include_directories(include/rslvi_mapping/)

```



