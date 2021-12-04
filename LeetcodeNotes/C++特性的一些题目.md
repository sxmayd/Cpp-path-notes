# C++特性的一些题目

### C++ 迭代器

#### [剑指 Offer 20. 表示数值的字符串](https://leetcode-cn.com/problems/biao-shi-shu-zhi-de-zi-fu-chuan-lcof/) ★

题目描述：
部分数值列举如下：["+100", "5e2", "-123", "3.1416", "-1E-16", "0123"]

部分非数值列举如下：["12e", "1a3.14", "1.2.3", "+-5", "12e+5.4"]

`也就是说，表示数值的字符串遵循模式 A.[B][eC]​ 或者 .[B][eC]`​

```C++
class Solution {
public:
    string::iterator sEnd;

    bool isNumber(string s) {
        if(s.empty()) return false;

        auto iter = s.begin();
        sEnd = s.end();

        // 去除开头的空格，例如 " 0"
        while(*iter == ' ') iter++;

        // 寻找 A 整数位
        bool numeric = scanInteger(iter);

        // 如果出现 '.' 则接下来是 B 小数部分
        if(*iter == '.'){
            iter++;
            // 小数位 和 整数位 存在一个即可
            numeric = scanUnsignedInteger(iter) || numeric;
        }

        // 存在 e, 查找指数位
        if(*iter == 'e' || *iter == 'E'){
            iter++;
            // 如果有 e 则指数位 C 必须存在
            numeric = scanInteger(iter) && numeric;
        }

        // 去除数字最后的空格 例如： "1 "
        while(*iter == ' ') iter++;

        // 得同时判断是否到达字符串结尾 以免出现这种情况 "1 4"
        return numeric && (iter == s.end());
    }

    // 扫描 0-9 的数字
    bool scanUnsignedInteger(string::iterator& iter){
        bool have_number = false;
        while(iter != sEnd && (*iter) >= '0' && (*iter) <= '9'){
            have_number = true;
            iter++;
        }
        return have_number;
    }

    // 扫描含有 +/- 号的 0-9 的数字
    bool scanInteger(string::iterator& iter){
        if(*iter == '+' || *iter == '-') iter++;

        return scanUnsignedInteger(iter);
    }
};
```

`注`

1. 将字符串划分为3部分，A:小数点之前 B:小数点之后,e之前 C:e之后
   `A和B只需存在一个，若e存在，C必须存在。按照此规则处理，若成功到达字符串结尾，返回true`
   **使用STL的迭代器传递参数，达到和数组的指针同等的便利性和空间复杂度**

### C++ 静态成员变量

#### [剑指 Offer 64. 求1+2+…+n](https://leetcode-cn.com/problems/qiu-12n-lcof/)

题目描述：求 $1+2+...+n$ ，要求不能使用乘除法、for、while、if、else、switch、case等关键字及条件判断语句（A?B:C）

```C++
class Temp{
public:
    // 构造函数
    // 将与累加相关的代码放到构造函数里
    Temp(){
        N++;
        Sum += N;
    }

    static void Reset(){
        N = 0;
        Sum = 0;
    }

    static int GetSum(){
        return Sum;
    }

private:
    static int N;
    static int Sum;
};

// 静态成员在类内声明，类外定义
int Temp::N = 0;
int Temp::Sum = 0;

class Solution {
public:
    int sumNums(int n) {
        //注意：力扣的所有测试用例好像会共用同一套静态成员变量，所以在return之前，记得把静态成员变量置0
        Temp::Reset();
        
        // 利用 创建 n 个类型的实例，就会调用 n 次该类型的构造函数
        Temp* a = new Temp[n];
        delete []a;
        a = NULL;

        return Temp::GetSum();
    }
};
```

### 虚函数

#### [剑指 Offer 64. 求1+2+…+n](https://leetcode-cn.com/problems/qiu-12n-lcof/)

题目描述：求 $1+2+...+n$ ，要求不能使用乘除法、for、while、if、else、switch、case等关键字及条件判断语句（A?B:C）

```C++
class A;
vector<A*> arry(2);

class A{
public:
    // 基类的虚函数
    virtual int getSum(int n){
        return 0;
    }
};

class B : public A{
public:
    // 派生类的虚函数
    virtual int getSum(int n){
        // !!(n) 实现:
        // n == 0 : !!(n) = 0  
        // n != 0 : !!(n) = 1
        return arry[!!n]->getSum(n - 1) + n;
    }
};


class Solution {
public:
    int sumNums(int n) {
        A* a = new A();
        B* b = new B();

        // 多态 n == 0 调用基类的虚函数； n != 0 调用派生类的基函数
        // 以此实现处理递归终止条件 n == 0的情况
        arry[0] = a;
        arry[1] = b;

        return arry[1]->getSum(n);
    }
};
```

