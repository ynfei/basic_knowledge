# C++智能指针unique_ptr详解

- **unique_ptr**是**c++11**提供的用于防止内存泄露的智能指针中的一种实现，独享被管理对象指针所有权的指针。unique_ptr对象包装一个原始指针，并负责其声明周期。当该对象被销毁时，会在其析构函数中删除关联的原始指针。

- `unique_ptr`具有`->`和`*`运算符，可以像普通指针一样使用。

- `unique_ptr`独享所有权。`unique_ptr`对象是关联原始指针的唯一所有者，我们无法复制`unique_ptr`对象，只能移动。

- 创建一个空的`unique_ptr`对象,因为没有与之关联的原始指针，所以它是空的。

  ```
  std::unique_ptr<int> ptr1;
  ```

- 检查`unique_ptr`是否为空

  ```c++
  //方法1
  if(!ptr1)
  {
  	std::cout << "ptr1 is empty" << std::endl;	
  }
  //方法2
  if(ptr1 == nullptr)
  {
  	std::cout << "ptr1 is empty" << std::endl;
  }
  ```

- 使用原始指针创建`unique_ptr`对象

  ```c++
  std::unique_ptr<int> int_ptr(new int(22));
  ```

- 不能通过赋值的方法创建对象，下面的这句是错误的

  ```c++
  std::unique_ptr<int> int_ptr = new int(22); //编译错误
  ```

- 使用`std::make_unique`创建`unique_ptr`对象/c++14

  ```c++
  std::unique_ptr<int> int_ptr = std::make_unique<int>(22);
  ```

- 使用`get()`获取被管理对象的指针

  ```c++
  int *p = int_ptr.get();
  ```

- 使用`reset()`重置`unique_ptr`对象

  ```c++
  int_ptr.reset();
  ```

- `unique_ptr`对象不可复制，只能移动。所以无法通过复制构造函数或赋值运算符创建unique_ptr对象的副本

  ```c++
  std::unique_ptr<int> int_ptr = int_ptr1; //编译错误
  int_ptr = int_ptr1; //编译错误
  ```

- 转移`unique_ptr`对象的所有权。无法赋值`unique_ptr`对象，但是可以转移它们。

  ```c++
  std::unique_ptr<int> int_ptr1(new int(22));
  std::unique_ptr<int> int_ptr2 = std::move(int_ptr1);
  ```

  `std::move()`将`int_ptr1`转换为一个右值引用。因此，调用`unique_ptr`的移动构造函数，并将关联的原始指针传输到`int_ptr2`。在转移完原始指针的所有权之后，`int_ptr1`将变为空。

- 使用`release()`释放其关联的原始指针的所有权，并返回原始指针。这里仅释放所有权，并没有`delete`原始指针，`reset()`会`delete`原始指针。通过`release`释放原始指针后，需要保存返回的指针，进行`delete`操作，否则会造成内存泄露。

  ```c++
  std::unique_ptr<int> int_ptr3(new int(22));
  int *p = int_ptr3.release();
  ```

  

