# Effective C++ notes-改善程序与设计的55个具体做法

[TOC]

## 1 让自己习惯C++

### 条款01：视C++为一个语言联邦

将C++视为一个由相关语言组成的联邦而非一个单一语言。在其某个次语言中，各种守则与通例都倾向简单、直观易懂、并且容易记住。然而当从一个次语言转移到另一个次语言，守则可能改变。为了理解C++，必须认识其主要的次语言，幸运的是C++只有四种次语言：

- C。C++是以C为基础的。
- Objec-Oriented C++。面向对象的C++。包含了封装、继承、多态、虚函数等等。
- Template C++。这是C++泛型编程的部分。
-  STL。STL是个template程序库。对容器、迭代器、算法以及函数对象的规约有极佳的紧密配合与协调。STL有自己特殊的办事方式，当你伙同STL一块工作，必须遵守它的规约。

记住这四个次语言，当从一个次语言切换到另外一个时，导致高效编程守则要求你改变策略时，不要感到惊讶。C++并不是一个带有一组守则的一体语言；它是从四个次语言组成的联邦，每个次语言都有自己的规约。记住这四个次语言你就会发现C++容易理解得多。

**请记住**

C++高效编程守则视状况而变化，取决于你使用C++哪一部分。

### 条款02: 尽量以`const` ,`enum`, `inline` 替换`#define`

**宁可以编译器替换预处理器**。宏定义的东西可能从来不会被编译器看到，出现因宏定义导致的错误很难找到。

对于浮点常量，使用`const`比`#define`更加节约空间。

当使用`const`时，它与指针的配合要注意控制的是指针本身，还是指针所指之物。

当使用const定义一个类的专属常量时，为确保此常量至多只有一份实体，必须让它成为一个`static`成员：

```c++
class GamePlayer
{
  private:
    static const int NumTurns = 5; //常量的声明
    int scores[NumTurns];          //使用该常量
};
```

需要注意的是，上面的常量是声明而不是定义。C++规定类内的`static`常量如果为整型的话，无需定义就还可以使用。只要不取它们的地址，就可以声明并使用它们而无需定义。但是如果要取这个常量的地址，或是编译器需要，则必须另外提供定义如下：

```c++
const int GamePlayer::NumTurns; //NumTurns的定义
```

这个有个特别之处是没有赋初值。这是由于class常量在声明的时候已经获得了初值，因此定义时不可以在设初值。有的比较旧的编译器不支持`static`成员在其声明式获得初值。此外所谓的**in-class 初值设定**，也只允许对整数常量进行。如果编译器不支持上述语法，可以将初值放在定义式。

这里有一个特别的例外是，当你在class编译期间需要一个class常量值，比如上面的数组中（编译器在编译期间必须知道数组的大小）。这时候万一编译器不支持**in class 初值设定**，可改用所谓的**the enum hack**补偿做法。其理论基础是：**一个属于枚举类型的数值可权充ints被使用**，于是可以定义如下：

```c++
class GamePlayer
{
  private:
    enum{NumTurns = 5};  //
    int scores[NumTurns];
};
```

这里有个**emun hack**的技术值得学习。第一：**enum hack**的行为某方面比较像`#define`，例如取一个`const`的地址是合法的，但是取一个`enum`的地址就不合法，而取一个`#define`的地址也不合法。如果你不想让别人获得一个`pointer`或`reference`指向你的某个整数常量，`enum`可以帮助你实现这个约束。第二：**enum hack**是模板元编程的基础计数，很多地方都会用到。

另外一个常见的`#define`误用情况是：宏看起来像一个函数，但是不会招致函数调用带来的额外开销。比如：

```c++
#define CALL_WITH_MAX(a, b) f((a) > (b)) ? (a) : (b)
```

这种宏必须带上括号，即使带上括号，也会出现问题。

```c++
int a = 5, b = 0;
CALL_WITH_MAX(++a, b);    //a被累加两次
CALL_WITH_MAX(++a, b+10); //a被累加一次
```

这种情况是很难检测到的。有另外一种方法既可以获得宏带来的效率以及一般函数所有可预料行为和类型安全性。那就是`template inline`函数：

```c++
template<typename T>
inline void callWithMax(const T& a, const T& b)
{
    f(a > b ? a : b);
}
```

**请记住**

**对于单纯常量，最好以`const`或`enums`替换`#deifine`**

**对于形似函数的宏，最好改用`inline`函数替换`#defines`**

### 条款03：尽可能使用const

如果一个变量在实际上是固定不变的，那么就用`const`来约束它，编译器会帮助减少不必要的错误。

**`const`最具威力的用法是面对函数声明时的应用。**

令函数返回一个常量值，往往可以降低因客户错误而造成的意外，而又不至于放弃安全性和高效性。

```c++
class Rational { ... };
const Rational operator* (const Rational& lhs, const Rational& rhs);
```

返回一个`const`对象的原因是，这样做可以避免这样的神操作：

```c++
Rational a, b, c;
...
(a * b) = c;
```

这很可能是一个无意识的一个错误。`if(a * b = c)`本来是想做一个比较的动作。

如果`a`和`b`都是内置类型，这样的代码直截了当就是不合法的。而一个**良好的用户自定义类型**的特征就是避免无端地与内置类型不兼容。

**`const`成员函数**

`const`成员函数之所以非常重要，基于两个理由。第一：使`class`接口比较容易理解。这是因为，得知哪个函数可以给改动对象内容而哪个函数不行，是很重要的。 第二：它们使`操作const对象`成为可能。因为改善C++程序效率的一个根本办法是以**pass by reference-to-const**方式传递对象，而此技术的可行的前提是，我们有const**成员函数**，可以用来处理**const对象**。

很多人漠视一件事情：两个成员函数如果只是**常量性**不同，可以被重载。

```c++
class TextBlock
{
  public:
    ...
    const char& operator[](std::size_t position) const
    {
        return text[position];
    }
    char& operator[](std::size_t position)
    {
        return text[position];
    }
  private:
    std::string text;
};
```

```c++
TextBook tb("Hello");
std::cout << tb[0];     //没问题，调用non-const TextBlock::operator[]
tb[0] = 'x';            //没问题，写一个non-const TextBlock
std::cout << ctb[0];    //没问题，读一个const TextBlock
ctb[0] = 'x';           //错误！写一个const TextBlock
```

注意，上述错误只`operator[]`的返回类型以致，至于`operator[]`调用动作自身没有问题。错误的起因是对`operator[]`返回的`const char&`进行赋值。

另外注意：`non-const operator[]`的返回类型是个`reference to char`，不是`char`。如果`operator[]`只是返回一个`char`，下面这样的句子就无法通过编译：

```c++
tb[0] = 'x';
```

那是因为，如果函数的返回类型是个内置类型，那么改动函数返回值从来就不合法。纵使合法，c++以`by value`返回对象这一事实意味着被改动的其实是`tb.text[0]`的一个副本，不是`tb.text[0]`自身，那不会是你想要的行为。

### 条款04：确定对象使用前已被初始化

避免出现各种因为初值不对引起的问题，最佳的办法是：**永远在使用对象之前先将它初始化**。对于无任何成员的内置类型，你必须手工完成此事。至于内置类型以外的任何东西，初始化责任落在构造函数身上。规则也很简单：**确保每一个构造函数都将对象的每一个成员初始化**。但是这里一个重要的点是别混淆了**赋值和初始化**。

```c++
class PhoneNumber{...};
class ABEntry
{
    public:
      ABEntry(const std::string& name, const std::string& address, const std::list<PhonebNumber>& phones);
    private:
      std::string theName;
      std::string theAdress;
      std::list<PhoneNumber> thePhones;
      int numTimesConsulted;
};

///这里有个非常重要的知识点
ABEntry::ABEntry(const std::string& name, const std::string& address, const std::list<PhoneNumber>& phones)
{
    theName = name;             //这些都是赋值，而非初始化。
    theAddress = address;
    thePhones = phones;
    numTimesConsulted = 0;
}
```

