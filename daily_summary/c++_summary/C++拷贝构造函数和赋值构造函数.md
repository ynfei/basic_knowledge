# C++拷贝构造函数和赋值构造函数

- 在c++中复制控制是一个比较重要的话题，主要包括拷贝构造函数，赋值构造函数。拷贝构造函数和赋值构造函数实现功能类似，如果没有手动的实现，编译器会自动生成一个，而且这两个函数的参数也是一致的 ，是不能够改变的。

- 拷贝构造函数和赋值构造函数是一个类必不可少的部分。由此可知，如果一个类没有定义任何的东西，编译器也会帮助我们生成四个函数：

  - 构造函数
  - 拷贝构造函数
  - 赋值构造函数
  - 析构函数

  ```c++
  clasee Base
  {
  	public:
  		Base();//构造函数
  		Base(const Base&);//拷贝构造函数
  		Base& operator=(const Base&); //赋值构造函数（赋值操作符）
  		～Base(); //析构函数
  	private:
  	........
  };
  ```

- 以一个比较特殊的例子进行说明

  ```c++
  class CExample
  {
      public:
      	//构造函数
      	CExample()
          {
              ptr = NULL;
              size = 0;
          }
      	//析构函数
      	～CExample()
          {
              delete ptr;
          }
      	void init(int n)
          {
              ptr = new char[n];
              size = n;
          }
      private:
      	char* ptr;
      	int size;
  };
  ```

  这个类的主要特点是包含指向其他资源的指针。ptr指向堆中分配的一段内存空间。

- 拷贝构造函数

  ```c++
  int main(int argc, char* argv[])
  {
      CExample A;
      A.init(40);
      
      CExample B = A; //把B初始化为A的副本
      ...
  }
  ```

  `CExample B = A;`用`A`初始化`B`。其完成方式是内存拷贝，复制所有成员的值。完成后`A.ptr = B.ptr`,它们将指向同样的地方，指针虽然复制了，但是所指向的空间并没有复制，而是由两个对象共用了。这样不符合要求，对象之间不独立了，并未空间的删除带来隐患。所以需要采取必要的手段（拷贝构造函数）来避免此类情况。

  - **拷贝构造函数的格式：构造函数名（类名的引用）**提供了拷贝构造函数后的`CExample`类定义为：

    ```c++
    class CExample
    {
        public:
        	//构造函数
        	CExample()
            {
                ptr = NULL;
                size = 0;
            }
        	//拷贝构造函数
        	CExample(const CExample&);
        	//析构函数
        	～CExample()
            {
                delete ptr;
            }
        	void init(int n)
            {
                ptr = new char[n];
                size = n;
            }
        
        private:
        	char* ptr;
        	int size;
    };
    
    //拷贝构造函数的定义
    CExample::CExample(const CExample& rightsides)
    {
        size = rightsides.size;
        ptr = new char[size];
        memcpy(ptr,rightsizes.ptr,size*sizeof(char));
    }
    ```

    这样的话，执行语句`CExample B = A;`时，拷贝构造函数将被调用，已有对象别名`rightsides`传给构造函数，以用来复制。原则上，应该为所有包含动态分配成员的类都提供拷贝构造函数。

- 拷贝构造函数被调用的情况有：

  - 定义新对象，并用已有的对象初始化新对象时；即执行`CExample B = A;`时；

  - 当对象直接作为函数参数传给函数时，函数将建立对象的临时拷贝，这个拷贝过程也将调用拷贝构造函数；

    ```c++
    void Test(CExample obj)
    {
        //针对obj的操作实际上是针对复制后的临时拷贝进行的
    }
    test(obj);//对象直接作为参数，拷贝构造函数将被调用
    ```

  - 当函数中的局部对象被返回给函数调用者时，也将建立此局部对象的一个临时拷贝，拷贝构造函数也将被调用；

    ```c++
    CTest func()
    {
        CTest theTest;
        return theTest;
    }
    ```

