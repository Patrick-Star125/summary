## STL总览

### 组成结构

| STL的组成  | 含义                                                         |
| :--------: | :----------------------------------------------------------- |
|    容器    | 一些封装[数据结构](http://c.biancheng.net/data_structure/)的模板类，例如 vector 向量容器、list 列表容器等。 |
|    算法    | STL 提供了非常多（大约 100 个）的数据结构算法，它们都被设计成一个个的模板函数，这些算法在 std 命名空间中定义，其中大部分算法都包含在头文件 <algorithm> 中，少部分位于头文件 <numeric> 中。 |
|   迭代器   | 在 [C++](http://c.biancheng.net/cplus/) STL 中，对容器中数据的读和写，是通过迭代器完成的，扮演着容器和算法之间的胶合剂。 |
|  函数对象  | 如果一个类将 () 运算符重载为成员函数，这个类就称为函数对象类，这个类的对象就是函数对象（又称仿函数）。 |
|   适配器   | 可以使一个类的接口（模板的参数）适配成用户指定的形式，从而让原本不能在一起工作的两个类工作在一起。值得一提的是，容器、迭代器和函数都有适配器。 |
| 内存分配器 | 为容器类模板提供自定义的内存申请和释放功能，由于往往只有高级用户才有改变内存分配策略的需求，因此内存分配器对于一般用户来说，并不常用。 |

### 头文件

| <iterator> | <functional> | <vector>  | <deque>  |
| ---------- | ------------ | --------- | -------- |
| <list>     | <queue>      | <stack>   | <set>    |
| <map>      | <algorithm>  | <numeric> | <memory> |
| <utility>  |              |           |          |

> 按照 C++ 标准库的规定，所有标准头文件都不再有扩展名。以 <vector> 为例，此为无扩展名的形式，而 <vector.h> 为有扩展名的形式，建议使用无扩展名的头文件

#### begin和end

[C++](http://c.biancheng.net/cplus/) 11 标准库增加了 begin() 和 end() 这 2 个函数，和 list 容器包含的 begin() 和 end() 成员函数不同，标准库提供的这 2 个函数的操作对象，既可以是容器，还可以是普通数组。当操作对象是容器时，它和容器包含的 begin() 和 end() 成员函数的功能完全相同；如果操作对象是普通数组，则 begin() 函数返回的是指向数组第一个元素的指针，同样 end() 返回指向数组中最后一个元素之后一个位置的指针（注意不是最后一个元素）。

#### iterator迭代器

## 算法

### sort

C++ STL 标准库中的 sort() 函数，本质就是一个模板函数。该函数专门用来对容器或普通数组中指定范围内的元素进行排序

**速览**

| 函数名                                                     | 用法                                                         |
| ---------------------------------------------------------- | ------------------------------------------------------------ |
| sort (first, last)                                         | 对容器或普通数组中 [first, last) 范围内的元素进行排序，默认进行升序排序。 |
| stable_sort (first, last)                                  | 和 sort() 函数功能相似，不同之处在于，对于 [first, last) 范围内值相同的元素，该函数不会改变它们的相对位置。 |
| partial_sort (first, middle, last)                         | 从 [first,last) 范围内，筛选出 muddle-first 个最小的元素并排序存放在 [first，middle) 区间中。 |
| partial_sort_copy (first, last, result_first, result_last) | 从 [first, last) 范围内筛选出 result_last-result_first 个元素排序并存储到 [result_first, result_last) 指定的范围中。 |
| is_sorted (first, last)                                    | 检测 [first, last) 范围内是否已经排好序，默认检测是否按升序排序。 |
| is_sorted_until (first, last)                              | 和 is_sorted() 函数功能类似，唯一的区别在于，如果 [first, last) 范围的元素没有排好序，则该函数会返回一个指向首个不遵循排序规则的元素的迭代器。 |
| void nth_element (first, nth, last)                        | 找到 [first, last) 范围内按照排序规则（默认按照升序排序）应该位于第 nth 个位置处的元素，并将其放置到此位置。同时使该位置左侧的所有元素都比其存放的元素小，该位置右侧的所有元素都比其存放的元素大。 |

sort() 函数是基于快速排序实现的，因此sort函()函数时间复杂度为$N*log_{2}N$。

需要注意的是，sort() 函数受到底层实现方式的限制，它仅适用于普通数组和部分类型的容器。换句话说，只有普通数组和具备以下条件的容器，才能使用 sort() 函数：

1. 容器支持的迭代器类型必须为随机访问迭代器。这意味着，sort() 只对 array、vector、deque 这 3 个容器提供支持。
2. 如果对容器中指定区域的元素做默认升序排序，则元素类型必须支持`<`小于运算符；同样，如果选用标准库提供的其它排序规则，元素类型也必须支持该规则底层实现所用的比较运算符；
3. sort() 函数在实现排序时，需要交换容器中元素的存储位置。这种情况下，如果容器中存储的是自定义的类对象，则该类的内部必须提供移动构造函数和移动赋值运算符。

值得一提的是，sort() 函数位于`<algorithm>`头文件中，因此在使用该函数前，程序中应包含如下语句：

```C++
#include <algorithm>
```

sort() 函数有 2 种用法，其语法格式分别为：

```C++
//对 [first, last) 区域内的元素做默认的升序排序
void sort (RandomAccessIterator first, RandomAccessIterator last);
//按照指定的 comp 排序规则，对 [first, last) 区域内的元素进行排序
void sort (RandomAccessIterator first, RandomAccessIterator last, Compare comp);
```

其中，first 和 last 都为随机访问迭代器，它们的组合 [first, last) 用来指定要排序的目标区域；另外在第 2 种格式中，comp 可以是 C++ STL 标准库提供的排序规则（比如 std::greater<T>），也可以是自定义的排序规则。

#### 自定义排序

> 关于如何自定义一个排序规则，除了《[C++ STL关联式容器自定义排序规则](http://c.biancheng.net/view/7213.html)》一节介绍的 2 种方式外，还可以直接定义一个具有 2 个参数并返回 bool 类型值的函数作为排序规则。

举个例子：

```C++
#include <iostream>     // std::cout
#include <algorithm>    // std::sort
#include <vector>       // std::vector
//以普通函数的方式实现自定义排序规则
bool mycomp(int i, int j) {
    return (i < j);
}
//以函数对象的方式实现自定义排序规则
class mycomp2 {
public:
    bool operator() (int i, int j) {
        return (i < j);
    }
};
int main() {
    std::vector<int> myvector{ 32, 71, 12, 45, 26, 80, 53, 33 };
    //调用第一种语法格式，对 32、71、12、45 进行排序
    std::sort(myvector.begin(), myvector.begin() + 4); //(12 32 45 71) 26 80 53 33
    //调用第二种语法格式，利用STL标准库提供的其它比较规则（比如 greater<T>）进行排序
    std::sort(myvector.begin(), myvector.begin() + 4, std::greater<int>()); //(71 45 32 12) 26 80 53 33
   
    //调用第二种语法格式，通过自定义比较规则进行排序
    std::sort(myvector.begin(), myvector.end(), mycomp2());//12 26 32 33 45 53 71 80
    //输出 myvector 容器中的元素
    for (std::vector<int>::iterator it = myvector.begin(); it != myvector.end(); ++it) {
        std::cout << *it << ' ';
    }
    return 0;
}
```

程序执行结果为：

12 26 32 33 45 53 71 80

可以看到，程序中分别以函数和函数对象的方式自定义了具有相同功能的 mycomp 和 mycomp2 升序排序规则。需要注意的是，和为关联式容器设定排序规则不同，给 sort() 函数指定排序规则时，需要为其传入一个函数名（例如 mycomp ）或者函数对象（例如 std::greater<int>() 或者 mycomp2()）。

## 顺序容器

### list

**概述**

list 容器，又称双向链表容器，即该容器的底层是以双向链表的形式实现的。这意味着，list 容器中的元素可以分散存储在内存空间里，而不是必须存储在一整块连续的内存空间中。下图是list存储数据的抽象结构。

![](http://1.14.100.228:8002/images/2022/05/26/20220526122202.png)

可以看到，list 容器中各个元素的前后顺序是靠[指针](http://c.biancheng.net/c/80/)来维系的，每个元素都配备了 2 个指针，分别指向它的前一个元素和后一个元素。其中第一个元素的前向指针总为 null，因为它前面没有元素；同样，尾部元素的后向指针也总为 null。

> 基于这样的存储结构，list 容器具有一些其它容器（array、vector 和 deque）所不具备的优势，即它可以在序列已知的任何位置快速插入或删除元素（时间复杂度为`O(1)`）。并且在 list 容器中移动元素，也比其它容器的效率高。
>
> 实际场景中，如何需要对序列进行大量添加或删除元素的操作，而直接访问元素的需求却很少，这种情况建议使用 list 容器存储序列。

而使用 list 容器的缺点是，它不能像 array 和 vector 那样，通过位置直接访问元素。举个例子，如果要访问 list 容器中的第 6 个元素，它不支持`容器对象名[6]`这种语法格式，正确的做法是从容器中第一个元素或最后一个元素开始遍历容器，直到找到该位置。

**使用**

list 容器以模板类 list<T>（T 为存储元素的类型）的形式在`<list>`头文件中，并位于 std 命名空间中。

list 容器有以下几种创建方式

~~~C++
//空容器创建
std::list<int> values;
//容量创建
std::list<int> values(10);
//初始值创建
std::list<int> value1(10, 5);
//拷贝其它list对象，创建list容器
std::list<int> value2(value1);
//拷贝普通数组，创建list容器
int a[] = { 1,2,3,4,5 };
std::list<int> values(a, a+5);
//拷贝其它类型的容器，创建 list 容器
std::array<int, 5>arr{ 11,12,13,14,15 };
std::list<int>values(arr.begin()+2, arr.end());//拷贝arr容器中的{13,14,15}
~~~

**方法**

~~~C++
//老样子那几个容器方法，这里写几个不太常见的
value.unique(); //删除容器中相邻的重复元素，只保留一个。
value.push_back(); //在容器尾部插入一个元素。
value.pop_back(); //删除容器尾部的一个元素。
~~~

**技巧**

~~~C++
//使用迭代器输出list容器中的元素
for (std::list<double>::iterator it = values.begin(); it != values.end(); ++it) {
    std::cout << *it << " ";
}
~~~

## 无序关联式容器

### unordered_set

**概念**

unordered_set 容器和 set 容器很像，唯一的区别就在于 set 容器会自行对存储的数据进行排序，而 unordered_set 容器不会，总的来说，有以下三个特性

1. 不再以键值对的形式存储数据，而是直接存储数据的值；
2. 容器内部存储的各个元素的值都互不相等，且不能被修改；
3. 不会对内部存储的数据进行排序

unordered_set 容器的类模板定义如下：

```
template <class Key, class Hash = hash<Key>, class Pred = equal_to<Key>, class Alloc = allocator<Key>> class unordered_set;
```

| 参数                 | 含义                                                         |
| -------------------- | ------------------------------------------------------------ |
| Key                  | 确定容器存储元素的类型，如果读者将 unordered_set 看做是存储键和值相同的键值对的容器，则此参数则用于确定各个键值对的键和值的类型，因为它们是完全相同的，因此一定是同一数据类型的数据。 |
| Hash = hash<Key>     | 指定 unordered_set 容器底层存储各个元素时，所使用的哈希函数。需要注意的是，默认哈希函数 hash<Key> 只适用于基本数据类型（包括 string 类型），而不适用于自定义的结构体或者类。 |
| Pred = equal_to<Key> | unordered_set 容器内部不能存储相等的元素，而衡量 2 个元素是否相等的标准，取决于该参数指定的函数。 默认情况下，使用 STL 标准库中提供的 equal_to<key> 规则，该规则仅支持可直接用 == 运算符做比较的数据类型。 |

可以看到，以上4个参数中，只有第一个参数没有默认值，这意味着如果我们想创建一个 unordered_set 容器，至少需要手动传递1个参数。事实上，在99%的实际场景中最多只需要使用前3个参数（各自含义如表1所示），最后一个参数保持默认值即可。

> 如果 unordered_set 容器中存储的元素为自定义的数据类型，则默认的哈希函数 hash<key> 以及比较函数 equal_to<key> 将不再适用，只能自己设计适用该类型的哈希函数和比较函数，并显式传递给 Hash 参数和 Pred 参数。

**创建**

创建空的 unordered_set 容器，该容器底层采用默认的哈希函数 hash<Key> 和比较函数 equal_to<Key>。

~~~C++
std::unordered_set<std::string> uset;
~~~

在创建 unordered_set 容器的同时，可以完成初始化操作。比如：

~~~C++
std::unordered_set<std::string> uset{ "http://c.biancheng.net/c/",
                                      "http://c.biancheng.net/java/",
                                      "http://c.biancheng.net/linux/" };
~~~

调用 unordered_set 模板中提供的复制（拷贝）构造函数，完成初始化

~~~C++
std::unordered_set<std::string> uset2(uset);
~~~

C++ 11 标准中还向 unordered_set 模板类增加了移动构造函数，即以右值引用的方式，利用临时 unordered_set 容器中存储的所有元素，给新建容器初始化。

~~~C++
//返回临时 unordered_set 容器的函数
std::unordered_set <std::string> retuset() {
    std::unordered_set<std::string> tempuset{ "http://c.biancheng.net/c/",
                                              "http://c.biancheng.net/java/",
                                              "http://c.biancheng.net/linux/" };
    return tempuset;
}
//调用移动构造函数，创建 uset 容器
std::unordered_set<std::string> uset(retuset());
~~~

如果不想全部拷贝，可以使用 unordered_set 类模板提供的迭代器，在现有 unordered_set 容器中选择部分区域内的元素，为新建 unordered_set 容器初始化。

~~~C++
//传入 2 个迭代器，
std::unordered_set<std::string> uset2(++uset.begin(),uset.end());
~~~

**成员函数**

| 成员方法         | 功能                                                         |
| ---------------- | ------------------------------------------------------------ |
| begin()          | 返回指向容器中第一个元素的正向迭代器。                       |
| end();           | 返回指向容器中最后一个元素之后位置的正向迭代器。             |
| cbegin()         | 和 begin() 功能相同，只不过其返回的是 const 类型的正向迭代器。 |
| cend()           | 和 end() 功能相同，只不过其返回的是 const 类型的正向迭代器。 |
| size()           | 返回当前容器中存有元素的个数。                               |
| max_size()       | 返回容器所能容纳元素的最大个数，不同的操作系统，其返回值亦不相同。 |
| find(key)        | 查找以值为 key 的元素，如果找到，则返回一个指向该元素的正向迭代器；反之，则返回一个指向容器中最后一个元素之后位置的迭代器（如果 end() 方法返回的迭代器）。 |
| count(key)       | 在容器中查找值为 key 的元素的个数。                          |
| equal_range(key) | 返回一个 pair 对象，其包含 2 个迭代器，用于表明当前容器中值为 key 的元素所在的范围。 |
| emplace()        | 向容器中添加新元素，效率比 insert() 方法高。                 |
| emplace_hint()   | 向容器中添加新元素，效率比 insert() 方法高。                 |
| insert()         | 向容器中添加新元素。                                         |
| erase()          | 删除指定元素。                                               |
| clear()          | 清空容器，即删除容器中存储的所有元素。                       |
| swap()           | 交换 2 个 unordered_map 容器存储的元素，前提是必须保证这 2 个容器的类型完全相等。 |
| hash_function()  | 返回当前容器使用的哈希函数对象。                             |

> 注意，此容器模板类中没有重载 [ ] 运算符，也没有提供 at() 成员方法。不仅如此，由于 unordered_set 容器内部存储的元素值不能被修改，因此无论使用那个迭代器方法获得的迭代器，都不能用于修改容器中元素的值。

**用法**

对于顺序结构难以解决的问题，可以用该结构来组成容易操作的数据，例如leetcode128题

> 增：set.insert(data)
>
> 删：set.erase(data|pointer)
>
> 查：set.count(data)

### unordered_map

unordered_map 容器，直译过来就是"无序 map 容器"的意思。所谓“无序”，指的是 unordered_map 容器不会像 map 容器那样对存储的数据进行排序。换句话说，unordered_map 容器和 map 容器仅有一点不同，即 map 容器中存储的数据是有序的，而 unordered_map 容器中是无序的。对于已经学过 map 容器的读者，可以将 unordered_map 容器等价为无序的 map 容器。

unordered_map 容器模板的定义如下所示：

```c++
template < class Key,                        //键值对中键的类型
           class T,                          //键值对中值的类型
           class Hash = hash<Key>,           //容器内部存储键值对所用的哈希函数
           class Pred = equal_to<Key>,       //判断各个键值对键相同的规则
           class Alloc = allocator< pair<const Key,T> >  // 指定分配器对象的类型
           > class unordered_map;
```

以上 5 个参数中，必须显式给前 2 个参数传值，并且除特殊情况外，最多只需要使用前 4 个参数，各自的含义和功能如表 1 所示。

**创建：**老一套

1. 空容器
2. 初始化
3. 内存复制，注意，无论是调用复制构造函数还是拷贝构造函数，必须保证 2 个容器的类型完全相同
4. 迭代器复制

**用法**



