虽然上面的操作可以是对象带有你期望的值，但并不是最佳的做法。C++规定，**对象的成员变量的初始化动作发生在进入构造函数本体之前**。在`ABEntry`构造函数内，`theName` `theAddress`等等都不是被初始化，而是被赋值。初始化的发生时间更早，发生于这些成员的默认构造函数被自动调用之时（比进入ABEntry构造函数本体的时间更早）。构造函数的一个较佳的写法是，使用所谓的**成员初始化列表**：

```c++
ABEtry::ABEntry(cosnt std::string&name, const std::string& address, const std::list<PhoneNumber>& phones)
    :theName(name),         //现在这些都是初始化
	 theAddress(address),
     thePhones(phones),
	 numTimesConsulted(0)
     {}                    //构造函数不必有任何动作
```

这个构造函数和上一个的最终结果相同，但效率通常更高。

有些情况下即使面对的成员变量属于内置类型，也一定得使用初始化列表，避免遗漏。如果类中存在很多成员变量，这种情况下可以合理地在初值列表中遗漏那些**赋值表现像初始化一样好**的成员变量，改用它们的赋值操作，并将那些赋值操作移往某个函数（通常是private），供所有构造函数调用。

C++有着十分固定的**成员初始化顺序**。次序总是相同的：`base class` 更早与其`derived class`，而成员变量总是以其声明的次序被初始化。即使它们在成员初始化列表中以不同的次序出现，也不会有任何影响。为了避免晦涩的错误（例如，初始化`array`时需要指定大小，因此代表大小的那个成员变量必须先有初始值），**成员初始化列表中最好以声明次序为顺序**。

**不同编译单元non-local satic对象的初始化顺序**

**`static`  对象**包括`global`对象。定义于`namespace`作用域内的对象，在类内，函数内，文件作用域内被声明为`static`的对象。**函数内的对象称为`local static`对象**。**其他`static`对象称为`non-local static`对象**。

所谓编译单元是指产出单一目标文件的那些源码。基本上是单一源码文件加上 其所包含的头文件。

现在的问题是，至少两个源码文件，每一个至少含有一个`no-local static`对象（也就是说该对象是`global`,或位于`namespace`内，或在class内，或在文件作用域内）。真正的问题是：如果某个编译单元内的某个`non-local static`对象的初始化动作使用了另一编译单元的`non-local static`对象，它所用到的对象可能未被初始化，因为，C++对不同编译单元的`non-local static`对象的初始化顺序无明确定义。   

```c++
class FileSystem
{
  public:
    ...
    std::size_t numDisks() const;
    ...
};
extern FileSystem tfs;   //预备给客户使用的对象

class Directory
{
    public:
    	Directory(params);
        ...
};

Directory::directory(params)
{
    ...
    std::size_t disks = tfs.numDisks();
}

Directory tempDir(params);
```

这时候除非`tfs`在`tempDir`之前先被初始化，否则`tempDir`的构造函数会用到尚未初始化的`tfs`。这里有一个小技巧实现如下：

```c++
class FileSystem{ ... };
FileSystem& tfs()      //这个函数用来替换tfs对象：
{
    static FileSystem fs;
    return fs;
}

class Directory { ... };
Director::Direectory(params)
{
    ...
    std::size_t disks = tfs().numDisks();
    ...
}

Directory& tempDir()   //这个函数用来替换temDir对象
{
    static Directory td;
    return td;
}
```

这种结构下的`reference-returning`函数往往十分单纯：第一行定义并初始化一个`local static`对象，第二行返回它。

**请记住**

**为内置型对象进行手工初始化，因为C++不保证初始化它们。**

**构造函数最好使用成员初始化列表，而不要在构造函数本体内使用赋值操作。初值列表中的成员变量，其排列次序应该和它们在class中声明的次序相同。**

**为免除跨编译单元值初始化次序问题，请以`local static`对象替换`non-local static`对象**。

+++

## 2 构造/析构/赋值运算

### 条款05: 了解C++默默编写并调用哪些函数

C++默认生成了**构造函数，析构函数，`copy`构造函数，`copy`赋值操作符**，并且这些函数是`public` `inline`的。

`copy`构造函数和`copy assignment`操作符，编译器创建的版本只是单纯地将来源对象的每一个`non-static`成员变量拷贝到目标对象。在某些特定的情况下`copy assignment`操作符不会被生成。如下：

```c++
template<typename T>
class NamedObject
{
    public:
    	NamedObject(std::string& name, const T& value);
        ...
    private:
        std::string& nameValue;  //这是一个引用
        const T objectValue;     //这是一个const
};

//考虑下面会发生什么事情
std::string newDog("Persephone");
std::string oldDog("Satch");

NamedObject<int> p(newDog, 2);
NamedObject<int> s(oldDog, 36);

p = s;  //现在p的成员变量应该发生什么事情
```

`p.nameValue`是一个引用，可是C++并不允许引用可以指向不同的对象。面对这个难题，C++会拒绝编译一个`copy assignment`操作符。那就说如果一个类里面含有**引用成员**,必须自己定义一个`copy assignment`操作符。面对内含有`const`成员变量的情况也是一样的。

**请记住**

编译器可以暗自为类创建**默认构造函数，`copy`构造函数， `copy assignment`操作符，以及析构函数**。

### 条款06： 若不想使用编译器自动生成的函数，就该明确拒绝

- 做法一：将`copy`构造函数和`copy assignment`函数声明为`private`。明确声明一个成员函数，就阻止了编译器暗自创建其专属版本；而令这些函数为`private`，使得成功阻止人们调用它。这个做法普遍为大家所接受。一般而言这个做法并不绝对安全，因为**成员函数**和**友元函数**还是可以调用私有函数。如果调用会获得一个连接错误。

  ```c++
  class HomeForSale
  {
      public:
        ...
      private:
        HomeForSale(const HomeForSale&);   //只有声明，不需要具体实现。
        HomeForSale& operator= (const HomeForSale&);
  };
  ```

- 做法二：将连接期的错误移动至编译期是可能的。（而且是好事，毕竟越早侦查出错误越好），只要将`copy`构造函数和`copy assignment`操作符声明为`private`,但不是`HomeForSale`本身，而是一个专门为了组织复制操作的一个类`base class`内。

  ```c++
  class Uncopyable
  {
      protected:
      	Uncopyable() {}     //允许`derived`对象构造和析构
          ~Uncopyable() {}  
      private:
          Uncopyable(const Uncopyable&);     //阻止copy
          Uncopyable& operator= (const Uncopyable&);
  }
  
  //为求阻止HomeForSale对象拷贝，位移需要做的就是继承Uncopyable
  class HomeForSale: private Uncopyable
  {
   ...
  };
  ```

   上述情况下，任何人甚至是成员函数和友元函数尝试拷贝`HomeForSale`对象，编译器会试着生成`copy`构造函数和一个`copy assignment`操作符，这些函数的编译器生成版会尝试调用其`base class`的对应兄弟，那些调用会被编译器拒绝，因为其`base class`的拷贝函数是`private`。

**请记住**

为驳回编译器自动提供的机能，可将相应的成员函数声明为`private`并且不予实现。使用像`Uncopyable`这样的`base class`也是一种做法。

### 条款07： 为多态基类声明virtual析构函数

任何`class`只要带有`virtual`函数都几乎确定应该也有一个`virtual`析构函数。如果`class`不含`virtual`函数，通常表示它并不意图被用作一个`base class`。同样，无端的将所有`class`的析构函数声明为`virtual`也是错误的。只有当`class`内含有至少一个`virtual`函数时，才为它声明`virtual`析构函数。

**请记住**

**带多态性质的`base class`应该声明一个`virtual`函数。如果`class`带有任何`virtual`函数，它就应该拥有一个`virtual`析构函数。**

