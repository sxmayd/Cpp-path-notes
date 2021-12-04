# 												C++基础知识「侯捷课程」

## 基于对象、面向对象

1. 基于对象
   - 无指针的类
   - 带有指针的类

2. 面向对象
   - 继承
   - ......

## 内联 inline

1. 内联与否是由编译器决定的，关键字`inline`只是给编译器一个建议

## 构造函数

1. 构造函数的一个特殊点在于它可以有`初始化列表`进行`初始化`

   ```C++
   class A{
   public:
   	A() : n2(0), n1(n2 + 2) {}
       
   	void print()
   	{
   		cout << "n1:" << n1 << " ,  n2:" << n2 << endl;
   	}
       
   private: 
   	int n1;
   	int n2;
   }；
   ```

   而下面这种方式，相当于放弃了`初始化`的过程，转而去进行`赋值`，所以不太好

   ```C++
   A() {
   		n2=0;
   		n1=n2+2;
   }
   ```

   `注` 

   - 成员变量在**使用初始化列表**初始化时，与构造函数中**初始化成员列表的顺序无关**，**只与定义成员变量的顺序有关**。因为成员变量的`初始化次序`是根据变量在`内存中次序`有关，而内存中的排列顺序早在`编译期`就根据变量的`定义次序`决定了。

     所以上述两种方式的结果分别为：

     ![image-20210731121405332](C:\Users\mWX1034919\AppData\Roaming\Typora\typora-user-images\image-20210731121405332.png)

2. 构造函数可以有很多个 （`函数重载`），但是要注意下面这种情况不可以重载，是因为编译器也不知道该调用哪一个构造函数了。

   ![image-20210731122130523](C:\Users\mWX1034919\AppData\Roaming\Typora\typora-user-images\image-20210731122130523.png)

## 成员函数

### 常量成员函数

例如下列：

```C++
double real() const {
	return re;
}
```

`注`

不会改变数据的内容，加上`const`

## 参数传递

- by value 

- by reference（/to const）

  > 传引用就像C中传指针一样快，所以建议多用传引用

`问题` 传引用和传指针一样，如果传给某个函数，其改变了该值，那么会影响别的调用它的地方的值

`Solution` 加上const，如下所示，告诉函数**不可以改变这个引用**

```C++
complex& operator += (const complex&);
```

## 返回值传递

- by value 

  > ```C++
  > int c = a + b;
  > return c;
  > ```

- by reference（/to const）

  > ```C++
  > int a& func(int a, int b){
  > 	a += b;
  > 	return a;		
  > }
  > ```

## 友元

> 一个对象的成员函数是该类其他对象的「友元函数」，或者说，相同 class 的各个对象互为友元(friends)

上面这句话就解释了下面的行为为什么可行：

```C++
class complex {
public:

    // 成员函数是该类其他对象的友元函数
    int func(const complex& param) {
        return param.im+param.re;
    } 
private:
    // 成员变量都设置为private
    double re, im;
}；
    
int main(){
    complex c1(2, 1);
    compledx c2;
    c2.func(c1);
}
```

1. 友元函数破坏了封装性 可以访问对象的私有成员 **友元函数不属于成员函数 定义在类的外面**

