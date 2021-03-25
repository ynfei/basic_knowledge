# C++中的显示构造函数

有如下一个简单的复数类：

```c++
class ClxComplex
{
    public:
    	ClxComplex(double dreal = 0.0, double dimage = 0.0)
        {
            dreal_ = dreal;
            dimage_ = dimage;
        }
    	~ClxComplex();
    	
    	double getReal() const
        {
            return dreal_;
        }
    	double getImage() const
        {
            return dimage_;
        }
    private:
    	double dreal_;
    	double dimage_;
};
```

我们知道，下面的3行代码是等价的：

``` c++
ClxComplex lxtest = 2.0;
ClxComplex lxtest = ClxComplex(2.0);
ClxComplex lxtest = ClxComplex(2.0, 2.0);
```

其实，对于前两行来说，编译器都是把他们转换成第3行代码来实现的。因为我们写了构造函数，编译器就按照我们的构造函数来进行隐式转换，**单参数构造函数被自动类型转换**，直接把一个`double`类型隐式转换成了一个`ClxComplex`的对象。可是有些时候，我们不希望进行隐式转换，或者隐式转换会造成错误。比如下面的类：

```c++
clss ClxString
{
	public:
		ClxString(int string_length);
		ClxString(const char* string_ptr);
		~ClxString();
	private:
		char* string_ptr_;
};

ClxString::ClxString(int string_length)
{
    if(string_length)
    {
       	string_ptr_ = new char[string_length];
    }
}

ClxString::ClxString(const char* string_ptr)
{
    string_ptr_ = new char[strlen(string_ptr)];
    strcpy(string_ptr_, string_ptr);
}
ClxStirng::~ClxString()
{
    if(string_ptr_ != NULL)
    {
        delete string_ptr_;
    }
}
```

我们可以用字符串的长度来初始化一个`ClxString`对象，但是我们却不希望看到下面的代码：

```c++
ClxString lxtest  = 13; // 等同于ClxString lxtest = ClxString(13);
```

这会给阅读代码造成不必要的歧义。

另外，我们知道下面的代码是用字符串`A`来初始化一个`ClxString`对象：

```c++
ClxString lxtest = "A"; // 等同于ClxString lxtest = ClxString("A");
```

可是，如果有人写成：

```c++
ClxString lxtest = 'A';
```

那么上面的代码就会初始化成一个长度为65的字符串。上面的这种情况不是我们希望看到得 。在这个时候我们就要用到显示构造函数了。将构造函数声明为`explicit`就可以防止隐式转换。

```c++
class ClxString
{
    public:
    	explicit ClxString(int string_length);
    	ClxString(const char* string_ptr);
    	~ClxString();
    private:
    	char* string_ptr_;
};
```

在这种情况下，要想要用字符串的长度来初始化一个`ClxString`对象，那就必须显示的调用构造函数：

```c++
ClxString lxtest = ClxString(13);
```

而下面这些代码将不能通过编译。

```c++
ClxString lxtest = 13;
ClxString lxtest = 'A';
```

