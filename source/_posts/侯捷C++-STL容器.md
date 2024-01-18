---
title: 侯捷C++-STL容器
date: 2023-12-23
index_img: "/img/bg/cpp.jpg"
tags: [C++]
categories: 
   -[C++]
---
<!-- more -->

# 容器，结构与分类

![](/article_img/2023-12-23-09-42-36.png)
容器可以分为两种类型，**序列式容器**和**关联式容器**，序列式中存储的就是单一的元素，关联式容器中存储的是一个键值对，并默认会根据个元素的键值大小做排序。
这里的符合是指如 **set** 拥有 **rb_tree**，STL的思想是组合优于继承，较少用到继承。容器是一个类，因此容器本身的大小就是容器类的大小，与其中储存的数据无关；

**所有的容器都是前闭后开区间！**

# 容器的使用场景

容器名称|概述|时间复杂度|优点|缺点|总结
---|---|---|---|---|---
**vector**|**动态数组**；在内存中连续存储，内存不够时调用**malloc**在堆分配二倍内存，进行拷贝，并调用**free**释放原内存|**尾插/删**：O(1)；**头/中间插/删**：O(N)；**查找（无序）**：O(N)；**查找（有序二分查找）**：O(logN)|支持随机访问|非末尾插入删除效率低|适合需要经常随机访问，并且不需要堆中间元素插入删除的情况
**list**|**双向循环链表**；内存空间不连续|**任何位置插/删**：O(1)；**查询**：O(N)|内存不连续，在任何位置插/删效率高|不能随机访问|适用于经常插入删除并不需要随机访问的情况
**deque**|**双向队列**；内部用vector作为控制中心，用buffer（一段定长的连续内存）存储数据|**头/尾插/删**：O(1)；**中间插/删：O(N)**；**查找**：O(N)；|支持随机访问（效率比vector差），在首尾操作效率很高|非首尾操作效率低|适用于需要随机访问，且需要对首尾频繁操作的情况
**set/multiset**|是一种容器适配器，内部由**红黑树rb_tree**（平衡二叉树）实现，内部元素自动排序|**增删改查**：O(logN)|便于元素查找，支持自动排序|每次插入元素要调整红黑树，对效率有一定影响|适用于需要经常查找一个元素是否在某集群中的情况
**map/multimap**|是一种容器适配器，内部由**红黑树rb_tree**（平衡二叉树）实现，有键值对，按照key自动排序|**增删改查**：O(logN)|便于查找，可以创建字典|每次插入元素要调整红黑树，对效率有一定影响|适用于需要字典的情况
**unordered_set/unordered_map**|是容器适配器,内部由**哈希表hashtable**实现，元素无序|**增删改查（理论上）**：O(1)；**最坏情况**：O(N)|理论上增删改查极快|需要较大内存是以空间换时间，插入数据是否均匀影响其性能|适用于一些特定算法，或者数据很分散的情况

# 容器list

list是STL提供的双向链表；**擅长在任何位置插入或删除元素 O(1)**，不能随机访问；

## list用法：
```C++
#include<list>
using namespace std;
int main(){
    list<int> L;
    L.push_back(7); // 在链表后插入
    L.push_front(4); // 在链表头插入
    L.push_back(5);
    L.sort(); // 链表需要用自带的sort
    /*迭代器用法，使用上和指针相同*/
    list<int>::iterator ite;
    for(ite = L.begin();ite!=L.end();ite++){
        cout<< *ite << "\n";
    }
    /*C++11新用法，很方便*/
    for(auto i : L){
        cout<< i << "\n";  
    }
}
```

## list底层（G2.9）

侯捷老师使用G2.9编译器的list底层实现作为范例讲解list实现原理，G2.9虽然很古老，但是胜在实现简单，逻辑清晰，很有利于理解实现思路。

![](/article_img/2023-12-23-09-46-06.png)

**list** 在底层是一个 **环状双向链表**，并有一个空白节点，用来满足STL规定的容器应该是 **前闭后开** 区间；STL中的数据结构为了搭配迭代器都比我们自己随便写的链表要复杂一些，链表节点（我们的写法，有数据，有两个指针实现双向链表）在 **__list_node** 中，**__list_node** 与 **list** 类是组合关系，同时还在 **list** 类中定义了迭代器（也是组合关系）；

## list的iterator

值得关注的是迭代器，list的迭代器是一个**希望看上去像一个指针**的类，需要完成 ++ -- 等之指针能完成的操作，所以在迭代器的定义中，有很多操作符重载：
![](/article_img/2023-12-23-11-15-14.png)
在G4.9，对迭代器的写法进行了一些优化，传递的参数只是模板 _Tp；也对链表节点做了优化，直接将指针类型设为本身（我们平时也这么实现），同时用了继承把数据和指针分开；
![](/article_img/2023-12-23-11-18-43.png)

## forward_list（单向链表）

![](/article_img/2023-12-24-13-21-11.png)

# 容器vector

vector在使用上就是一个可变长的数组，**擅长从尾部添加和删除元素 O(1)**，

