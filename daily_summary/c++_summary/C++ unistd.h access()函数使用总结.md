# C++ <unistd.h> access()函数使用总结

## 功能描述

`acess(const char* pathname, int mode)`:检查调用进程是否可以对指定的文件进行某种操作

参数：

- **pathname**:需要测试的文件路径名
- **mode**:需要测试的操作模式，F_OK(文件存在)，R_OK(可读)，W_OK(可写)，X_OK(可执行)，操作模式可以有一种或多种。
- 返回说明：
  - EINVAL： 模式值无效  
  - EACCES： 文件或路径名中包含的目录不可访问 
  -  ELOOP ： 解释路径名过程中存在太多的符号连接 
  -  ENAMETOOLONG：路径名太长 
  -  ENOENT： 路径名中的目录不存在或是无效的符号连接 
  -  ENOTDIR： 路径名中当作目录的组件并非目录 
  -  EROFS： 文件系统只读 
  -  EFAULT： 路径名指向可访问的空间外 
  -  EIO： 输入输出错误 
  -  ENOMEM： 不能获取足够的内核内存 
  -  ETXTBSY：对程序写入出错 
  - 0：成功
  - -1：失败

## 例

```
//判断是否存在
if (access(file_name.c_str(), F_OK) == 0)
//判断是否存在，可读，可写
if (access(file_name.c_str(), F_OK | R_OK | W_OK) == 0)
```

