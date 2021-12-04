# 内存管理

## 学习大纲&概览

<img src="C++内存管理.assets/image-20211113162136022.png" alt="image-20211113162136022" style="zoom: 50%;" />

### 上述 4 个Primitives 的使用举例

<img src="C++内存管理.assets/image-20211115152638534.png" alt="image-20211115152638534" style="zoom:80%;" />

- 这里的 primitive翻译为 原生的 比较好，指的是 C++原生的东西

`注`

​	上述GNUC 的版本是GNUC 2.9版本的，在GNUC 4.9版本中，已经变的和其他的正统一样了；而原来的那个alloc的换了个名字（pool_alloc）

<img src="C++内存管理.assets/image-20211113164740386.png" alt="image-20211113164740386" style="zoom:67%;" />

​	上述也是GNUC 的其中两个分配器，其实GNUC一共有7、8个分配器。



## new expression & delete expression

- new 是一个表达式，在new 的后面应该加上 class name

- new 的行为模式实际上是分为了以下三步

  <img src="C++内存管理.assets/image-20211113210932836.png" alt="image-20211113210932836" style="zoom:50%;" />

- delete 的行为模式实际上被划分为以下两步

  ![image-20211115152946646](C++内存管理.assets/image-20211115152946646.png)

## array new & array delete

- 使用方法以及 delete 不带 `[]` 的后果

  ![image-20211115164145879](C++内存管理.assets/image-20211115164145879.png)

- 小测试如下：

  ![image-20211115165133664](C++内存管理.assets/image-20211115165133664.png)

例如，在下面的例子中，加不加[]是一样的，析构函数没有意义

![image-20211115173304007](C++内存管理.assets/image-20211115173304007.png)

## placement new

`placement new 允许我们将对象构建于 已经 allocated 的内存中`

![image-20211115174448122](C++内存管理.assets/image-20211115174448122.png)



## 重载 operator new

<img src="C++内存管理.assets/image-20211115181543275.png" alt="image-20211115181543275" style="zoom:50%;" />

<img src="C++内存管理.assets/image-20211115181731242.png" alt="image-20211115181731242" style="zoom:50%;" />

### 重载的方式

![image-20211115182349985](C++内存管理.assets/image-20211115182349985.png)

### 重载new() delete()

- 这其中有一个容易混淆的地方是：new() 是placement new接收一个指针参数，但是我们不仅可以放一个指针，其实还可以重载放更多的参数；

- 我们可以重载 class member operator new()，写出多个版本，其中`第一参数必须是 size_t`*（用来接收对象大小）

  <img src="C++内存管理.assets/image-20211116114713862.png" alt="image-20211116114713862" style="zoom:67%;" />

  标准库中的字符串的重载例子：

  <img src="C++内存管理.assets/image-20211116115337045.png" alt="image-20211116115337045" style="zoom:80%;" />

- 我们所重载的 operator delete()，不会被delete调用；只有当new所调用的Ctor抛出exception时，才会调用这些重载的operator delete()，用来完成未能完全创建成功的对象所占用的内存



## new handler

![image-20211116180644894](C++内存管理.assets/image-20211116180644894.png)

其自己书写的例子如下：

![image-20211116181048395](C++内存管理.assets/image-20211116181048395.png)

- 参考[typedefine 相关](./C++基础知识.md)



## 内存池的设计

**目标1：加快速度，一次申请一大块内存池，切成小块供使用**

**目标2：节约空间，每一次申请的一大块内存池带有cookie，每一小块不带cookie**



### 设计方法一：per-class allocaotor，1

<img src="C++内存管理.assets/image-20211116151224355.png" alt="image-20211116151224355" style="zoom:80%;" />

- 上述例子可以看到：`内存池使用链表来管理`

测试程序如下：

<img src="C++内存管理.assets/image-20211116151723980.png" alt="image-20211116151723980" style="zoom:80%;" />



### 设计方法二：per-class allocaotor，2 ★

![image-20211116160002178](C++内存管理.assets/image-20211116160002178.png)

和第一个版本的唯一差别在于，使用了`内嵌指针 embedded pointer` ，使用低位的4个字节来指示freestore，在申请和归还时使用：

![image-20211116160352449](C++内存管理.assets/image-20211116160352449.png)



### 设计方法三：static allocator ★★