- 赋值构造函数（赋值符的重载）

  ```c++
  int main(int argc, char* argv[])
  {
      CExample A;
      A.init(40);
      
      CExample C;
      C.init(60);
      
      //现在需要一个赋值操作，被赋值对象的原内容被清除，并用右边对象的内容填充。
      C = A;
      return 0;
  }
  ```

  上面的例子中也用到了`=`，但与`CExample B = A;`不同。语句`CExample B = A;`中的`=`在对象声明语句中，表示初始化。更多时候，这种初始化也可以用括号表示。例如`CExample B(A);`

  而本例中`=`号表示赋值操作。将对象A的内容赋值到内容C，这其中涉及到对象C原有内容的丢弃，新内容的复制。但`=`的缺省操作只是将成员变量的值相应复制。旧的值被自然丢弃。由于对象内包含指针，将造成不良后果；指针的值被丢弃了，但指针指向的内容并没有释放。指针的值被复制了，但指针所指的内容并未复制。因此，包含动态分配成员的类除提供拷贝构造函数外，还应该考虑重载`=`赋值操作符。

  ```c++
  class CExample
  {
      public:
      	//构造函数
      	CExample()
          {
              ptr = NULL;
              size = 0;
          }
      	//拷贝构造函数
      	CExample(const CExample&);
      	//赋值构造函数（赋值符重载）
      	CExample& operator = (const CExample&); //赋值符重载
      	//析构函数
      	～CExample()
          {
              delete ptr;
          }
      	void init(int n)
          {
              ptr = new char[n];
              size = n;
          }
      
      private:
      	char* ptr;
      	int size;
  };
  
  CExample& CExample::operator = (const CExample& rightsides)
  {
      size = rightsides.size;
      char* temp = new char[size];
      memcpy(temp, rightsides.ptr, size * sizeof(char));
      
      delete[]ptr;
      ptr = temp;
      return *this;
  }
  ```

- 拷贝构造函数使用运算符重载的代码

  ```c++
  CExample::CExample(const CExample& rightsizes)
  {
      ptr = NULL;
      *this = rightsizes;
  }
  ```

- 拷贝构造函数和赋值运算符重载的重要意义

  - 首先明确基类和派生类的关系

    ```c++
    class Derived: public Base
    {
    	public:
    	....
    	private:
    	....
    };
    ```

    不同的继承方式的基类和派生类的特性

    ![1577245718130](img/inheritance.png)

  - 动态绑定中存在两个条件：1，必须是vitural虚函数；2，必须是通过基类的引用或者是基类的指针进行成员函数的调用。

  - 由于派生类中存在基类的成员，也就相当于一个派生类对象中包含了一个基类对象，所以可以采用一个基类引用来绑定一个派生类对象。引用实质上针对一块内存区域，引用是一个标号，是这块内存区域的一个名字，一个引用与一块内存区域绑定，因为派生类对象中存在基类部分，可以认为派生对象的区域中存在着基类对象，这时可用基类的引用来表明这块内存区域，即采用一个基类的别名来绑定这段区域，派生对象的地址以及内容都没有发生改变，没有重新创造出一个新的对象，基类的引用还是指向这个派生对象。对于指针类似。因此可以采用基类引用绑定派生类对象。

  - 如何实现派生类对象到基类的转换？

    这时候的转换与前面的绑定存在很大的差别，因为这是重新分配一个基类对象，而不在是引用问题，不再是绑定问题，而是依据派生类对象生成一个新的基类对象。因为派生类对象中存在一个基类对象的基本信息，完全可以生成一个基类对象，完全将此过程看成是一个初始化或者赋值问题。也就是采用派生类创建一个新的对象或者赋值一个对象。

    从上面的分析来看，可以采用下面的方式实现：

    ```c++
    Base(const Derived&);
    Base& operator = (const Derived&);
    ```

    是在基类函数中采用构造函数基于派生类来重载一系列的构造函数，但是也存在一个问题，如果存在很多派生类，就需要重载很多构造函数，这并不是我们需要的。

    这时候对于类而言，拷贝构造函数和赋值构造函数就很重要了。因为这两个函数都是接受一个基类的引用，根据前面的分析，可以知道一个基类引用完全可以绑定一个派生类的对象，而派生类对象中又包含了一个基类对象的基本信息。我们能够实现一个从派生对象到基类的构造过程。

    我们用一个基类引用绑定一个派生对象，然后采用基类引用对基类成员进行访问，完成了一个基类对象基本要素的填充操作，相当于完成了基类对象的创建，也就是构造问题。这样也就能完成有派生类对象到基类对象的构造过程。

    重载赋值操作符则是发生在使用一个派生对象来赋值一个基类对象时，这时候也是`const`基类引用绑定一个派生类对象，然后复制对象的基类成员到基类对象对应的成员中，完成一个基类对象成员的更新操作。

    拷贝构造函数不仅仅实现了同类型之间的初始化操作，同时也完成了派生类对象初始化一个基类对象的操作，赋值构造函数实现了同类型之间的赋值操作，也完成了派生类对象赋值基类对象的操作。

    

    