`class`的设计目的如果不是作为`base class`使用，或不是为了具备多态性，就不该声明`virtual`析构函数。

### 条款08： 别让异常逃离析构函数

如果析构函数的过程中不可避免地出现故障，不应该让这个故障传出析构函数。这时候有两个解决方案：

```c++
DBConn::~DBConn()
{
    try
    {
        db.close();
    }
    catch(...)
    {
        制作运转记录，记下对close的调用失败；
        std::abort();
    }
}
```

如果程序遭遇一个析构期间发生的错误后无法继续执行，强迫结束成熟是个合理选项。毕竟它可以阻止异常从析构函数传播出去。

另外一个做法是吞咽下调用`close`而发生的异常

```c++
DBConn::~DBConn()
{
    try
    {
        db.close();
    }
    catch(...)
    {
        制作运转记录，记下对close的调用失败；
    }
}
```

一般而言，将异常吞掉是个坏注意。一个较佳的策略是重新设计`DBConn`接口，使其客户有机会对可能出现的问题做出反应。如下：

```c++
class DBCoon
{
    public:
     ...
     void close()
     {
         db.close();
         closed = true;
     }
    ~DBCoon()
    {
        if(!closed)
        {
            try
            {
                db.close();
            }
            catch(...)
            {
                制作运转记录，记下对close的调用失败;

            }
        }
    }
};
```

**请记住**

**析构函数绝对不要吐出异常。如果一个被析构函数调用的函数可能抛出异常，析构函数应该捕捉任何异常，然后吞下它们（不传播）或结束程序。**

如果给客户需要对某个操作函数运行期间抛出的异常做出反应，那么`class`应该提供一个普通函数（而非在析构函数中）执行该操作。

### 条款09： 绝不在构造和析构过程中调用`virtual`函数

不该在构造函数和析构函数期间调用`virtual`函数，因为这样的调用不会带来预想的结果。如下：

```c++
class Transaction      //所有交易的base class
{
    public:
      Transaction();
      virtual void logTransaction() const = 0;     //做出一份因类型不同而不同的日志记录
};

Transcation::Transcation()
{
    ...
    logTransaction();    //最后动作是志记这比交易
}

class BuyTransaction: public Transaction
{
	public:
	  virtual void logTransaction() const;
	  ...
};

class SellTransaction: public Transaction
{
	public:
	   virtual void logTransaction() const;
	   ...
};

//执行这句会发生什么事情
BuyTransaction b;
```

当指向最后一句时，`BuyTransaction`构造函数会被调用，但是首先会调用`Transaction`构造函数。`Transaction`最后一句调用`virtual`函数会发生问题。这时候调用的`logTransaction`是`Transaction`内的版本，不是`BuyTransaction`的版本。`base class`构造期间`virtual`函数绝不会下降到`derived class`中。一种说法是：**在`base class`构造期间，`virtual`函数不是`virtual`函数**。

还有另外一种更加隐秘的错误：

```c++
class Transaction
{
    public:
    	Transaction()
        {
            init();               //这里调用non-virtual
        }
    	virtual void logTransaction() const = 0;
    private:
    	void init()
        {
            logTransaction();     //这里调用virtual
        }
};
```

当`logTransaction()`是一个纯虚函数时，会出现错误。然而如果`logTransaction()`是个正常的`virtual`函数（非纯虚函数）并在`Transaction`内带有一份实现，该版本就会被调用。但是建立`derived class`时会调用错误版本的`logTransaction`。唯一能够避免此问题的做法是：确定构造函数和析构函数都没有调用`virtual`函数，而它们所调用的所有函数都服从同一约束。

那么如何解决`logTransaction`有适当的版本被调用？

要求`derived class`构造函数传递必要信息给`Transaction`构造函数。

```c++
class Transaction
{
    public:
       explicit Transaction(const std::string& logInfo);
       void logTransaction(const std::string& logInfo) const;  //现在是个非虚函数
};

Transaction::Transaction(const std::string& logInfo)
{
    logTransaction(logInfo);             //现在是个非虚函数调用
}

class BuyTransaction: public Transaction
{
   public:
   	  BuyTransaction(parameters): Transaction(createLogString())   
   	  {...}                                 //将log信息传给`base class`
   private:
      static std::string createLogString(parameter);
};
```

换句话说，由于无法使用`virtual`函数从`base class`向下调用，可以令`derived class`将必要的构造信息向上传递至`base class`构造函数。

请注意本例中`private static`函数`createLogString`的运用。 比起在成员初始化列表内给予`base class`所需数据，利用辅助函数创建一个值传递给`base class`函数往往比较方便。此函数为`static`，也就不可能意外指向初期未成熟之`BuyTransaction`对象内尚未初始化的成员变量。

**请记住**

**在构造和析构期间不要调用`virtual`函数，因为这类调用从不下降至`derived class`。**

### 条款10： 令`operator=`返回一个`reference to *this`

为了实现连锁赋值，赋值操作符必须返回一个`reference`指向操作符的左侧实参。这是实现操作符时应该遵循的协议：

```c++
class Widget
{
    public:
       ...
       Widget& operator=(const Widget& rhs)    //返回类型是个reference指向当前对象。
       {
           ...
           return* this;    //返回操作符左侧对象
       }
}
```

这个协议不仅适用于以上的标准赋值形式，也使用于所有相关赋值运算。

```c++
class Widget
{
    public:
    	Widget& operator+=(const Widget& rht)
        {
            ...
            return* this;
        }
        Widget* operator=(int rhs)
        {
            ...
            return* this;
        }
};
```

注意这只是个协议，并无强制性，如果不遵循它，代码一样可以通过编译。

**请记住**

**令赋值操作符返回一个`reference to *this`。**

### 条款11： 在`operator=`中处理“自我赋值”

自我赋值发生在对象自己给自己赋值时。

```c++
class Widget {...};
Widget w;
w = w;     //赋值给自己

//有些比较隐蔽
a[i] = a[j]
*px = *py;
```

有些情况下的操作是比较危险的。

```c++
class Bitmap {...};
class Widget
{
    ...
    private:
    	Bitmap* pb;    //指针，指向一个从heap分配得到的对象
}
```

下面这个`operator=`赋值并不安全。

```c++
Widget& Widget::operator=(const Widget& rhs)
{
    delete pb;                 //停止使用当前的bitmap
    pb = new Bitmap(*rhs.pb);  //使用rhs bitmap的副本
    return* this;
}
```

如果`operator=`函数内的`this`和`rhs`是同一对象，`delete`不只是销毁当前对象的`bitmap`,它也销毁了`rhs`的`bitmap`。这时返回的指针指向一个已经被删除的对象。

避免这样做的故障是进行一个“证同测试”

```c++
Widget& Widget::operator=(const Widget& rhs)
{
    if(this == &rhs) return* this;    //如果不是同一个对象才进行赋值
    
    delete pb;
    pb = new Bitmap(*rhs.pb);
    return* this;
}
```

虽然这样赋值具备**自我赋值安全**，但是不具备**异常安全性**。如果`new Bitmap`出现异常，`Widget`最终会持有一个指针指向一块被删除的`Bitmap`。

那么为了避免上述问题，可以这么做：

```c++
Widget& Widget::operator=(const Widget& rhs)
{
    Bitmap* pOrig = pb;        //记住原先的pb;
    pb = new Bitmap(*rhs.pb);  //令pb指向*pb的一个副本
    delete pOrig;              //删除原先的pb
    return* this;
}
```

 现在，如果`new Bitmap`抛出异常，`pb`保持原状。

推荐的一个好方法(确保代码不但异常安全，而且自我赋值安全)。

```c++
class Widget
{
    ...
    void swap(Widget& rhs);    //交换*this和rhs的数据
};

Widget& Widget::operator=(const Widget& rhs)
{
    Widget temp(rhs);        //为rhs数据制作一份副本
    swap(temp);              //将*this数据和上述副本数据交换
    return* this;
}
```

