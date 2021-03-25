# C++ STL stack queue priority_queue

在C++的STL库中，stack，queue, priority_queue底层用的是deque来实现的。

这样操作的主要原因是：deque相比于vector,当数据量增加是，内存分配不够时需要另外开辟新的内存拷贝数据，消耗较大，而相对于list而言，其内存的利用率更高，list会导致更多的内存碎片