## vector用法
```C++
#include<list>
using namespace std;
	vector<int> v;
	v.push_back(1);
	v.push_back(2);
	vector<int>::iterator it;
	for (it = v.begin(); it != v.end(); it++) {
		cout << *it << "\n";
	}
	cout << "-------------------" << "\n";
	vector<int>::iterator ite;
	for (ite = v.begin(); ite != v.end(); ite++) {
		cout << *ite << "\n";
	}
	cout << "-------------------" << "\n";
	for (int i : v) {
		cout << i << "\n";
	}
```
## vector的iterator

![](/article_img/2023-12-23-18-55-29.png) | ![](/article_img/2023-12-23-18-54-08.png)
---|---

vector的iterator就是一个 **T*** 指向T的指针，这是因为对于vector这种在连续内存空间存储的使用指针就完全足够了；

## vector实现

![](/article_img/2023-12-23-17-17-00.png)

vector本身的大小就是三个iterator，也就是三个指针，分别是**start（记录开头位置），finish（记录末尾位置），end_of_storage（记录当前空间的大小也叫容量）**；

vector的实现简单来说就是 **申请二倍空间，之后把当前vector的元素挨个拷贝过去，再插入新元素，并删除原本的元素**；因此，一旦vector的空间增长，就会涉及大量的**拷贝构造函数**和**析构函数**，这是使用vector时要考虑到的开销；

# 容器array

## array的用法

```C++
#include<array>
using namespace std;
int main() {
	array<int, 10> myArray;
	auto ite = myArray.begin();	
	// array<int, 10>::iterator ite = myArray.begin();
	for (ite; ite != myArray.end(); ite++) {
		cout << *ite;
	}
}
```
## array的iterator

array的迭代器也是一个**指针**；

## array的实现

array就是要模拟语言中基础的**数组**，所以其实现就是一个数组，专门为语言中自带的数组写一个容器是**为了与算法配合**；

# 容器deque

deque 是 double-ended queue 的缩写，又称双端队列容器。**擅长从首尾添加和删除元素 O(1)**，不擅长从队列中添加和删除；

## deque的用法

```C++
#include<deque>
using namespace std;
int main(){
    deque<int> d1; // 初始化空deque
    deque<int> d2(10); // 初始化一个空间为10的deque，元素默认为0
    deque<int> d3(10, 5); // 空间为10，元素为5
    d1.push_back(1); // {1}
    d1.push_front(2);  // {2, 1}
    for(auto i = d1.begin(); i!=d1.end();i++){
        cout << *i << "\n";
    }
}
```

## deque的iterator

![](/article_img/2023-12-24-14-02-17.png)
deque的iterator比较复杂，是一个类，其中有四个指针，分别表示**当前位置**，**当前位置所在buffer的首尾位置**，**在map中的位置**；

## deque的实现

![](/article_img/2023-12-24-13-54-31.png)
deque的实现如上图所示，deque对外声称自己是连续存储的，但其实是由多段 **连续内存（buffer）** 拼起来的，**map** 作为索引（控制中心），用指针负责将多段**buffer**拼起来，**map**本身是一个 **vector**；当向前或者向后插入元素且内存不够时，会申请一个 **buffer**，并用将buffer的头指针插入 **map** 的前端或尾端； 
![](/article_img/2023-12-24-14-48-21.png)
上面提到，**map** 是用 **vector** 实现的，且map需要在头部插入数据，vector本身不擅长头插，因此在vector扩充内存时，**会将现有数据拷贝到新申请的内存的中部**，减少头插出现的概率；

**deque类实现**：
![](/article_img/2023-12-24-14-06-04.png)

### deque如何模拟连续空间
deque利用其iterator类中的各种操作符重载模拟连续空间；
![](/article_img/2023-12-24-14-23-48.png)
![](/article_img/2023-12-24-14-14-10.png)
![](/article_img/2023-12-24-14-24-46.png)
![](/article_img/2023-12-24-14-24-58.png)

## queue队列

**queue的实现完全基于deque**，queue类中有一个成员变量是deque，所有的函数都交给这个deque成员变量操作；因此有些人也把queue不叫做一个容器而叫做**适配器（adapter）**
![](/article_img/2023-12-24-14-52-58.png)

## stack栈

**stack的实现也是完全基于deque**：
![](/article_img/2023-12-24-14-55-50.png)

**stack和queue都不允许遍历，也不提供iterator！** 因为栈和队列都不能被直接访问元素，应该按照对应的先进后出和先进先出访问元素。

# 容器rb_tree

接下来要介绍的就是**关联式容器**，在STL中有**map，multimap，set，multiset，unordered_map，unordered_set，unordered_multimap，unordered_multiset；** 其中前面几种不带unordered的前缀的底层实现都是**容器rb_tree**，所以它们都可以称为**容器适配器**。

![](/article_img/2023-12-25-10-01-22.png)