**请记住**

**确保对象自我赋值时`operator=`有良好行为。其中包括比较来源对象和目标对象的地址，精心周到的语句顺序，以及`copy-and-swap`。**

**确定任何函数如果操作一个以上的对象，而其中多个对象是同一个对象时，其行为依然正确。**

### 条款12： 赋值对象时勿忘其每一个成分

如果你生命自己的`copy`函数，意思就是编译器不用生成默认的复制函数，而当你的复制函数发生错误的行为时，编译器却不一定提醒。

考虑如下情况：

```c++
void logCall(const std::string& funName);    //制造一个log entry
class Customer
{
    public:
       ...
       Customer(const Customer& rhs);
       Customer& oprator=(const Customer& rhs);
       ...
    private:
       std::string name;
};

Customer::Customer(const Customer& rhs)
    :name(rhs.name)                            //复制rhs的数据
    {
        logCall("Customer copy constructor");
    }
Customer& Customer::operator=(const Customer& rhs)
{
    logCall("Customer copy assignment operator");
    name = rhs.name;                        //复制rhs的数据
    return* this;
}
```

到目前为止每一件事看起来都很好，直到另外一个成员变量加入：

```c++
class Date{ ... };
class Customer
{
	public:
    	...
    private:
        std::string name;
        Date lastTransaction 
};
```

这时候既有的拷贝构造函数执行的是局部拷贝，复制了`name`，但是没有复制`lastTranscation`。大多数编译器对这种情况不会报错。所以结论很明显：如果你为class添加了一个成员变量，你必须同时修改`copy`函数。

另外一种更为严重的情况，如果发生继承：

```c++
class PriorityCustomer: public Customer
{
    public:
    	PriorityCustomer(const PriorityCustomer& rhs);
        PriorityCustomer& operator=(const PriorityCustomer& rhs);
        ...
    private:
        int priority;
};

PriorityCustomer::PriorityCustomer(const PriorityCustomer& rhs)
    : priority(rhs.priority)
    {
        logCall("PriorityCustomer copy constructor");
    }
PriorityCustomer& PriorityCustomer::operator=(const PriorityCustomer& rhs)
{
    logCall("PriorityCustomer copy assignment operator");
    priority = rhs.priority;
    return* this;
}
```

`PriorityCustomer`的`copy`函数仅仅复制了本类的成员变量，它所含有的继承的`Customer`的成员变量并没有被复制。`PriorityCustomer`的复制函数并没有指定实参传递给`base class`构造函数。默认构造函数将针对`name`和`lastTransaction`执行缺省的初始化动作。

如何对那些参数也进行复制呢，因为这写参数往往都是私有的，不可访问。具体做法如下：

```c++
PriorityCustomer::PriorityCustomer(const PriorityCustomer& rhs)
    :Customer(rhs),priority(rhs.priority)             //调用base class的copy构造函数
    {
        logCall("PriorityCustomer copy constructor");
    }
	
PriorityCustomer& PriorityCustomer::operator=(const PriorityCustomer& rhs)
{
    logCall("PriorityCustomer copy assignment operator");
    Customer::operator=(rhs);      //对base class成分进行复制操作
    priority = rhs.priority;
    return* this;
}
```

所以复制每一个成分包括①复制所有的local成员变量②调用所有base classes内的适当`copy`函数。

如果发现`copy`构造函数和`copy assignment`操作符里面有相近的代码，消除重复代码的做法是，建立一个新的成员函数给两者调用，这个函数往往是`private`的，且常备命名为`init`。

**请记住**

`copy`函数应该确保复制对象内的所有成员变量及所有`base class`成分。

不要尝试以某个`copying`函数实现另外一个`copying`函数。应该将共同机能放进第三个函数中，并由两个`copying`函数共同调用。

+++

## 3 资源管理

### 条款13： 以对象管理资源

假设我们有一个投资类

```c++
class Investment  {...}
```

有一个工厂函数供应我们特定的`Investment`对象

```c++
Investment* createInvestment();
```

先有一个函数调用了函数返回的对象后，有责任删除之。

```c++
Void f()
{
    Investment* pInv = createInvesment();    //调用factory函数
    ...
    delete pInv;                             //释放pInv所指对象
}
```

这种做法看起来妥当，但是若干情况下`f`可能无法删除这个投资对象。为了确保`createInvestment`返回的资源总是被释放，我们需要将资源放进对象内，当控制流离`f`，该对象的析构函数会自动释放那些资源。底层原理是：把资源放进对象内，遍可依赖C++的析构函数自动调用机制，确保资源被释放。以对象管理资源的两个关键想法是：

①**获取资源后立刻放进管理对象**(Resource Acquisition is Initialization; RAII)②**管理对象运用析构函数确保资源被释放。**一个很好的例子是：

```c++
void f()
{
    ...
    std::shared_ptr<Investment> pInv(createInvestment());
    ...
}
```

C++11中的只能指针就属于**RAII**对象。这里有一个需要注意的是，突然返现，c++并没有特别针对c++动态分配数组而设计类似的`shared_ptr`那样的东西，因为`vector`和`string`几乎总是可以取代动态分配的数组，如果你还希望拥有数组类型的`shared_ptr`，看看`Boost`吧，`boost::scoped_array`和`boost::shared_array`它们都提供你要的行为。

**请记住**

**为防止资源泄露，请使用RAII对象，它们在构造函数中获得资源并在析构函数中释放资源。**

**两个经常被使用的RAII class 分别是`shared_ptr`和`unique_ptr`。**

### 条款14： 在资源管理类中小心copying行为

如果一个资源是`heap-based`，那么像`shared_ptr`，`weak_ptr`等是适合的，然而并非所有的资源都是`heap_based`,那么这样的情况下，上面两种类就不适合作为资源掌管者。

```c++
class Lock
{
    public:
    	explict Lock(Mutex* pm): mutexPtr(pm)
        {
            lock(mutexPtr);            //获得资源
        }
        ～Lock()
        {
            unlock(mutexPtr);         //释放资源
        }
    private:
     	Mutex* mutexPtr;
};
```

客户对Lock的用法符合**RAII**方式

```c++
Mutex m;         //定义所需要的互斥器
...
{
Lock m1(&m);    //锁定互斥器
...
}               //在区块末尾，自动接触互斥器锁定

Lock m11(&m);   //锁定m
Lock m12(m11);  //将m11复制到12，将会发生什么？
```

当一个**RAII**对象发生复制情况时，大多时候会选择以下两种可能：

- **禁止复制**。将`copying`操作声明为`private`。

  ```c++
  class Lock: private: Uncopyable
  {
      public:
         ...   //如前
  }
  ```

- **对底层资源使用引用计数法**

  ```c++
  class Lock
  {
      public:
      	explict Lock(Mutex* pm): mutexPtr(pm, unlock) //以某个Mutex初始化shared_ptr
          {                                             //并以unlock函数为删除器
              lock(mutexPtr.get());                            
          }
      private:
          std::shared_ptr<Mutex> mutexPtr;
  }
  ```

当涉及到复制的时候，要么**复制底部资源**（深度拷贝），要么**转移底部资源的拥有权**（unique_ptr）。

**请记住**

**复制RAII对象必须一并复制它所管理的资源，所以资源的`copying`行为决定RAII对象的`copying`行为。**

**普遍而常见的RAII `copying`行为是：抑制`copying`,引用计数。**

### 条款15： 在资源管理类中提供对原始资源的访问

在条款13中，使用智能指针保存诸如`createInvestment()`的返回结果，假设你希望某个函数处理`Investment`对象，像这样：

```c++
int daysHeld(const Investment* pi);        //返回投资天数
```

使用如下方式调用：

```c++
int days = daysHeld(PInv);              //错误
```