更进一步的，为了避免在每一个类里面都要像版本2一样做重载，直接把内存池这一设计抽象出来，设计成一个类

首先其使用的方法如下图：

![image-20211116165706399](C++内存管理.assets/image-20211116165706399.png)

设计如下：

![image-20211116170140163](C++内存管理.assets/image-20211116170140163.png)

其测试结果为：

![image-20211116170511943](C++内存管理.assets/image-20211116170511943.png)

`总结`

​	每个allocator维护一条链表，这个allocator是专属于某个类的。

### 设计方式四：macro for static allocator

- 和版本三原理完全相同

<img src="C++内存管理.assets/image-20211116175849553.png" alt="image-20211116175849553" style="zoom:67%;" />





## malloc概述

### malloc在内存中的基本占用情况

<img src="C++内存管理.assets/image-20211116232709461.png" alt="image-20211116232709461" style="zoom:80%;" />

- 可以看到在申请内存较小的时候浪费很大，都浪费在 cookie 和 overhead 上了

- 因此要考虑去除 cookie，但是要去除 cookie 的先决条件是，每一块的大小都一样，否则怎么能去除；

  于是就考虑到了容器（里面每个元素大小都一样）



### allocator

无论是在VC，BC，还是GNC中，allocator的实现都如同下图所示一样：

![image-20211116234915093](C++内存管理.assets/image-20211116234915093.png)

没有任何的特殊设计，只是调用了 ::operator new ，然后内部调用 malloc，导致每一小块都带有 cookie；

但是不同的是，GNC中的容器虽然实现了allocator，但是容器并没有用它（如上图所示）

`而是用了另一个很棒的类 alloc`，如下图所示：

![image-20211116235456878](C++内存管理.assets/image-20211116235456878.png)

在GNC4.9版本中，这个好东西被换了一个名字，如下图所示：

![image-20211117004007980](C++内存管理.assets/image-20211117004007980.png)





## 第二讲 GNU std::alloc 

### std::alloc 的原理

- 在前面第一讲中，我们自己写的分配器中，让这个分配器变成某一个类的专属的分配器，只维护一条自由链表，专门为它服务
- 现在，把这些自由链表全部收集在一起，一共16条，16种大小，超过最大的alloc将直接调用malloc

![image-20211117112708711](C++内存管理.assets/image-20211117112708711.png)

### std::alloc 运行流程

<img src="C++内存管理.assets/image-20211117165659670.png" alt="image-20211117165659670" style="zoom:80%;" />

<img src="C++内存管理.assets/image-20211117170243769.png" alt="image-20211117170243769" style="zoom:80%;" />

<img src="C++内存管理.assets/image-20211117170759624.png" alt="image-20211117170759624" style="zoom:80%;" />

<img src="C++内存管理.assets/image-20211117171215929.png" alt="image-20211117171215929" style="zoom:80%;" />

<img src="C++内存管理.assets/image-20211117171717530.png" alt="image-20211117171717530" style="zoom:80%;" />

<img src="C++内存管理.assets/image-20211117171833011.png" alt="image-20211117171833011" style="zoom:80%;" />

<img src="C++内存管理.assets/image-20211117171941715.png" alt="image-20211117171941715" style="zoom:80%;" />

<img src="C++内存管理.assets/image-20211117172744302.png" alt="image-20211117172744302" style="zoom:80%;" />

<img src="C++内存管理.assets/image-20211117172856415.png" alt="image-20211117172856415" style="zoom:80%;" />

<img src="C++内存管理.assets/image-20211117173008085.png" alt="image-20211117173008085" style="zoom:80%;" />

接下来，我希望看看把内存用到用光，直到分配失败，会发生什么：

![image-20211117173923077](C++内存管理.assets/image-20211117173923077.png)

<img src="C++内存管理.assets/image-20211117174135739.png" alt="image-20211117174135739"  />

最终，向右也无法找到：

![image-20211117174600458](C++内存管理.assets/image-20211117174600458.png)



### std::alloc 源码剖析

![image-20211117175458702](C++内存管理.assets/image-20211117175458702.png)

所以第一级的代码不再看，

![image-20211117175821228](C++内存管理.assets/image-20211117175821228.png)

第二级开始：重要

![image-20211117181151890](C++内存管理.assets/image-20211117181151890.png)