红黑树是一种**平衡二叉树**，它可以自动排序，这里红黑树的原理（如何排序，涉及各种旋转操作）就先不赘述了，rb_tree提供了迭代器，但是我们不应该用迭代器改变元素值，rb_tree提供了迭代器是为了作为set和map的底部支持。
rb_tree可以直接使用，但是其实没必要，我们直接使用set和map即可，下面是使用的用例：
![](/article_img/2023-12-25-10-05-39.png)

## set、multiset

set和multiset的 **键（key）** 和 **数据（data）** 合二为一，set不允许key重复，也就是set的元素不能重复（使用tb_tree的 **insert_unique()**），multiset允许元素重复（使用tb_tree的 **insert_equal()**）；

```C++
template < class T,                        // 键 key 和值 value 的类型
            class Compare = less<T>,        // 指定 set 容器内部的排序规则
            class Alloc = allocator<T>      // 指定分配器对象的类型
            > class set;
```

### set、multiset的实现

![](/article_img/2023-12-25-10-12-17.png)
set的底层实现完全使用红黑树，但是由于set的键值合二为一，因此set的迭代器不能更改值，这里set类使用**const迭代器**确保迭代器指向的键值对不会被改变。

## map，multimap

map和multimap就是很明显的关联式容器了，其 **键（key）** 和 **数据（data）** 分离，与set、multiset类似，multimap允许 **键（key）** 重复。

```C++
template < class Key,                                     // 指定键（key）的类型
           class T,                                       // 指定值（value）的类型
           class Compare = less<Key>,                     // 指定排序规则
           class Alloc = allocator<pair<const Key,T> >    // 指定分配器对象的类型
           > class map;
```

### map，multimap的实现

![](/article_img/2023-12-25-10-18-41.png)

map的 **键（key）** 和 **数据（data）** 分离，所以我们当然应该可以更改每个元素的**数据**，因此map的迭代器就不能简单的设置为const iterator了，STL 提供了 **pair 类模板**，其专门用来将 2 个普通元素 **first** 和 **second**（可以是 C++ 基本数据类型、结构体、类自定的类型）创建成一个新元素 **<first, second>**。用法如下：
```C++
int main(){
    pair<string, double> pair1;
    pair<string, string> pair2("hello", "world");
    cout<< pair1.first << " " << pair2.second << "\n";
}
```
map的 **operator[]** 有独特的设计：
![](/article_img/2023-12-25-10-31-24.png)

# 容器hashtable

![](/article_img/2023-12-25-14-58-29.png)

hash表的思路是：先申请一个bucket，为每个元素计算一个编号，根据这个编号和bucket大小的余数将元素放入表中，如果发生碰撞就用链表将碰撞的元素串起来；但是当碰撞太多时，链表过长，查找效率会严重降低，因此当链表过长时（经验上是**元素**个数多于bucket时），**将bucket扩为原来的二倍**并调整为**质数**（可见bucket用vector实现很好），重新计算每个元素应该放在哪里，这个过程也叫**rehashing**。
```C++
static const int _stl_num_primes=28;
static const unsigned long _stl_prime_list[_stl_num_primes]={ // STL将扩容的大小硬编码
    53,97,193,389,769,1543,3079,6151,12289,24593,1572869,3145739,6291469...
}
```

## hashtable的iterator

![](/article_img/2023-12-25-15-11-25.png)

hashtable的迭代器要记录当前指向的链表节点（hashtable_node），也要能在走到一个链表尽头的时候返回bucket走到下一个链表，因此还需要一个指针指向bucket（hashtable* ht）；

hashtable要用的时候不光要给出**元素的类型（class Value）**，**键的类型（class Key）**，**怎样取出键的函数（class ExtractKey）**，**怎样判断键相等（class EqualKey）** 这些在rb_tree中要给出的模板参数，还要给出一个 **hashFuc** 用来计算 **hash_code** 从而将元素填入哈希表。这个hashFunc在我们使用的时候也要手动给出，不过好在STL已经提供了一些常见类型的hashFunc，我们定义自己的类的hashFunc时可以使用这些hashFunc的和：
![](/article_img/2023-12-25-15-52-45.png)

## unordered_set、unordered_multiset

```C++
template < class Key,            //容器中存储元素的类型
            class Hash = hash<Key>,    //确定元素存储位置所用的哈希函数
            class Pred = equal_to<Key>,   //判断各个元素是否相等所用的函数
            class Alloc = allocator<Key>   //指定分配器对象的类型
            > class unordered_set;
```
unordered_set、unordered_multiset这两种容器与set、multiset的区别就在于**无序**，set、multiset底层是红黑树实现，中序遍历可以输出有序元素；

## unordered_map、unordered_multimap

```C++
template < class Key,                        //键值对中键的类型
            class T,                          //键值对中值的类型
            class Hash = hash<Key>,           //容器内部存储键值对所用的哈希函数
            class Pred = equal_to<Key>,       //判断各个键值对键相同的规则
            class Alloc = allocator< pair<const Key,T> >  // 指定分配器对象的类型
            > class unordered_map;
```
unordered_map、unordered_multimap这两种容器与map、multimap的区别就在于**无序**，map、multimap底层是红黑树实现，中序遍历可以输出有序元素；