上述过程编译不通过，因为`daysHeld`需要的是`Investment*`指针，传递的却是个`shared_ptr<Invesment>`的对象。这时候需要一个将函数将`RAII class` 对象转换为其所内含的原始资源。有两个做法可以达成目标：显示转换和隐式转换。

```c++
//显式转换
int days = daysHeld(pInv.get());       //将pInv内的原始指针传给daysHeld
```

几乎所有只能指针都重载了指针取值操作符，它们允许隐式转换至底部原始指针。

考虑下面的这个例子：

```c++
FontHandle getFont();
void releaseFont(FontHandle fh);

class Font
{
    public:
    	explicit Font(FontHandle fh): f(fh)      //获取资源
        {}
        ~Font()
        {
            releaseFont(f);                      //释放资源
        }
    private:
    	FontHandle f;                            //原始资源
}；
```

假设有很多与字体相关的`API`，它们处理的是`FontHandle`，那么将`Font`对象转换为`FontHandle`会是一种很频繁的需求。`Font class`可为此提供一个显式转换函数，像`get()`那样：

```c++
class Font
{
    public:
    	...
        FontHandle get() const                   //显式转换函数
        {
            return f;           
        }
}
```

客户每次使用API时就必须调用`get`:

```c++
void changeFontSize(FontHandle f, int newSize);
Font f(getFont());
int newFontSzie;
...
changeFontSize(f.get(), newFontSize);      //显式将Font转换为FontHandle
```

某些程序员认为，从此这般地到处要求显式转换，足以使人们倒尽胃口。另一个办法是令`Font`提供隐式转换函数，转型为`FontHandle`：

```c++
class Font
{
    public:
    	...
        operator FontHandle() const      //隐式转换函数
        {
            return f;
        }
        ...
};
```

这使得客户调用时比较轻松自然：

```c++
Font f(getFont());
int newFontSize;
...
changeFontSize(f, newFontSize);         //将Font隐式转换为FontHandle
```

但是这个隐式转换会增加错误机会。是否应该提供一个显示转换函数将RAII class转换为其底部资源，或是应该提供隐式转换，答案主要取决于RAII class被设计执行的特定工作，以及被使用的情况。最佳设计是坚持这个忠告：让接口容易被正确使用，不易被误用。通常显式转换是比较受欢迎的。

**请记住：**

**APIs往往要求访问原始资源，所以每一个RAII class 应该提供一个“取得其所管理之资源”的办法。**

**对原始资源的访问可能经由显式转换或隐式转换。一般而言显式转换比较安全，但隐式转换对客户比较方便。**

### 条款16: 成对使用new和delete时要采取相同形式

首先看以下动作有什么错误？

```c++
std::string* stringArray = new std::string[100];
...
delete stringArray;
```

上述数组中的对象的不太可能被适当删除，因为它们的析构函数很可能没有被调用。实际上这个问题可以更加简单些：即将被删除的那个指针，所指的是单一对象或对象数组？这是个必不可缺的问题，因为单一对象的内存布局一般而言不同与数组的内存布局。正确的操作方法如下：

```c++
std::string* stringPtr1 = new std::string;
std::string* stringPtr2 = new std::string[100];
...
delete stringPtr1;     //删除一个对象
delete [] stringPtr2;  //删除一个由对象组成的数组
```

如果对`stringPtr1`使用`delete[]`形式，会发生什么？结果未有定义，但不太可能让人愉快。`delete`会读取若干内存并将它解释为数组大小，然后开始多次调用析构函数，浑然不知它所处理的那块内存不但不是个数组，而且可能是其他对象。

**请记住**

**如果调用`new`时使用[],必须在`delete`时也使用[]。如果在`new`表达式中不使用[],一定不要在相应的`delete`表达式中使用[]。**

### 条款17：以独立语句将`newed`对象置入智能指针

考虑一个如下的函数：

```c++
int priority();
void processWidget(std::shared_ptr<Widget> pw, int priority);
```

如果采用如下的调用方式：

```c++
processWidget(new Widget, priority());
```

上述的情况不能通过编译，`shared_ptr`构造函数是一个`explicit`构造函数，无法进行隐式转换。所以写成如下形式：

```c++
processWidget(std::shared_ptr<Widget>(new Widget), priority());
```

令人惊讶的是，虽然我们在此使用了“对象管理资源”，上述调用却可能出现泄漏资源。为什么？

因为上述第一个实参`std::shared_ptr<Widget>(new Widget)`由两部分组成：

- 执行`new Widget`表达式
- 调用`std::shared_ptr`构造函数

于是在调用`processWidget`之前，编译器必须创建代码，做以下三件事：

- 调用`priority`
- 执行`new Widget`
- 调用`std::shared_ptr`

上述三个过程是以什么样的顺序执行？可以确定的是`new Widget`一定执行于`std::shared_ptr`之前，但是`priority`的调用顺序不一定。可能在第一，第二或第三。如果编译器以第二顺序执行它，然后`priority`的调用发生异常，会发生什么事？`new Widget`返回的指针将会丢失，引发资源泄漏。

避免这类问题的方法很简单：

```c++
std::shared_ptr<Widget> pw(new Widget);
processWidget(pw, priority());
```

**请记住**

**以独立语句将`newed`对象存储于智能指针内。如果不这样做，一旦异常被抛出，有可能导致难以察觉的资源泄漏。**

+++

## 4 设计与声明

### 条款18：让接口容易被正确使用，不易被误用

欲开发一个“容易被正确使用，不容易被误用”的接口，首先必须考虑客户可能做出什么样的错误。假设有一个专门用来表现日期的类如下：

```c++
class Date
{
    public:
    	Date(int month, int day, int year);
        ...
};
```

乍一看，这个接口通情达理，但客户很容易犯下两个错误：

```c++
Date d(30, 3, 1995);        //错误，应该是3，30,1995
Date d(2, 30, 1995);        //按键错误，本来想输入3，却输入了2
```

在防范“不值得拥有的代码”上，类型系统是你的主要同盟国。导入简单的外覆类型：

```c++
struct Day
{
    explicit Day(int d): val(d)
    {}
    int val;
};

struct Month
{
    explicit Month(int m): val(m)
    {}
    int val;
};

struct Year
{
    explicit Year(int y): val(y)
    {}
    int val;
}

class Date
{
    public:
    	Date(const Month& m, const Day& d, const Year& y);
        ...
};

Date d(30, 3, 1995);                     //类型不正确
Date d(Day(30), Month(3), Year(1995));   //类型顺序不正确
Date d(Month(3), Day(30), Year(1995));   //正确
```

一旦正确的类型就定位，限制其值有时候是合理的。例如，一年只有12个年份，所以`Month`应该反应这一事实。可以用`enum`，但是`enums`不具备我们希望的类型安全性。比较安全的解法是预先定义所有有效的`Months`:

```c++
class Month
{
    public:
    	static Month Jan() {return Month(1);}    //很奇怪这里为什么不用对象，后面解释
    	static Month Feb() {return Month(2);}
        ...
    	static Month Dec() {return Month(12);}
    private:
    	explicit Month(int m);     //阻止生成新的月份，这是月份专属数据
        
        
}
```

如果用对象的话可能会出现`non-local static`对象的初始化次序有可能出现问题。建议阅读条款4。

预防客户错误的另一个办法是，限制类型内什么事可以做，什么事不能做。常见的限制是加上`const`。

另外一个常见的准则是除非有好理由，否则应该尽量让你的`types`的行为与内置`types`一致。任何接口如果要求客户必须记得做某些事情，就是有着不正确使用的倾向。例如：

```c++
//条款13的一个工厂函数，返回一个指针
Investment* createInvestment();
```

为避免资源泄漏`createInvestment()`返回的指针必须删除，这样操作就很容易出错。应该返回一个智能指针，但是万一客户忘了?所以较佳的设计接口是先发制人,l令函数返回一个智能指针：

