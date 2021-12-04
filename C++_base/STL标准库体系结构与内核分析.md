# STL标准库体系结构与内核分析

## 前言

**所谓 Generic Programming （GP，泛型编程）就是使用 template 模板为主要工具来编写程序。而STL正是泛型编程最成功的作品**

STL源码存放位置：C:\mingw64\lib\gcc\x86_64-w64-mingw32\8.1.0\include\c++\bits

## OOP vs. GP

OOP（Object-Oriented Programming）: OOP将数据和算法关联起来

GP（generic Programming）：将数据和算法分开来

- 优势：Container和Algorithm开发可以分开来，通过迭代器沟通；算法通过迭代器确定范围，并通过迭代器取用元素

  <img src="STL标准库体系结构与内核分析.assets/image-20210928165353760.png" alt="image-20210928165353760" style="zoom:33%;" />

## 目标

- 使用C++标准库
- 认识C++标准库
- 良好使用C++标准库
- 扩充C++标准库

## STL六大部件

- 容器 Container     各种数据结构，如 vector, list, deque, set, map 等，用来存放数据，从实现的角度来看，STL 是一种 class template。

<img src="STL标准库体系结构与内核分析.assets/image-20210930154910715.png" alt="image-20210930154910715" style="zoom:50%;" />

- 分配器 Allocator   负责空间配置与管理。从实现的角度来讲，配置器是一个实现了动态空间配置、空间管理、空间释放的 class template （可省略）
- 算法 Algorithm      常用算法如 sort, search, copy, erase 等，从实现的角度来看，STL算法是一种 function template
- 迭代器 Iterator
- 适配器 Adapter    一种用来修饰容器或仿函数或迭代器接口的东西。
- 仿函数 Functor   行为类似函数，可作为算法的某种策略。从实现的角度看，仿函数是一种重载了operator()的class或class template，一般函数指针可视为狭义的仿函数。

<img src="STL标准库体系结构与内核分析.assets/image-20210925154913041.png" alt="image-20210925154913041" style="zoom:50%;" />

## 关于 「左闭右开」区间

**标准库使用的是：begin() 表示第一个元素；end()表示最后一个元素的后一个元素**

<img src="STL标准库体系结构与内核分析.assets/image-20210925161128290.png" alt="image-20210925161128290" style="zoom: 50%;" />

## STL-1 容器的分类

### 顺序容器

- array 

- vector

- list

- forward_list[==slist]

- deque

### 关联容器（查找快）

`**有序容器（内部基于 红黑树 实现、数学地）*`

![image-20210928110845516](STL标准库体系结构与内核分析.assets/image-20210928110845516.png)

#### 有序

- multiset 

- multimap

- set

- map

#### 无序

`（内部基于 HashTable 实现，经验地）`

C++ 11 之后无序容器的名称发生的变化如下图所示

<img src="STL标准库体系结构与内核分析.assets/image-20211009114103616.png" alt="image-20211009114103616" style="zoom: 50%;" />

- unordered_multiset

- unordered_multimap

- unordered_set

- unordered_map

### 容器适配器

> 由于 stack 和 queue 中实际上没有自己的数据结构，所以不把他们叫做容器，而是叫做  `容器适配器` （也有人就叫容器）

- stack

- queue



## 容器的详细介绍

### 1 array 

- 无法扩充大小，因此其使用时必须指定大小，如下图所示：

  ![image-20211005193207489](STL标准库体系结构与内核分析.assets/image-20211005193207489.png)

#### 迭代器

<img src="STL标准库体系结构与内核分析.assets/image-20211005193557507.png" alt="image-20211005193557507" style="zoom:50%;" />