![image-20211117181635880](C++内存管理.assets/image-20211117181635880.png)

下图是补充的上图

![image-20211117233953396](C++内存管理.assets/image-20211117233953396.png)

其中 `refill()`函数如下图所示：

![image-20211117235348719](C++内存管理.assets/image-20211117235348719.png)

**chunk_alloc()函数的分析：**

![image-20211118000854192](C++内存管理.assets/image-20211118000854192.png)

![image-20211118205115759](C++内存管理.assets/image-20211118205115759.png)

接下来是类外的静态成员变量的定义：

![image-20211118204954412](C++内存管理.assets/image-20211118204954412.png)



### GNU2.9 std::alloc 观念大整理 ★★★

![image-20211118210306723](C++内存管理.assets/image-20211118210306723.png)

![image-20211118211740865](C++内存管理.assets/image-20211118211740865.png)

- **但是，这种设计是有先天缺陷的，也就是之前 malloc() 分配出去的内存的cookie那里已经没有指针了，也就是如果想要 free() 的话是没有指针指向的了**

- **容器调用分配器进行内存分配时， 不会直接使用array new进行内存分配；即使要分配一块大内存，也会进行多次的内存获取再乘起来**

`注意`

<img src="C++内存管理.assets/image-20211122002137111.png" alt="image-20211122002137111" style="zoom:67%;" />

如上图所示：list中的每个节点的大小为 元素double 4 字节 + 2个list指针 8字节 = 16字节



### GNU4.9 中使用 pool allocator 和不使用时间的对比

![image-20211122002931121](C++内存管理.assets/image-20211122002931121.png)





## 第三讲 VC malloc 内存分配

### 总体流程

![image-20211122172638948](C++内存管理.assets/image-20211122172638948.png)



### SBH小区快机制 & 第一次内存分配

![image-20211122211616146](C++内存管理.assets/image-20211122211616146.png)

![image-20211122214815239](C++内存管理.assets/image-20211122214815239.png)

![image-20211122220433549](C++内存管理.assets/image-20211122220433549.png)

![image-20211122222157221](C++内存管理.assets/image-20211122222157221.png)

![image-20211123114745800](C++内存管理.assets/image-20211123114745800.png)

![image-20211124152639081](C++内存管理.assets/image-20211124152639081.png)

![image-20211124153336545](C++内存管理.assets/image-20211124153336545.png)

![image-20211124154051406](C++内存管理.assets/image-20211124154051406.png)

![image-20211124171246621](C++内存管理.assets/image-20211124171246621.png)

### 后续第2次....的内存分配流程

![image-20211124171721729](C++内存管理.assets/image-20211124171721729.png)

### 第一次面对回收的动作

![image-20211124174249716](C++内存管理.assets/image-20211124174249716.png)

### 再一次的分配

![image-20211124174506812](C++内存管理.assets/image-20211124174506812.png)

### 合并

![image-20211125102154103](C++内存管理.assets/image-20211125102154103.png)

所以每一个被malloc分配出去的内存都带着8字节的上下cookie

### 回收时的确认顺序

![image-20211202104637284](C++内存管理.assets/image-20211202104637284.png)



## GNU allocator 和 VC malloc（2+3讲小结）

![image-20211202144430419](C++内存管理.assets/image-20211202144430419.png)

![image-20211202144708258](C++内存管理.assets/image-20211202144708258.png)



## loki::allocator

- 因为GNU allocator的分配器非常霸道，不会还给操作系统内存，因此，介绍下面的 Loki Library下的allocaotr

### 概述：包含有3个重要的classes

![image-20211202163329101](C++内存管理.assets/image-20211202163329101.png)

![image-20211202164103610](C++内存管理.assets/image-20211202164103610.png)

![image-20211202172637454](C++内存管理.assets/image-20211202172637454.png)



## 其他的一些分配器（7种）

![image-20211203152515983](C++内存管理.assets/image-20211203152515983.png)

![image-20211203152532793](C++内存管理.assets/image-20211203152532793.png)

![image-20211203152550828](C++内存管理.assets/image-20211203152550828.png)

### array_allocator

![image-20211203153627304](C++内存管理.assets/image-20211203153627304.png)

### 最精巧的两个分配器总结

![image-20211203172853739](C++内存管理.assets/image-20211203172853739.png)