```c++
std::shared_ptr<Investment> createInvestment();
```

同时`shared_ptr`允许当只能指针建立起来时指定一个资源释放函数，绑定于智能指针身上。

```c++
std::shared_ptr<Investment> createInvestment()
{
    std::shared_ptr<Investment> retVal(static_cast<Investment*>(0),                                                            getRidOffInvestment);
    retVal = ...;   //令retVal指向正确的对象
    return retVal;
}
```

`std::shared_ptr`有一个特别好的性质是：它会自动使用它的每个指针专属的删除器,因而可以消除一个潜在的错误：`cross-DLL problem`。这个问题发生于对象在动态连接程序库（`DLL`）中被`new`创建，却在另一个`DLL`内被`delete`销毁。比如：`Stock`派生自`Investment`:

```c++
std::shared_ptr<Investment> createInvestment()
{
    return std::shared_ptr<Investment>(new Stock);
}
```

返回的那个`std::shared_ptr`可被传递给其他`DLLs`，无需在意`cross-Dll problem`问题。

**请记住**

**好的接口很容易被正确使用，不容易被误用。你应该在你的所有接口中努力达成这些性质。**

**促进正确使用的办法包括接口的一致性，以及与内置类型的行为兼容。**

**阻止误用的办法包括建立新类型，限制类型上的操作，束缚对象值，以及消除客户的资源管理责任。**

**`std::shared_ptr`支持定制删除器，这可防范`DLL`问题，可被用来自动解除互斥锁等等。**

### 条款19： 设计`class`犹如设计`type`

如何设计高效的`class`？首先必须了解你面对的问题。几乎每个`class`都会要求你面对以下提问，而你的回答往往导致你的设计规范：

- **新的`type`对象如何被创建和销毁**？这会影响到`class`的构造函数和析构函数以及内存分配函数和释放函数。
- **对象的初始化和赋值该有什么样的差别**？这个答案决定了你的构造函数和赋值操作符的行为。
- **新`type`的对象那个如果被`passed-by-value`,意味着什么**？记住，`copy`构造函数用来定义一个`type`的`pass-by-value`该如何实现。
- **什么是新`type`的合法值**？对于`class`的成员变量而言，通常只有某些数值集是有效的。那些数值决定了你的`class`必须维护的约束条件，以及必须进行的错误检查操作。
- **你的新`type`需要配合某个继承图系吗**？如果你继承自某些既有的`classes`，你就受到那些`classes`的设计的束缚，特别是受到`virtual`或`no-virtual`的影响。如果你允许其他`class`继承你的`class`，那会影响你所声明的函数，尤其是析构函数，是否为`virtual`。
- **你的新`type`需要什么样的转换**？你的`type`应该有转换行为吗？如果你允许隐式类型转换，就必须写一个隐式的构造函数。如果只允许`explicit`构造函数存在，就得专门写出负责执行转换的函数。
- **什么样的操作符和函数对此新`type`而言是合理的**？这个问题的答案决定将为你的`class`声明哪些函数。其中某些应该是`member`函数，某些则否。
- **什么样的标准函数应该驳回**？那些正是你必须声明为`private`者。
-  **谁该取用新的`type`的成员**？这个提问可以帮助你决定哪个成员为`public`，哪个为`protected`，哪个为`private`。
- **什么是新`type`的未声明接口**？它对效率、异常安全性以及资源运用提供何种保证？
- **你的新`type`有多么一般化**？或许你其实并非定义一个新`type`，而是定义一整个`types`家族，那么就应该定义一个新的`class template`。
- **你真的需要一个新`type`吗**？如果只是定义新的派生类，那么说不定单纯定义一个或多个`non-member`函数或`template`，更加能达到目标。

**请记住**

**`class`的设计就是`type`的设计。在定义一个新`type`之前，请确定你已经考虑过本条款覆盖的所有讨论主题。**

### 条款20： 宁以`pass-by-reference-to-const`替换`pass-by-value`

缺省情况下`c++`都是以`by value`方式传递对象至函数。除非另外指定，否则函数参数都是以实际实参的复件为初值，而调用端所获得的亦是函数返回值的一个复件。这些复件由对象的`copy`构造函数产出，这使得`pass-by-value`成为昂贵的操作。比如以下情况：

```c++
class Person
{
    public:
    	Person();
        virtual ~Person();
        ...
    private:
    	std::string name;
        std::string address;
};

class Student: public Person
{
    public:
    	Student();
        ~Student();
        ...
    private:
    	std::string schoolName;
        std::string schoolAddress;
};
```

现在考虑以下代码，其中调用函数`validateStudent`，后者需要一个`Student`实参（by value）并返回它是否有效：

```c++
bool validStudent(Student s);
Student plato;
bool platoIsOK = validateStudent(plato);
```

当上述调用发生时`Student`的`copy`构造函数会被调用，以`plato`为蓝本将`s`初始化。同样明显地，当`validateStudent`返回时`s`会被销毁。因此对此函数而言，参数传递的成本是“一次`Student copy`构造函数调用，加上一次`Student`析构函数调用”。

但那还不是整个完整的过程。`student`对象内有两个`string`对象，所以每次构造一个`Student`对象也就构造了两个`string`对象。此外`student`对象继承自`Person`对象，所以也必须构造出一个`Person`对象。一个`Person`对象又有两个`String`对象在其中，因此每一个`Person`构造动作有需要承担两个`string`构造动作。最终结果是，以`by value`方式传递一个`Student`对象会导致调用一次`student copy`构造函数，一个`Person copy`构造函数，四次`String copy`构造函数。因此以`by value`方式传递一个`student`对象，总体成本是**六次构造函数和六次析构函数**。

避免这种的方式很简单：

```c++
bool validateStudent(const Student& s);
```

这种传递方式的效率高的多，因为没有任何构造函数和析构函数被调用，因为没有任何对象被创建。修订后的版本中`const`是至关重要的。原先的`validateStudent`以`by value`方式接受一个`Student`参数，因此调用者知道它们是收到保护的，函数内绝不会对传入的`Student`做任何改变，只能对其副本做修改。现在`Student`以`by reference`方式传递，将它声明为`const`是必要的，因为不这样做的话，调用者会忧虑`validStudent`会不会改变它们传入的那个`Student`。

以`by reference`方式传递参数也可以避免`slicing`（对象切割）问题。当一个`derived class`对象以`by value`方式传递并被视为一个`base class`对象，`base class`的`copy`构造函数会被调用，而`derived`对象的那些特化性质全被切割掉了，仅仅留下了一个`base class`对象。但这并不是我们想要的：

```c++
class Window
{
    public:
    	std::string name() const;
        virtual void display() const;
};

class WindowWithScrollBars: Public Window
{
    public:
    	...
        virtual void display() const;
};
```

下面是错误的一个示范：

```c++
void printNameAndDisplay(Window w)
{
    std::cout << w.name();
    w.display();
}
```

当你调用上述函数：

```c++
WindowWithScrollBar wwsb;
printNameAndDisplay(wwsb);
```

参数w会被构造成为一个`window`对象，函数内不论传递过来的对象原本是什么类型，参数`w`就像一个`Window`对象。因此总是调用`Window::display`。解决办法是：

```c++
void printNameAndDisplay(const Window& w)
{
	std::cout << w.name();
    w.display();
}
```

现在传进来的窗口是什么类型，`w`就表现出那种类型。

对于内置类型而言，当你有机会选择采用`pass-by-value`或`pass-by-reference-to-const`时，选择`pass-by-vlaue`并非没有道理。这个忠告也适用于STL的迭代器和函数对象，因为习惯上它们都被设计为`pass-by-value`。一般而言，你可以合理假设`pass-by-value`并不昂贵的唯一对象就是内置类型和STL的迭代器和函数对象。至于其他任何东西都请遵守本条款的忠告，尽量以`pass-by-reference-to-const`替换`pass-by-value`。

**请记住**

