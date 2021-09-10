# C++标准库学习笔记

#### C++内存空间

一个C++程序编译后占用的内存分为如下几个部分：

|                                            |                                                              |
| :----------------------------------------: | ------------------------------------------------------------ |
|              **栈（Stack）**               | 由编译器自动分配释放，存放函数的参数值，局部变量的值。在一个进程中，位于用户虚拟地址空间顶部的是用户栈，编译器用它来实现函数的调用 |
|               **堆（Heap）**               | 由程序员分配和释放，若程序员不释放，则程序结束时被OS回收。存放由new，malloc分配的内存，可动态扩展和收缩 |
| **全局初始化区（静态区）（Data Segment）** | 全局变量和静态变量的存储是放在一起的，初始化的全局变量和初始化的静态变量在一块区域；在程序运行之初就为数据段申请了空间，**程序退出的时候释放空间，其生命周期是整个程序的运行时期。**未初始化的全局变量和未初始化的静态变量在相邻的另一块区域。 |
|         **未初始化数据区（BSS）**          | 用来存放程序中**未初始化的全局变量和静态变量**，位置在数据段之后，可以不相连。**其生命周期和数据段一样。**（2）和（3）统称为静态存储区。 |
|       **程序代码区（text segment）**       | 存放函数体的二进制代码                                       |

const修饰的全局变量保存在代码区中，const修饰的局部变量保存在栈段中。



##### 程序样例

```c++
//main.cpp
int a = 0;                     //全局初始化区
char *p1;                      //全局未初始化区

int main()
{
  int b;                       //栈区
  char s[] = "abc";            //栈区
  char *p2;                    //栈区
  char *p3 = "123456";         //123456在常量区，指针p3在栈上。
  static int c =0；            //全局（静态）初始化区
  p1 = (char *)malloc(10);
  p2 = (char *)malloc(20);
  //分配得来得10和20字节的区域就在堆区。
  strcpy(p1, "123456");      
  //123456放在常量区，编译器可能会将它与p3所指向的"123456"优化成一个地方。
}
```



#### 二分查找