- 参考[vector](# vector 的迭代器)



### 2 vector

`注意` vector中分配的空间的大小是 `*2 增长`的，如下图所示：

![image-20210926232055555](STL标准库体系结构与内核分析.assets/image-20210926232055555.png)

![image-20210926232109938](STL标准库体系结构与内核分析.assets/image-20210926232109938.png)

`并且十分需要注意的是` ：vector的成长两倍的过程是比较缓慢的，这是因为不可能直接成长2倍，而是必须要找到另外一块2倍大小的空间，然后将原来位置的元素一个个搬过去；因此其每次的成长都会大量的调用 **拷贝构造函数** 和 **析构函数**

#### vector 类的大小

<img src="STL标准库体系结构与内核分析.assets/image-20211004174308919.png" alt="image-20211004174308919" style="zoom: 33%;" />

由于只要这`三个指针`就可以完全确定一个 vector，所以其大小为 $4 字节 * 3 = 12 字节$（参见[蓝色方框](# 容器之间的关系)）

```C++
template <class T, class Alloc = alloc>
class vector {
...
protected:
    iterator start;          // 表示目前使用空间的头
    iterator finish;         // 表示目前使用空间的尾，即最后一个元素的下一个元素
    iterator end_of_storage; // 表示目前分配的整个空间的尾
    ...
};
```

#### vector 的迭代器

<img src="STL标准库体系结构与内核分析.assets/image-20211005175321781.png" alt="image-20211005175321781" style="zoom:50%;" />

`注` 由于vector的存储空间是连续的，因此其迭代器不用设计成一个class，可以直接使用一个指针来代替，如上图所示，因此萃取机会偏特化成 T* 类型的版本。







### 3 list（双向链表） /  forward_list（单向链表）[==slist]

- list 的继承图：

  <img src="STL标准库体系结构与内核分析.assets/image-20211004164837535.png" alt="image-20211004164837535" style="zoom:50%;" />

- list和forward_list的主要区别在于：forward_list是传统数据结构意义中的单链表，而list是双链表

![image-20210927232024072](STL标准库体系结构与内核分析.assets/image-20210927232024072.png)

`注意` 

```swift
sort() 函数模板定义在头文件 algorithm 中，要求使用随机访问迭代器。但 list 容器并不提供随机访问迭代器，只提供双向迭代器，因此不能对 list 中的元素使用 sort() 算法。但是，还是可以进行元素排序，因为 list 模板定义了自己的 sort() 函数。

也就是说：标准库定义的 sort() 函数在 list 排序中无法使用，16 行的代码是错误的，15 行是正确的
```

#### list 类 的大小

 `注意` 要注意区分 `list 本身的大小` 和 `List 类的节点的大小`

如下图所示是 STL 中 list 类的定义：

<img src="STL标准库体系结构与内核分析.assets/image-20211004155210543.png" alt="image-20211004155210543" style="zoom:50%;" />

其中数据部分只包含一个 list_node*  的指针，所以，其在32 位机上 所占用的内存为 4 字节（参见[蓝色方框](# 容器之间的关系)）

而每个节点的大小：

<img src="STL标准库体系结构与内核分析.assets/image-20211004155518742.png" alt="image-20211004155518742" style="zoom:50%;" />

是由 2个 指针和一个 T类型的模板数据所构成的。

- 补充：

  <img src="STL标准库体系结构与内核分析.assets/image-20211004164320549.png" alt="image-20211004164320549" style="zoom:50%;" />

#### 迭代器 iterator

<img src="STL标准库体系结构与内核分析.assets/image-20211004163418030.png" alt="image-20211004163418030" style="zoom:80%;" />



### 4 deque

- `分段连续` ： `连续是假象，分段是事实`，迭代器走到分段的边界时，要跳到下一个分段的首

- 模型：

  <img src="STL标准库体系结构与内核分析.assets/image-20211005195248826.png" alt="image-20211005195248826" style="zoom:50%;" />

- 具体做法： 依赖 buffer 和 指针 连接起来，`每次扩充一个 buffer` ，如下图所示：

  <img src="STL标准库体系结构与内核分析.assets/image-20211005195904834.png" alt="image-20211005195904834" style="zoom:50%;" />

- 注意：几乎每个容器都提供了 begin() 和 end() 这两个迭代器，所以上图也画出了 start 和 finish

- > iterator 中的 node 是指针的指针，其余是指针？求证

#### deque 类的大小

<img src="STL标准库体系结构与内核分析.assets/image-20211005201418750.png" alt="image-20211005201418750" style="zoom: 80%;" />

所以其大小为 ：$16 * 2 + 4 + 4 = 40 字节$

其中，迭代器的大小如下图所示：

<img src="STL标准库体系结构与内核分析.assets/image-20211005201534604.png" alt="image-20211005201534604" style="zoom:50%;" />

`至于他内部放多少个元素是动态分配获得的，跟这个对象本身没有关系`



#### 以 deque<T>::insert() 为例谈deque的妙处

由于：每次调用 insert() 插入新数据的时候，都需要调用构造函数将元素向前或者向后推动，以空出一个位置让新数据填入，因此，如果元素比较靠近前端就应该将前面的元素向前推动；否则就应该将后端的元素向后推动：

<img src="STL标准库体系结构与内核分析.assets/image-20211005202852742.png" alt="image-20211005202852742" style="zoom:50%;" />

<img src="STL标准库体系结构与内核分析.assets/image-20211005203021038.png" alt="image-20211005203021038" style="zoom:50%;" />

#### deque如何模拟连续空间

`答案 ： 全部都是 deque iterator 的功劳`

`迭代器 de 操作符重载`

如下图所示：一定是对 $-$ 号做了操作符重载，因为计算 $size$ 大小 其实就是要计算 「在 start 到 finish 之间有多少个元素」也就是 ：

```swift
在 start 到 finish 之间的buffer 个数 * 每个buffer 的大小 + start这个 buffer 中的元素 + finish所指向的buffer 的元素个数
```

<img src="STL标准库体系结构与内核分析.assets/image-20211006140916839.png" alt="image-20211006140916839" style="zoom:50%;" />

<img src="STL标准库体系结构与内核分析.assets/image-20211006141639238.png" alt="image-20211006141639238" style="zoom: 50%;" />

源码实现如下：

<img src="STL标准库体系结构与内核分析.assets/image-20211006142716113.png" alt="image-20211006142716113" style="zoom:50%;" />





### 5 stack

- 源码里面有一个 deque，也就是不需要重新写一个类， ，这样就形成一个新的容器，如下图所示：

  <img src="STL标准库体系结构与内核分析.assets/image-20211006155649964.png" alt="image-20211006155649964" style="zoom:50%;" />

- stack  和 queue 都不允许遍历，也不提供迭代器：

  <img src="STL标准库体系结构与内核分析.assets/image-20211006160213659.png" alt="image-20211006160213659" style="zoom:50%;" />

- stack 和 queue 都可以选择 list 或者 deque 作为 **「底层结构」**，默认是 deque，要改的话如下图所示：

  <img src="STL标准库体系结构与内核分析.assets/image-20211006160356887.png" alt="image-20211006160356887" style="zoom:50%;" />

- 另外，stack还可以使用 vector 作为**「底层结构」**
- `结论`：如果要作为**「底层结构」**，需要转调用的函数底层结构都有

### 6 queue

- 源码里面有一个 deque，也就是不需要重新写一个类， ，这样就形成一个新的容器，如下图所示：

<img src="STL标准库体系结构与内核分析.assets/image-20210928103012223.png" alt="image-20210928103012223" style="zoom:50%;" />

<img src="STL标准库体系结构与内核分析.assets/image-20211006155448296.png" alt="image-20211006155448296" style="zoom: 50%;" />

- queue不可以使用 vector 作为**「底层结构」**，这是因为 vector 不提供 pop_front() 成员函数，如下图所示：

  <img src="STL标准库体系结构与内核分析.assets/image-20211006160737138.png" alt="image-20211006160737138" style="zoom:50%;" />



### 7 multiset 

- set 的 value 就是 key ，key 就是 value

- 标准库有一个 find [::find(c.begin(), c.end(), target)]，容器自身有一个 find [c.find(target)]，容器自带的 find 查找速度会更快，如下所示：

  <img src="STL标准库体系结构与内核分析.assets/image-20211007230403084.png" alt="image-20211007230403084" style="zoom:80%;" />

![image-20210928104353716](STL标准库体系结构与内核分析.assets/image-20210928104353716.png)

- set/multiset 是以 rb_tree 为底层结构，因此有 「元素自动排序」的特性，排序的依据是 key
- `**无法**`使用红黑树的迭代器的 iterator改变元素值（会打破排序的状态），具体技术实现见[图片](# 标准库的实现)
- multiset 元素的 key 可以重复，因此其 insert() 用的是 rb_tree 的 insert_equal()



#### 标准库的实现

<img src="STL标准库体系结构与内核分析.assets/image-20211007224511647.png" alt="image-20211007224511647" style="zoom:80%;" />





### 8 multimap

- 要使用 $pair<int, string>(i, buf)$​ 类似的结构插入数据
- 特别 注意： multimap 不可以使用 [] 做数据的插入，只能使用上面的方式，参见 map

### 9 set

### 10 map

- 可以使用 [] 做数据的插入，例如：

  ![image-20210928111617745](STL标准库体系结构与内核分析.assets/image-20210928111617745.png)

  会自动组合成一个 pair



### 11 unordered_multiset

其概括地形象的结构图如下所示：

![image-20210928110300624](STL标准库体系结构与内核分析.assets/image-20210928110300624.png)

实际上其内部的实现是 由一个个的 bucket 串联起来的，每个bucket 上又挂了不同个数的 元素，如下图所示：

<img src="STL标准库体系结构与内核分析.assets/image-20210928110534188.png" alt="image-20210928110534188" style="zoom: 80%;" />

测试如下所示：

![image-20210928110621453](STL标准库体系结构与内核分析.assets/image-20210928110621453.png)

所以说，bucket 的个数 比元素的数量还多也是正常的，因为有些 bucket 是空的（事实上，bucket 一定比元素多 ）

### 12 unordered_multimap

### 13 unordered_set

### 14 unordered_map



### 更多的一些容器

#### priority_queue



## 哈希表 (Hash Table 散列表)

- `哈希表更多的是根据经验的设计`
- `bucket 个数 一定大于 元素的个数，否则就要 rehashing 扩充`

其实现过程如下所示：

![image-20211009102849951](STL标准库体系结构与内核分析.assets/image-20211009102849951.png)

### 哈希表的直接使用

<img src="STL标准库体系结构与内核分析.assets/image-20211009105641600.png" alt="image-20211009105641600" style="zoom: 50%;" />

<img src="STL标准库体系结构与内核分析.assets/image-20211009113627505.png" alt="image-20211009113627505" style="zoom:80%;" />

`注` strcmp() 函数的声明如下图所示：

<img src="STL标准库体系结构与内核分析.assets/image-20211009110109687.png" alt="image-20211009110109687"  />



## STL-5 迭代器的设计原则和Iterator Trait的作用与设计

- trait 本身的意思是：特征
- 萃取

### Iterator 需要遵循的原则

<img src="STL标准库体系结构与内核分析.assets/image-20211004171313858.png" alt="image-20211004171313858" style="zoom: 80%;" />

<img src="STL标准库体系结构与内核分析.assets/image-20211004171346740.png" alt="image-20211004171346740" style="zoom:50%;" />

**上面图中的 5 种typedefine 就是迭代器给出的回答**

### 问题的关键

但：如果 iterator 并不是一个 class ，例如 `native pointer` (普通的指针被视为一种退化的迭代器)

这样的话，算法问的问题，指针是无法回答的，因为根本就不存在类的定义

这时候就需要 trait

<img src="STL标准库体系结构与内核分析.assets/image-20211004172115138.png" alt="image-20211004172115138" style="zoom:50%;" />

### trait 的做法

`不直接问` 直接问是如下这样的：

<img src="STL标准库体系结构与内核分析.assets/image-20211004172355464.png" alt="image-20211004172355464" style="zoom:50%;" />

此时，我们先不直接问，间接问：

<img src="STL标准库体系结构与内核分析.assets/image-20211004172714987.png" alt="image-20211004172714987" style="zoom: 50%;" />

完整的5个问题的版本如下：

<img src="STL标准库体系结构与内核分析.assets/image-20211004172932466.png" alt="image-20211004172932466" style="zoom: 50%;" />

### 各式各样的 Traits

<img src="STL标准库体系结构与内核分析.assets/image-20211004173038820.png" alt="image-20211004173038820" style="zoom: 33%;" />





## 容器之间的关系

![image-20211006164227718](STL标准库体系结构与内核分析.assets/image-20211006164227718.png)

`a要用b的功能，a可以继承b，或者拥有b，STL 中用的是复合（拥有）`

左边蓝色的方框中标注的是在 32位机 上，在不同版本下容器本身所占的字节大小



## STL- 3 分配器 Allocator

分配器的实现流程如下图所示：

![image-20210930152504372](STL标准库体系结构与内核分析.assets/image-20210930152504372.png)

> GCC 2.9 的 `allocator类` 只是以 ::operator new 和 ::operator delete 完成 allocate() 和 deallocate() ，没有任何特殊设计；但是在调用 malloc 的时候，会有许多的 overhead 也就是头尾的额外开销，而且这种简单的封装只会减缓运行速度，因此，GNU 2.9 并没有使用这个类，其默认的调用的分配器是 `alloc 类`，其行为模式如下图所示：

<img src="STL标准库体系结构与内核分析.assets/image-20210930153401818.png" alt="image-20210930153401818" style="zoom:50%;" />

思路是，由于容器的大小是固定的，因此可以省去上下的 cookie ：

<img src="STL标准库体系结构与内核分析.assets/image-20210930153755907.png" alt="image-20210930153755907" style="zoom:50%;" />



## STL-2 标准库提供的算法

### 算法举例

#### find() 

- 顺序查找

#### _if

`一般算法中出现 if 条件说明会有一个仿函数的模板参数`

<img src="STL标准库体系结构与内核分析.assets/image-20211011150012318.png" alt="image-20211011150012318" style="zoom:80%;" />

#### sort算法

<img src="STL标准库体系结构与内核分析.assets/image-20211011151200541.png" alt="image-20211011151200541" style="zoom:80%;" />





## 关于 reverse_iterator

**这里需要注意，指针指向（黑色箭头） 和 指针指向的内容（绿色箭头） 的区别**

<img src="STL标准库体系结构与内核分析.assets/image-20211011153758016.png" alt="image-20211011153758016" style="zoom:80%;" />



## STL - 4 仿函数 Functors

- 最容易融入标准库，服务于算法

- 加就加，减就减，为什么要把这些动作设计成一个函数呢，这些因为要把这些动作传到算法里面去

<img src="STL标准库体系结构与内核分析.assets/image-20211011160038261.png" alt="image-20211011160038261" style="zoom:80%;" />

### 举例 重要 

![image-20211011160450055](STL标准库体系结构与内核分析.assets/image-20211011160450055.png)

`注意` 标准库中的仿函数都继承了 binary_function(两个操作数)，（实际上还可以继承自 unary_function），只有继承了这两个类之一，才可以融入「STL体系」

如下所示：

<img src="STL标准库体系结构与内核分析.assets/image-20211011162610095.png" alt="image-20211011162610095" style="zoom:80%;" />

这种继承叫做：`仿函数可适配(adaptable)的条件`



## STL-6 适配器 

- 实际上叫做 改造器 的话还挺好的

<img src="STL标准库体系结构与内核分析.assets/image-20211011163142448.png" alt="image-20211011163142448" style="zoom:50%;" />

### 概述

如果a要改造b的功能，那么也就是要在幕后调用b，有两种方式 a继承自b 或者 a 复合（内含）一个b；在STL中，统统采用后者的方式；如上图所示总结为下图

<img src="STL标准库体系结构与内核分析.assets/image-20211011163943677.png" alt="image-20211011163943677" style="zoom:50%;" />

#### 最简单的-容器的适配器 adapter

<img src="STL标准库体系结构与内核分析.assets/image-20211011164044982.png" alt="image-20211011164044982" style="zoom:50%;" />

`改造了函数的接口，开放部分底层的函数，改变名字等`

#### 函数的适配器 

<img src="STL标准库体系结构与内核分析.assets/image-20211011172132460.png" alt="image-20211011172132460" style="zoom:80%;" />

#### 迭代器adapter适配器

关键是 来修饰、改造原来的正向的adapter，如下图所示：

<img src="STL标准库体系结构与内核分析.assets/image-20211011224842783.png" alt="image-20211011224842783" style="zoom:80%;" />

#### X适配器

##### ostream_iterator

<img src="STL标准库体系结构与内核分析.assets/image-20211012224524254.png" alt="image-20211012224524254" style="zoom:80%;" />

##### istream_iterator



## 一个万用的Hash Function

- 基本想法：一些基本的元素已经有他们的Hash Function，例如整数、字符串等；是否可以将一个复杂的结构拆分成一些简单的部分，然后将他们的Hash Code相加

  ![image-20211015164049480](STL标准库体系结构与内核分析.assets/image-20211015164049480.png)

  问题：单纯的相加可以存在很多碰撞，所以更好的方式已经被设计，如下所示： 

  <img src="STL标准库体系结构与内核分析.assets/image-20211015165150730.png" alt="image-20211015165150730" style="zoom:80%;" />

### Hash Function 实现的 3 种方法/形式

![image-20211015163746118](STL标准库体系结构与内核分析.assets/image-20211015163746118.png)

或者，去实现类似标准库中偏特化版本：

<img src="STL标准库体系结构与内核分析.assets/image-20211015170030884.png" alt="image-20211015170030884" style="zoom: 80%;" />

## Tuple 元组

### 使用方法

<img src="STL标准库体系结构与内核分析.assets/image-20211015180355761.png" alt="image-20211015180355761" style="zoom:80%;" />

### 实现tuple的重要技术——自动递归

<img src="STL标准库体系结构与内核分析.assets/image-20211015180503188.png" alt="image-20211015180503188" style="zoom:80%;" />