**尽量以`pass-by-reference-to-const`替换`pass-by-value`。前者通常比较高效，并可避免切割问题。**

**以上规则并不适用于内置类型，以及STL的迭代器和函数对象。对他们而言，`pass-by-value`往往比较适当。**

### 条款21： 必须返回对象时，别妄想返回其`reference`

一旦程序员领悟了`pass-by-value`的效率牵连层面，往往一心一意根除`pass-by-value`带来的种种邪恶。在坚定追求`pass-by-reference`的纯度中，一定会犯下一个致命错误，开始传递一些`references`指向其实并不存在的对象。例如：

```c++
class Rational
{
    public:
    	Rational(int numerator = 0, int denominator = 1);
    private:
    	int n, d;               //分子（numerator）和分母（denominator）
    friend
        const Rational operator*(const Rational& lhs, const Rational& rhs);
};
```

这个版本`operator*`系以`by value`方式返回其计算结果。如果你完全不担心该对象的构造和析构成本，你其实明显逃避了责任。若非必要，没有人想要为这样的对象付出太多代价，问题是需要付出任何代价吗？

如果可以改而传递`reference`,就不需要付出代价。但是记住，所谓`reference`只是个名称，代表某个既有对象。任何时候看到一个`reference`声明式，你都应该立刻问自己，它的另外一个名称是什么？因为它一定是某物的另一个名称。以上述`operator*`为例，如果它返回一个`reference`，后者一定指向某个既有的`Rational`对象，内含两个`Rational`对象的乘积。

例如：

```c++
Rational a(1, 2);              //a = 1/2
Rational b(3, 5);              //b = 3/5
Rational c = a * b;            //c = 3/10
```

我们当然不希望这样一个内含乘积的`Rational`对象在调用`operator*`之前就存在。期望原本就存在一个其值为`3/10`的`Rational`对象并不合理。如果`operator*` 要返回一个`reference`指向如此数值，它必须自己创建那个`Rational`对象。

函数创建新对象的途径有二：在`stack`空间或在`heap`空间创建之。如果定义一个`local`变量，就是在`stack`空间创建对象。根据这个策略写：

```c++	
const Rational& operator*(const Rational& lhs, const Rational& rhs)
{
    Rational result(lhs.n * rhs.n, lhs.d * rhs.d);    //警告，糟糕的代码
    return result;
}
```

你可以拒绝这种做法，因为你的目标是要避免调用构造函数，而`result`却必须像任何对象一样地由构造函数构造起来。更严重的是，这个函数返回一个`reference`指向`result`，但是`result`是个`local`对象，而`local`对象在函数退出前就被销毁了。事情的真相是，任何函数如果返回一个`reference`指向某个`local`对象，都将一败涂地。（如果函数返回指针指向一个`local`对象，也是一样）。

于是让我们考虑在`heap`内构造一个对象，并返回`reference`指向它。

```c++
const Rational& operator*(const Rational& lhs, const Rational& rhs)
{
    Rational* result = new Rational(lhs.n * rhs.n, lhs.d * rhs.d);
    return* result;
}
```

你还是必须付出一个**构造函数调用**代价。但此外又有了另外一个问题：谁该对着被你`new`出来的对象实施`delete`?

在这种情况下，很难阻止内存泄漏：

```c++
Rational w, x, y, z;
w = x * y * z;                  //operator*(operator*(x, y), z)
```

这里，同一个语句调用了两次`operator*`，因而调用了两次`new`，也就需要两次`delete`,这绝对会导致内存泄漏。

所以不论`on-the-stack`或`on-the-heap`做法都不能避免资源泄漏。

或许你心里又出现下面这样的代码：

```c++
const Rational& operator*(const Rational& lhs, const Rational& rhs)
{
    static Rational result;            //警告，又一堆烂代码
    resutl = ...;
    return result;
}
```

就像所有用上`static`对象的设计一样，这一个也立刻造成对多线程安全的忧虑。还有更加深层的瑕疵：

```c++
bool operator==(const Ratoinal& lhs, const Rational& rhs);
Rational a, b, c, d;
if((a * b) == (c * d))
{
	...
}
else
{
    ...
}
```

表达式`((a * b) == (c * d))`总是被核算为`true`,不论`a,b,c,d`的值是什么！

上述语句写成下面的形式：

```c++
if（operator==(operator*(a, b), operator*(c, d))）
```

注意：在`operator==`被调用前，已有两个`operator*`调用起作用，两次调用的确各自改变了`static Rational`对象值，但由于它们返回的都是`reference `因此调用端永远看到就是对象的现值。

一个**必须返回新对象**的函数的正确做法是，就让哪个函数返回一个新对象。

```c++
inline const Rational operator*(const Rational& lhs, const Rational& rhs)
{
    return Rational(lhs.n * rhs.n, lsh.d * rhs.d);
}
```

当你必须在返回一个`reference`和返回一个`object`之间抉择时，尽量挑出正确的那个，让编译器尽可能降低成本。

**请记住**

**绝不要返回`pointer`或`reference`指向一个`local stack`对象，或返回`reference`指向一个`heap-allocated`对象，或返回`pointer`或`reference`指向一个`local static`对象而有可能同时需要多个这样的对象。条款4已经为在单线程中合理返回`reference`指向一个`local static`对象提供了一份设计实例。**

### 条款22： 将成员变量声明为`private`

为什么成员变量不该是`public`?

从语法一致性开始，如果成员变量不是`public`,客户唯一能够访问对象的办法就是通过成员函数。如果`public`接口内的每样东西都是函数，客户就不需要记住是否该使用小括号，因为每样东西都是函数。

另外使用函数可以让你对成员变量的处理有更精确的控制。可以实现出**不准访问**、**只读访问**、以及**读写访问**、甚至可以**惟写访问**。

```c++
class AccessLeves
{
    public:
    	...
        int getReadOnly() const
        {
            return readOnly;
        }
        void setReadWrite(int value)
        {
            readwrite = value;
        }
       int getReadWrite() const
       {
           return readwrite;
       }
       void setWriteOnly(int value)
       {
           writeonly = value;
       }
    private:
    	int noAccess;         //对此int无任何访问操作
        int readonly;         //对此int只做读访问
        int readwrite;        //对此int做读写访问
        int writeonly;        //对此int做惟写访问
};
```

最后，考虑到封装性。如果你通过函数访问成员变量，日后可改某个计算替换这个成员变量，而客户一点也不知道。如下：

```c++
class SpeedDataCollection
{
    public:
    	void addValue(int speed);                 //添加新数据
    	double averageSoFar() const;              //返回平均速速
};
```

`averageSoFar`函数有两种常规做法。做法之一是在class内设计一个成员变量，记录至今以来所有速度的平均值，当`averageSoFar`被调用，只需返回那个成员变量就好。另一个做法就是令`averageSoFar`每次被调用时重新计算平均值，次函数有权利调用每一笔速度值。第一种会使`SpeedDataCollection`对象变大，然而`averageSoFar`却十分高效；而第二种方式`averageSoFar`执行较慢，但是每个`SpeedDataCollection`对象比较小。哪种方式好取决与应用场景。重点是，由于通过成员函数来访问平均值，就可以替换不同的实现方式。封装的重要性比你最初见到它时还重要。

假如有一个`public`成员变量，而我们最终取消了它，多少代码可能被破坏？所有使用它的客户代码都会被破坏，那是一个不可知的大量。`protected`成员变量与`public`成员变量一样缺乏封装性。从封装性的角度看，其实只有两种访问权限：**`private`(提供封装)和其他（不提供封装）**。

**请记住**

**切记将成员变量声明为`private`。这可赋予客户访问数据的一致性、可细微划分访问控制、允诺约束条件获得保证，并提供作者以充分的实现弹性。**

**`protected`并不比`public`更具封装性。**

### 条款23：宁以`non-member`,`non-friend`替换`member`函数