二分查找也称折半查找（Binary Search），它是一种效率较高的查找方法。但是，折半查找要求线性表必须采用[顺序存储结构](https://baike.baidu.com/item/顺序存储结构/1347176)，而且表中元素按关键字有序排列。

1. 必须采用[顺序存储结构](https://baike.baidu.com/item/顺序存储结构)。
2. 必须按关键字大小有序排列。

##### 算法

```c++
int bsearchWithoutRecursion(int array[],int low,int high,int target)
{
    while(low<=high)
        {
            int mid=low+(high-low)/2;/*使用(low+high)/2会有整数溢出的问题
            （问题会出现在当low+high的结果大于表达式结果类型所能表示的最大值时，
                这样，产生溢出后再/2是不会产生正确结果的，而low+((high-low)/2)
                不存在这个问题*/
            if(array[mid]>target)
                high=mid-1;
            else if(array[mid]<target)
            low=mid+1;
            else
                return mid;
        }
    return-1;
}
```



##### 程序样例

```c++
const size_t asize = 500000;
cout << "\n tesr array()......... \n";
static array<long, asize> c; // static 会在Data Segment区申请变量，不加的话会在栈区申请，有2M大小限制

for (long i = 0; i < asize; ++i) {
    c[i] = rand();
}

long target = get_a_target_long();

clock_t timeStar = clock();

// 第一步排序
qsort(c.data(), asize, sizeof(long), compareLongs); // c 标准库函数qsort()
// 第二部二分法查找
long* pItem = (long*)bsearch(&target, c.data(), asize, sizeof(long), compareLongs);// c 标准库函数 bsearch()

cout << "qsort() + bsearch(), milli-seconds: " << clock() - timeStar << endl;
if (pItem != NULL)
    cout << "found" << *pItem << endl;
else
    cout << "not found!" << endl;
```

------



#### sort()与qsort()区别

在C++的STL里面有sort与qsort可以直接用于对各种类型的数据以及容器进行排序，都定义在头文件<algorithm>中。

1.qsort

用 法: 

```c++
void qsort(void *base, int nelem, int width, int (*fcmp)(const void *,const void *)); 　　 

```

参数：1、待排序数组首地址； 2、数组中待排序元素数量； 3、各元素的占用空间大小； 4、指向函数的指针，用于确定排序的顺序 
比如：对一个长为1000的数组进行排序时，int a[1000]; 那么base应为a，num应为 1000，width应为 sizeof(int)，cmp函数随自己的命名。
2.sort

sort是qsort的升级版，如果能用sort尽量用sort，使用也比较简单，不像qsort还得自己去写 cmp 函数，只要注明 使用的库函数就可以使用，参数只有两个（如果是普通用法）头指针和尾指针； 
默认sort排序后是升序，如果想让他降序排列，可以使用自己编的cmp函数 

```c++
bool compare(int a,int b) 
{ 
	return a>b; //www.cdtarena.com降序排列，如果改为return a<b，则为升序
} 
sort(*a,*b,cmp);
```

3、sort与qsort的对比

（1）最直观的差别，函数形式不一样，

qsort的使用方式为：

void qsort( void *base, size_t num, size_t width, int (__cdecl *compare ) 

sort的使用方式为：

template <class RandomAccessIterator>

void sort ( RandomAccessIterator first, RandomAccessIterator last )；

template <class RandomAccessIterator, class Compare>

void sort ( RandomAccessIterator first, RandomAccessIterator last, Compare comp );

sort有二个参数与三个参数版本，两个参数默认升序排序，第三个参数可用于指定比较函数，调整排序方式。

（2）compare函数的写法也是不一样的。

qsort的compare函数写法为：

int compare (const void *elem1, const void *elem2 ) );

sort的compare函数返回的是bool值；

（3）sort是一个改进版的qsort. std::sort函数优于qsort的一些特点：对大数组采取9项取样，更完全的三路划分算法，更细致的对不同数组大小采用不同方法排序。如果能用sort尽量用sort，使用也比较简单，不像qsort还得自己去写 cmp 函数，只要注明 使用的库函数就可以使用，参数只有两个（如果是普通用法）头指针和尾指针；默认sort排序后是升序，如果想让他降序排列，可以使用自己编的cmp函数 





#### nullptr 而不是 NULL？

##### **前言**

在C语言中，我们常常用NULL作为指针变量的初始值，而在C++中，却不建议你这么做。

##### **NULL是什么**

在《[NULL,0,'\0',"0","\0"的区别](http://mp.weixin.qq.com/s?__biz=MzI2OTA3NTk3Ng==&mid=2649284887&idx=1&sn=e97526b0e2cbe6a739c9783ddb38cd92&chksm=f2f99270c58e1b66738cba8102f64eec72730d0a91a1bb7b39cad9c9eda7939e8043a2a3c089&scene=21#wechat_redirect)》一文中，我们已经知道了在C中NULL是什么，在C的头文件中，通常定义如下：

```c++
#define NULL ((void*)0)
```

但是在C++中，它是这样定义的：

```c++
#define NULL 0
```

或者你可以在stddef.h看到完整的这段：

```c++
#undef NULL
#if defined(__cplusplus)
#define NULL 0
#else
#define NULL ((void *)0)
#endif
```

也就是说，在C++中，NULL不过也是0罢了，把它当成空指针只是一个无可奈何的选择罢了。

那么为什么在C++和C中不一样呢？因为C++中不能将void *类型的指针**隐式转换**成其他指针类型，从下面的例子可以看出来：

```c++
//null.cpp
#include<iostream>
int main(void)
{
    char p[] = "12345";
    int *a = (void*)p;
    return 0;
}
```

编译：

```c++
$ g+ -o null null.cpp
null.cpp: In function 'int main()':
null.cpp:5:17: error: invalid conversion from 'void*' to 'int*' [-fpermissive]
  int *a =(void*)p;
```

所以不能将NULL定义为(void*)0。

##### **nullptr**

nullptr并非整型类别，甚至也不是指针类型，但是能**转换成任意指针类型**。nullptr的实际类型是std:nullptr_t。

> 来源：公众号【编程珠玑】，专注但不限于分享计算机编程基础，Linux，C语言，C++，数据结构与算法，工具，资源等编程相关[原创]技术文章。博客：https://www.yanbinghu.com/2019/08/25/36794.html

##### **为什么该使用nullptr**

回到最开始的问题，为什么作为指针的语义，我们应该使用nullptr，而不是NULL。 请看下面的代码：

```c++
//来源：公众号【编程珠玑】，https://www.yanbinghu.com
//test.cpp
#include<iostream>
using namespace std;
void test(void *p)
{
    cout<<"p is pointer "<<p<<endl;
}
void test(int num)
{
    cout<<"num is int "<<num<<endl;
}
int main(void)
{

    test(NULL);
    return 0;
}
```

编译：

```c++
$ g++ -o test test.cpp
main.cpp: In function ‘int main()’:
main.cpp:16:14: error: call of overloaded ‘test(NULL)’ is ambiguous
     test(NULL);
```

很不幸，编译报错了，提示我们有二义性，按照《[重载函数匹配规则](http://mp.weixin.qq.com/s?__biz=MzI2OTA3NTk3Ng==&mid=2649284296&idx=1&sn=7bbce919d0c7f2f6f62623d7054a8dbc&chksm=f2f9adafc58e24b9db5a46249d0a3364265065e075f0e6066ee0e8a064aefc8c23ab3710a826&scene=21#wechat_redirect)》，两个都可以匹配，因此最终报错。

但是如果我们使用nullptr却不会：

```c++
test(nullptr);
```

除了这点之外，在C++模板中它还有更好的表现。 看下面的代码：

```c++
//来源：公众号【编程珠玑】，https://www.yanbinghu.com
#include<iostream>
using namespace std;
template<typename Type1,typename ptrType>
void test(Type1 fun,ptrType ptr)
{
    /*do something*/
    fun(ptr);
    return;
}
void fun(int *val)
{
    cout<<"fun"<<endl;
}
int main(void)
{
    test(fun,NULL);
    return 0;
}
```

编译报错了：

```c++
main.cpp:8:8: error: invalid conversion from ‘long int’ to ‘int*’ [-fpermissive]
     fun(ptr);
```

很显然NULL被推导为long int，而不是空指针，因而导致函数类型不匹配而报错。

但是如果我们用nullptr就不会有上面的问题。

##### **总结**

如果你想表示空指针，那么使用nullptr，而不是NULL。

注：nullptr在C++ 11中才出现。