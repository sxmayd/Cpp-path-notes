# 						赋值函数和构造函数

## 构造函数和赋值函数的区别

构造函数、析构函数、赋值函数是每个类最基本的的函数。

每个类只有一个析构函数和一个`赋值函数`。

但是有很多`构造函数`(一个为复制构造函数，其他为普通构造函数。对于一个类A，如果不编写上述四个函数，c++编译器将自动为A产生四个默认的函数，即：

```c++
A(void)                  		//默认无参数构造函数
A(const A &a)             		//默认复制构造函数
~A(void);                 		//默认的析构函数
A & operator = (const A &a); 	//默认的赋值函数
```

## 位拷贝 值拷贝（解释为什么需要自定义拷贝构造函数）

- 原因之一是“默认的复制构造函数”和"默认的赋值函数“均采用”位拷贝“而非”值拷贝“
- **位拷贝拷贝的是地址，而值拷贝拷贝的是内容**

```C++
class String  
{
    public:
        String(void);
        String(const String &other);
        ~String(void);
        String & operator =(const String &other);
    private:
    	char *m_data;        
    	int val;
};
```

如果定义两个String对象a, b。当利用位拷贝时，$a=b$，其中的$a.val=b.val$​；但是$a.m_data=b.m_data$就错了：$a.m_data$和$b.m_data$指向同一个区域。这样出现问题：

- $a.m_data$原来的内存区域未释放，造成内存泄露
- $a.m_data$和$b.m_data$指向同一块区域，任何一方改变，会影响到另一方
- 当对象释放时，$b.m_data$​会释放掉两次

因此当类中含有指针变量时，复制构造函数和赋值函数就隐含了错误。此时需要自己定义。

## 拷贝构造函数 和 赋值函数

```C++
class String  
{
    public:
        String(const char *str);
        String(const String &other);
        String & operator=(const String &other);
        ~String(void); 
    private:
        char *m_data;
};

String::String(const char *str)
{
    cout << "自定义构造函数" << endl;
    if (str == NULL)
    {
        m_data = new char[1];
        *m_data = '\0';
    }
    else
    {
        int length = strlen(str);
        m_data = new char[length + 1];
        strcpy(m_data, str);
    }
}

String::String(const String &other)
{
    cout << "自定义拷贝构造函数" << endl;
    int length = strlen(other.m_data);
    m_data = new char[length + 1];
    strcpy(m_data, other.m_data);
}

String & String::operator=(const String &other)
{
    cout << "自定义赋值函数" << endl; 

    if (this == &other)
    {
        return *this;
    }
    else
    {
        delete [] m_data;
        int length = strlen(other.m_data);
        m_data = new char[length + 1];
        strcpy(m_data, other.m_data);
        return *this;
    }
}

String::~String(void)
{
    cout << "自定义析构函数" << endl; 
    delete [] m_data;
}
```

`注`

1. 赋值函数中，上来比较 this == &other 是很必要的，因为防止自复制，这是很危险的，因为下面有delete []m_data，如果提前把m_data给释放了，指针已成野指针，再赋值就错了

2. 赋值函数中，接着要释放掉m_data,否则就没机会了（下边又有新指向了）

3. 拷贝构造函数是对象被创建时调用，赋值函数只能被已经存在了的对象调用

   ```C++
   String a("hello"); 
   String b("world"); 调用自定义构造函数
   String c = a；调用拷贝构造函数，因为c一开始不存在，最好写成String c(a);
   ```

   