比如有一个浏览网页的类，提供众多的访问函数

```c++
class WebBrowser
{
	public:
    	void clearCache();
    	void clearHistory();
    	void removeCookies();
    	...
};
```

许多用户想一次性执行这些动作，所以`WebBrowser`也可能提供这样一个函数：

```c++
class WebBrowser
{
    public:
    	void clearEverything();  //调用上面三个函数
    	...
}
```

当然这个机能也可以给由一个非成员函数调用适当的成员函数提供出来：

```c++
void clearBrowser(WebBrowser& web)
{
	web.clearCache();
    web.clearHistory();
    web.removeCookies();
}
```

面向对象守则要求，数据以及操作数据的那些函数应该捆绑在与块，这意味着它建议`member`函数是一个比较好的选择，**不幸的是，这个建议不正确**。面向对象守则要求数据应该尽可能被封装，`member`函数会带来更低的封装性。此外，提供`non-member`函数可允许对`WebBrowser`相关机能能有较大的包裹弹性，而导致比较低的编译相依度，增加`WebBrowser`的可延伸性。因此在许多方面`non-member`做法比`member`做法好。

对于成员变量来说，越多函数可访问它，数据的封装性就越低。如果要在`member`函数和一个`non-member,non-friend`函数之间作抉择，而且两者提供相同的机能，那么导致较大封装性的是`non-member no-friend`函数，因为它不增加**能够访问`class`内之`private`成分**的函数数量，它导致`WebBrowser`有比较大的封装性。

有两件事情需要注意：① 这个论述只适用于`non-member non-friend`函数。`friends`函数对`class private`成员的访问权利和`member`函数相同，从封装的角度看，选择的关键在`member`和`non-member non-friend`函数之间；②只因在意封装性而让函数**成为`class`的`non-member`**,并不意味着**它不可以是另一个`class`的`member`**。

在`C++`，比较自然的做法是让`clearBrowser`成为一个`non-member`函数并位于`WebBrowser`所在的同一`namespace`内：

```c++
namespace WebBrowserStuff
{
    class WebBrowser{...};
    void clearBrowser(WebBrowser& wb);
    ...
}
```

注意，这是`C++`标准程序库的组织方式。标准程序库并不是拥有单一、整体的头文件，而是拥有`vector``memory`等等，每个头文件声明`std`的某些机能。将所有便利函数放在多个头文件内但隶属同一个命名空间，意味着客户可以轻松扩展这一组便利函数。

**请记住**

- **宁可拿`non-member non-friend`函数替换`member`函数。这样做可以增加封装性、包裹性和机能扩充性。**

### 条款24：若所有参数皆需要类型转换，请为此采用`non-member`函数

令`classes`支持隐式类型转换通常是个糟糕的注意，但是也有例外，常见的例外是在建立数值类型时。假设有一个类型

```c++
class Rational
{
	public:
		Rational(int numerator = 0, int denominator = 1); //刻意允许隐式转换
    	int numerator() const;
    	int denominator() const;
    private:
    	...
};
```

想要这个类支持加法，乘法的等，那么这些计算函数是否应该由成员函数实现，或非成员函数实现？

假如先用成员函数实现：

```c++
class Rational
{
    public:
    	...
        const Rational operator* (const Rational& rhs) const;
};
```

这个设计能使两个有理数以最轻松的方式相乘

```c++
Rational oneEighth(1, 8);
Rational oneHalf(1, 2);
Rational result = oneHalf * oneEighth;
```

然而当尝试混合运算时，发现问题：

```c++
result = oneHalf * 2; //正常
result = 2 * oneHalf; //错误
//写成如下形式
result = oneHalf.operator*(2);
result = 2.operator*(oneHalf);
```

正常的操作是：

```c++
class Rational
{
    ...
};
const Rational operator* (const Rational& lhs, const Rational& rhs);
```

**请记住**

- **如果你需要为某个函数的所有参数进行类型转换，那么这个函数必须是个`non-member`**

## 5 实现

### 条款26：尽可能延后变量定义式的时间

**尽可能延后**的定义不仅在于延后变量的定义，直到非得使用该变量的前一刻为止，甚至应该尝试延后这份定义直到能够给它初值实参为止。

另外一个比较好的列子：

```c++
Widget w;
for (int i = 0; i < n; ++i)
{
    w = 取决于i的某个值；
}
//另外一种方法
for (int i =0; i < n; ++i)
{
    Widget w(取决于i的某个值)；
}
```

在Widget函数内部，以上两种写法的成本如下：

- 做法A：一个构造函数，一个析构函数，n个赋值操作；
- 做法B：n个构造函数，n个析构函数；

如果classes的一个赋值成本低于一组构造析构成本，做法A大体而言比较高效，尤其当n很大的时候。否则做法B或许比较好，此外做法A造成名称w的作用域更大。因此除非①你知道赋值成本比构造加析构低②你正在处理代码中效率高度敏感部分，否则应该使用做法B。

**请记住**

- **尽可能延后变量定义式的出现。这样做可增加程序的清晰度并改善程序效率**

### 条款27：尽量少做转型动作

**请记住**

- 如果可以，请尽量避免转型，特别是在注重效率的代码中避免`dymamic_casts`。如果有个设计需要转型动作，试着法阵无需转型的替代设计。
- 如果转型是必要的，试着将它隐藏于某个函数背后。客户随和是可以调用该函数，而不要将转型放进他们自己的代码内。
- 宁可使用c++类型的转型语句，不要使用旧式转型。前者很容易辨识出来，而且也比较有着分门别类的执掌。

### 条款30：透彻了解inlining的里里外外

`incline`只是对编译器的申请，并不是强制命令。这项申请可以隐喻提出，也可以提明确提出。隐喻方式是将函数定义于`class`定义式内部。

**请记住**

- 将大多数`inclining`限制在小型、被频繁调用的函数身上。这可使日后的调试过程和二进制升级更容易，也可使潜在的代码膨胀问题最小化，使程序的速度提升机会最大化。
- 不要只因为`function templates`出现在头文件，就将它们声明为`incline`。

### 条款31：将文件之间的编译依存关系将至最低

 **请记住**

- 支持“编译依存性最小化”的一般构想是：相依于声明式，不要相依于定义式。基于此构想的两个手段是`Handle classes`和`Interface classes`。
- 程序库头文件应该以“完全且仅有声明式”的形式存在。这种做法不论是否涉及`templates`都适用。

## 6 继承与面向对象设计

### 条款32：确定你的public继承塑模出is-a关系

**请记住**

- **`public`继承意味着`is-a`。适用于`base classes`上的每一件事情一定也适用于`derived classes`身上，因为每一个`derived class`对象也都是一个`base class`对象。**

### 条款33：避免遮掩继承而来的名称



- 派生类内的名称会遮掩基类内的名称。在`public`继承下从来没有人希望如此。
- 为了让被遮掩的名称再见天日，可使用using声明式或转交函数。

### 条款34：区分接口继承和实现继承

- 接口继承和实现继承不同。在public继承之下，派生类总是继承基类的接口。
- 纯虚函数只具体指定接口继承。
- 虚函数具体指定接口继承以及缺省实现继承。
- 非虚函数具体指定接口继承以及强制性实现继承。

### 条款35：考虑virtual函数以外的其他选择

### 条款36：绝不重新定义继承而来的non-virtual函数

### 条款37：绝不重新定义继承而来的缺省参数值

- vritual函数系动态绑定，而缺省参数值却是静态绑定。

- 绝对不要重新定义一个继承而来的缺省参数值，因为缺省参数值都是静态绑定的，而virtual函数是唯一应该覆盖的东西，却是动态绑定。

### 条款38：通过复合塑模出has-a或根据某物实现出

- 复合的意义和public继承完全不同。
- 在应用域，复合意味着has-a。在实现域，复合意味着根据某物实现出。

### 条款39：明智而审慎地使用private继承











































