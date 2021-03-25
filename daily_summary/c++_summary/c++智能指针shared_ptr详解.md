# c++智能指针shared_ptr详解

- `shared_ptr`是c++11提供的一种智能指针类，它可以在任何地方都不使用自动删除相关指针，从而帮助彻底消除内存泄露和悬空指针的问题。

- 它遵循共享所有权的概念，即不同的`shared_ptr`对象可以与相同的指针相关联，并在内部使用引用计数机制来实现这一点。

- 每个`shared_ptr`对象在内部指向两个内存位置：

  - 指向对象的指针
  - 用于控制引用计数的指针

- 共享所有权在引用计数的帮助下工作：

  - 当新的`shared_ptr`对象与指针关联时，则在其构造函数中，将与此指针关联的引用计数增加1
  - 当任何`shared_ptr`对象超出作用域时，则在其析构函数中，将与此指针关联的引用计数减1。如果引用计数变为0，则表示没有其他`shared_ptr`对象与此内存关联，在这种情况下，它使用`delete`删除该内存。

- 使用原始指针创建`shared_ptr`对象

  ```c++
  std::shared_ptr<int> p1(new int(22));
  ```

  这行代码在堆上创建了两块内存：1：存储`int`。2：用于引用计数的内存，管理附加此内存的`shared_ptr`对象的计数，最初计数将为1。

- 使用`make_shared`创建`shared_ptr`对象

  ```c++
  std::shared_ptr<int> p1 = std::make_shared<int>();
  ```

  因为带有参数的`shared_ptr`构造函数的`explicit`类型的，所以不能像这样`std::shared_ptr<int> p1 = new int();`隐式调用构造函数。创建新的`shared_ptr`对象的最佳方法是`std::make_shared`，该语句一次性为`int`对象和用于引用计数的数据都分配了内存，而`new`操作符只为`int`分配了内存。

- 查看`shared_ptr`对象的引用计数

  ```c++
  p1.use_count();
  ```

- 分离关联的原始指针

  - 不带参数的`reset()`

    ```c++
    p1.reset();
    ```

    它将引用计数减少1，如果引用计数变为0，则删除指针。

  - 带参数的`reset()`

    ```c++
    p1.reset(new int(34));
    ```

    在这种情况下，它将在内部指向新指针，因此其内部计数将再次变为1。

  - 使用`nullptr`重置

    ```c++
    p1 = nullptr;
    ```

- 自定义删除器deleter

  当`shared_ptr`对象超出范围时，将调用其析构函数。在其析构函数中，它将引用计数减1，如果计数为0，则删除关联的原始指针。析构函数中删除内部原始指针时，默认调用的是`delete()`函数。有时候在析构函数中，`delete()`函数并不能满足我们的需求，还需要其他的额外处理。

  - 当`shared_ptr`对象指向数组

    ```c++
    std::shared_ptr<int> p3(new int[88]);
    ```

    像这样申请的数组，应该调用`delete[]`释放内存，而`shared_ptr`析构函数中默认`delete`并不能满足要求。

  - 给`shared_ptr`添加自定义删除器

    在上面这种情况下，我们可以将回调函数传递给`shared_ptr`的构造函数，该构造函数将从其析构函数中调用以进行删除，即

    ```c++
    struct Sample
    {
        Sample()
        {
            std::cout << "Sample" << std::endl;
        }
        ~Sample()
        {
            std::cout << "~Sample" << std::endl;
        }
    };
    
    void deleter(Sample * x)
    {
    	delete[] x;
    }
    std::shared_ptr<Sample> p3(new Sample[12], deleter);
    ```

  - 使用Lambda表达式/函数对象作为删除器

    ```c++
    class Deleter
    {
        public:
        void operator()(Sample *x)
        {
            delete[] x;
        }
    };
    
    //函数对象作为删除器
    std::shared_ptr<Sample> p3(new Sample[3], Deleter());
    //Lambda表达式作为删除器
    std::shared_ptr<Sample> p4(new Sample[3], [](Sample * x)
                               {
                                   delete[] x;
                               });
    ```

- `shared_ptr`相对于普通指针的优缺点

  与普通指针相比，`shared_ptr`仅提供`->` `*`和`==`运算符，没有`+`,`-`, `++`,--,`[]`运算符。

- NULL检测

  当我们创建`shared_ptr`对象而不分配任何值时，它就是空的；普通指针不分配空间的时候相当于一个野指针，指向垃圾空间，且无法判断指向的是否是有用数据。

  `shared_ptr`检测空值的方法

  ```c++
  std::shared_ptr<Sample> ptr3;
  if(!ptr3)
  {}
  if(ptr3 == NULL)
  {}
  if(ptr3 == nullptr)
  {}
  ```

- 创建`shared_ptr`注意事项

  - 不要使用同一个原始指针构造`shared_ptr`

    创建多个`shared_ptr`的正常方法是使用一个已经处在的`shared_ptr`进行重建，而不是使用同一个原始指针进行创建。

    ```c++
    int *num = new int(23);
    std::shared_ptr<int> p1(num);
    
    //正确使用方法
    std::shared_ptr<int> p2(p1);
    //不正确的使用方法
    std::shared_ptr<init> p3(num);
    std::cout << "p1 Reference = " << p1.use_count() << std::endl; // 输出 2
    std::cout << "p2 Reference = " << p2.use_count() << std::endl; // 输出 2
    std::cout << "p3 Reference = " << p3.use_count() << std::endl; // 输出 1
    ```

    假如使用原始指针`num`创建了p1，又用同样的方法创建了p3，当p1超出作用域时会调用`delete`释放`num`内存，此时`num`成了悬空指针，当p3超出作用域再次`delete`时就会出错。但是这种方法还是造成了悬空指针，后面还是要进行`delete`删除。

  - 不要还是用栈中的指针构造`share_ptr`对象

    `shared_ptr`默认的构造函数中使用的是`delete`来删除关联的指针，所以构造的时候也必须使用`new`出来的堆空间的指针。

    ``` c++
    #include<iostream>
    #include<memory>
    
    int main()
    {
        int x= 12;
        std::shared_ptr<int> ptr(&x);
        return 0;
    }
    ```

    当`shared_ptr`对象超出作用域调用析构函数`delete`指针`&x`时会出错。

  - 建议使用`make_shared`

    为了避免以上两种情况，建议使用`make_shared`创建对象。

    ```c++
    std::shared_ptr<int> ptr_1 = std::make_shared<int>();
    std::shared_ptr<int> ptr_2(ptr_1);
    ```

  - 另外不建议使用`get()`函数获取`shared_ptr`关联的原始指针，因为如果在`shared_ptr`析构之前手动调用了`delete`函数，同样会导致错误。

