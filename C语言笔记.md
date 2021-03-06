# C语言笔记
> wvv 20200315

[toc]
### 数据类型
#### 常见数据类型

在不同类型系统上，不同类型所占宽度不同，在32位系统上

* 整型

| char           | 1 字节      | -128 到 127 或 0 到 255                              |
| -------------- | ----------- | ---------------------------------------------------- |
| unsigned char  | 1 字节      | 0 到 255                                             |
| signed char    | 1 字节      | -128 到 127                                          |
| int            | 2 或 4 字节 | -32,768 到 32,767 或 -2,147,483,648 到 2,147,483,647 |
| unsigned int   | 2 或 4 字节 | 0 到 65,535 或 0 到 4,294,967,295                    |
| short          | 2 字节      | -32,768 到 32,767                                    |
| unsigned short | 2 字节      | 0 到 65,535                                          |
| long           | 4 字节      | -2,147,483,648 到 2,147,483,647                      |
| unsigned long  | 4 字节      | 0 到 4,294,967,295                                   |

* 浮点型

| float       | 4 字节  | 1.2E-38 到 3.4E+38     | 6 位小数  |
| ----------- | ------- | ---------------------- | --------- |
| double      | 8 字节  | 2.3E-308 到 1.7E+308   | 15 位小数 |
| long double | 16 字节 | 3.4E-4932 到 1.1E+4932 | 19 位小数 |

* void类型

 void 类型表明没有可用值。 

* void*类型

 void *类型，代表任何类型的指针，可以强制转换为任何指针类型。 

#### 数组

##### 多维数组

`int  a[2][2][3]={{{1},{2}},{{3},{4}}};`

也可以用下标进行初始化

`int  a[2][2][3]={1,[0][1][0]=5,[1][1][2]=8};`

也可以用下标进行初始化

`int  a[2][2][3]={{1},[0][1]={5,2},[1][0]={2,4}};`

##### 指针数组

指针数组的实质就是一个数组，数组中存的所有元素是指针变量。 

` int *p[5]; `

因为运算符[]优先级比 * 要高，所以p先和[]结合，所以p是一个数组，指向的类型是int *

所以`sizeof(p)` 会得到5倍int的大小。


##### 数组指针

数组指针的实质是一个指针，指向的是一个数组。 

` int (*p)[5];`

因为运算符()所以p先和 * 结合，所以p是一个指针，指向了一个int数组。

所以`sizeof(p)`会得到指针自身长度。

#### 结构体

* 声明

```c
struct St
{
    int a;
    char b;
};
struct St st={1,'a'};
```

c语言中，每次使用结构体都需要带struct前缀，可以使用typedef来简化

```c
typedef struct
{
    int a;
    char b;
}  St;
```

上面实际上利用typedef声明了一个匿名结构体，然后加上struct前缀合成了St，以后所有使用结构体的地方都不需要加Struct的前缀。

虽然采用typedef的的方式简化了书写，但是一般不推荐这种做法，某些特殊的情况下容易引起混乱。

* 初始化

```c
struct St st1={1,'a'};
struct St st2={.a = 1};
```

上面提供了两种不同方式的初始化

* 嵌套

结构体内部可以包含其他结构体，也可以包含自身的结构体指针，这个特性是链表等数据结构的基础。

```c
struct Node
{
    int a;
    struct Node *next;
};
```

* 不完整声明

如果两个结构体需要嵌套，则需要对其中一个进行不完整声明

```c
struct B;
struct A
{
    struct B *p;
};
struct B
{
    struct A *p;
};
```

* 结构体作为参数

可以通过传值或传指针的方式将结构体传入函数

```c
void f1( struct St s );
void f2( struct St *s );
```

结构体数组通过指针的方式进行参数传入。

* 位域

位域或者位段，是把一个字节划分成不同的区域，达到节省空间的一种手段，这在硬件资源匮乏的嵌入式领域是一种常用的技巧。位域宽度不能大于8。

```c
struct St{
    int a:4;
    int b:2;
    int c:2;
}
```

上述结构体实际上只占用了一个int的大小。

```c
struct St{
    int a:4;
    int  :2;
    int c:2;
}
```

位域中的变量可以是无名位域，这些空间会默认填充为0，一般起到填充对齐的作用，上述代码进行了演示。

#### 枚举

枚举一般用于常量的定义，可以增加代码可读性。

```c
enum Day
{
      MON=1, TUE, WED, THU, FRI, SAT, SUN
};
enum Day day;
```

枚举可以定义成不连续分配

```c
enum Ms
{
    M1,
    M2 = 10,
    M3
};
```

整形转换成枚举时需要进行强制转换。

#### 字符串
#### 指针
指针是一个变量，其值为另一个变量的地址
NULL 指针是一个定义在标准库中的值为零的常量，为指针变量赋一个 NULL 值是一个良好的编程习惯。
##### 指针算术运算
可以对指针进行四种算术运算：++、--、+、-。
指针可以用关系运算符进行比较，如 ==、< 和 >。
指针在递增和递减时地址的变化取决于指针所指向变量数据类型长度。
##### 函数指针
函数指针是指向函数的指针变量。
`int (* p)(int, int);`
声明了带两个int参数的返回值也为int的函数指针p，注意小括号将 * 和 p 括起来。
可以通过 `typedef`来定义函数指针的别名，可以简化书写。
```c
typedef int (* pF)(int, int);
pF fun;
```
可以将函数指针传入函数，从而构成回调函数。
```c
int add(int a, int b)
{
    return a+b;
}
int calc(pF f, int a, int b)
{
    return f(a, b);
}
calc(add, 1, 2);
```
函数指针数组可以如下声明
```c
int (* pF[10])(int, int);
```
或者
```c
typedef int (* pF)(int, int);
pF fun[10];
```
很明显后者更清晰，方便。
##### 数组名与指向数组的指针的关系

* 相同点：

数组名和指向数组的指针都指向的是数组中第一个元素的地址。

都可以通过下标或者指针的方式访问。

* 不同点

数组名是常量指针，不能再次赋值。

普通指针是变量指针，可以随时修改指向。

对数组名利用&取地址时等价于不用&符号，意味着&符号对数组名操作时可以省略。

对指针利用&取地址时得到的是指针自身存储的地址。

数组名进行加1操作时，得到的偏移量是整个数组的大小。

对指针进行加1操作时，得到的偏移量只是单个元素的大小。

* 示例代码

```c
#include <stdio.h>
int main() {
    int a[]={1,2,3};
    int *pa =a;
    printf("%p %p %p\n",a,&a,&a[0]);
    printf("%p %p\n",pa,&pa);
    printf("%p %p\n",&a+1,pa+1);
    printf("%d %d %d\n",sizeof(a),sizeof(pa),sizeof(*pa));
    return 0;
}
```

**输出结果**

```
000000000065FE14 000000000065FE14 000000000065FE14
000000000065FE14 000000000065FE08
000000000065FE20 000000000065FE18
12 8 4
```

**结果分析**

`printf("%p %p %p\n",a,&a,&a[0]);`

输出内容都为 000000000065FE14 说明对数组名来讲，a,&a,&a[0]表示的都是第一个元素的地址

`printf("%p %p\n",pa,&pa);`

输出内容pa,&pa不同，分别表示了指针其指向的地址和自身的地址。

`printf("%p %p\n",&a+1,pa+1);`

输出内容相对元素首地址来讲，&a+1 增加了12字节，pa+1增加了4字节，这个是跟他们指向内容表示大小有关，他们指向内容的大小可以通过sizeof得到，结合下句

初始声明

```c
    int a[]={1,2,3};
    int *pa =a;
```

以及 `printf("%d %d %d\n",sizeof(a),sizeof(pa),sizeof(*pa));`输出12 8 4，表明了：

1. sizeof(a)得到了数组常量指向的宽度，是整个数组的大小
2. sizeof(pa)表示的是指针pa自身的大小（我的系统是64位，所以指针类型都是64位即8字节大小）
3. sizeof(*pa)表示的是指针所指向的内容的大小，即第一个int型元素的大小

##### const 指针

参考 https://www.cnblogs.com/bencai/p/8888760.html 

请看下面几种情况

```c
const int *p;
int const *p;
int * const p;
const int * const p;
```

上面的p有的是指向常量的指针（指向的内容不可以修改，但是可以指向不同的位置），

有的是不可变指针（就是自身不能重新再赋值指向其他地址），

区分的办法，从右往左读，遇到* 替换为 point to，

所以上面四行p可以翻译为：

p point to int const

p point to const int

p const point to int

p const point to int const

其中int const 和const int等价，所以上面第一、二行等同，表示p是一个指向常量的指针。指向的内容不可改变。

第三行表示p是一个不可变指针指向int。不可指向不同内容。

第四行表示p是一个不可变指针指向一个常量。不可指向不同内容，指向的内容也不可改变。

下面给出个测试代码

```c
int main() {
    int a=10,b=20;
    const int *p1=&a;
    int *const p2=&a;
    //*p1=20;
    p1=&b;
    *p2=20;
    //p2=&b;
    return 0;
}
```

根据前面的分析，代码中我们定义了p1指针指向一个常量a，不可变指针p2指向一个int变量，所以

注释掉的第一处代码`*p1=20;`会无法通过编译，因为修改了p1指向位置的值，但是p1指向的应该是个常量。

注释掉的第二处代码`p2=&b;`也无法通过编译，因为p2是个不可变指针，但是又重新指向了不同的位置。

### 可变参数

c语言中，可以通过三个点 ... 的方式声明参数，实现可变个数参数的输入。

像printf函数就是用这种方式实现的。

需要包含头文件 `#include <stdarg.h>`

```c
int f(int num, ... ) 
{
   ...
}
 
```

 **va_list**  

 **va_start**  

 **va_arg**  

 **va_end**  

需要用到上述几个宏。

c语言中需要用到此技巧的地方不多，真的有这个需求的时候再自行查阅更详细资料。

宏中可变参数传递示例代码：

```c
void h_log(const char *fmt, ...);

#ifndef DBG_NAME
#define DBG_NAME    DBG_TAG
#endif
#define _log_line(lvl, fmt, ...)                					\
    do{                                           					\
    	h_log("["lvl"/"DBG_NAME"\t]"fmt"\n", ##__VA_ARGS__);     \
    }                                           					\
    while (0)
#define log_e(fmt, ...)      _log_line("E", fmt, ##__VA_ARGS__)
#define log_d(fmt, ...)      _log_line("D", fmt, ##__VA_ARGS__)
#define log_i(fmt, ...)      _log_line("I", fmt, ##__VA_ARGS__)
```

可变参数到va_list转换示例代码：

```c
#define LOG_BUF_SZ 256
void h_log(const char *fmt, ...)
{
	char buf[LOG_BUF_SZ];
    va_list ap;
    va_start(ap, fmt);
	h_sprintf(buf,fmt,ap);
	va_end(ap);
	hlib.log_out(buf);
}
```



### 宏和预处理

c语言中可以通过#define来定义宏。在预处理阶段，宏会被替换成对应展开内容。

* 无参数宏

格式型如`#define NAME CONTENT`

* 带参数宏

格式型如`#define NAME(P1,P2...) CONTENT`

注意宏名称与左小括号间不能有任何空白。否则会当成无参数宏来处理。

* 可选参数宏

c99中允许定义有省略号的宏。标识符 ` __VA_ARGS__  `代表输入参数。

* 延续运算符 \

当一行写不下时，可以使用\进行换行书写。

* 字符串化符 #

 当形参名称出现在替换文本中，并且具有前缀 # 字符时，预处理器会把与该形参对应的实参放到一对双引号中，形成一个字符串字面量。 

* 连接符 ##

 运算符##会把左、右操作数结合在一起 ， 出现在 ## 运算符前后的空白符连同 ## 运算符本身一起被删除。 

* 宏中使用宏

注意宏只能展开一次。

* 预处理

 C 预处理器是一个文本替换工具而已，它们会指示编译器在实际编译之前完成所需的预处理。 

| 指令     | 描述                                                        |
| :------- | :---------------------------------------------------------- |
| #define  | 定义宏                                                      |
| #include | 包含一个源代码文件                                          |
| #undef   | 取消已定义的宏                                              |
| #ifdef   | 如果宏已经定义，则返回真                                    |
| #ifndef  | 如果宏没有定义，则返回真                                    |
| #if      | 如果给定条件为真，则编译下面代码                            |
| #else    | #if 的替代方案                                              |
| #elif    | 如果前面的 #if 给定条件不为真，当前条件为真，则编译下面代码 |
| #endif   | 结束一个 #if……#else 条件编译块                              |
| #error   | 当遇到标准错误时，输出错误消息                              |
| #pragma  | 使用标准化方法，向编译器发布特殊的命令到编译器中            |

ANSI C中有一些预定义宏，可以直接使用

| 宏       | 描述                                                |
| :------- | :-------------------------------------------------- |
| __DATE__ | 当前日期，一个以 "MMM DD YYYY" 格式表示的字符常量。 |
| __TIME__ | 当前时间，一个以 "HH:MM:SS" 格式表示的字符常量。    |
| __FILE__ | 这会包含当前文件名，一个字符串常量。                |
| __LINE__ | 这会包含当前行号，一个十进制常量。                  |
| __STDC__ | 当编译器以 ANSI 标准编译时，则定义为 1。            |

#### 奇淫技巧

* QP Framework中有段代码

```python
#define Q_TRAN(target_)  \
    ((Q_HSM_UPCAST(me))->temp = Q_STATE_CAST(target_), (QState)Q_RET_TRAN)
```

使用时

`status_ = Q_TRAN(&Table_paused);`

这个宏利用了c语言中赋值操作 

`a=(b,c);`

实际上会先执行语句b，然后将 c 赋值给 a，

所以上面的宏等价于 先执行了 一个赋值语句，然后再返回了一个返回值。

* Linux container_of

```c
#define offsetof(TYPE, MEMBER) ((size_t) &((TYPE *)0)->MEMBER)

#define container_of(ptr, type, member) ({	    \
	const typeof( ((type *)0)->member ) *__mptr = (ptr);    \
	(type *)( (char *)__mptr - offsetof(type,member) );})
```

功能：根据结构体元素指针找出结构体指针

### 内存知识

* 内存区间

1. **代码区** .text

存放代码指令，常量。

2. **数据段** .data

存放已经初始化，且不为0的全局变量和静态变量

3. **bss段** .bss

存放所有未初始化的全局变量和静态变量，bss段不会记入可执行文件大小，可执行文件大小由 text和data段构成

4. **堆区**

堆上的内存可以动态申请和释放。

内存申请是通过系统调用brk完成，malloc存在缓存，不用每次都调用brk。

内存释放后，要避免野指针的使用。

5. **栈区**

栈是一种后入先出的结构，常用于函数调用后的数据恢复，存放函数调用，参数，内部变量等。

每个程序/线程都有自己的栈，栈超过了最大尺寸，则会造成栈溢出。

内部变量存放在栈上，离开作用域后，栈空间会自动释放。

* 相关函数

在`stdlib.h`头文件中，包含了几个内存处理函数，包括

```c
void *calloc(int num, int size);			/*分配num*size字节大小并初始化为0*/
void *malloc(int num);						/*分配num字节，不初始化*/
void *realloc(void *address, int newsize); /*重新分配大小*/
void free(void *address);
```

典型应用代码如下

```c
int *p = (int*)malloc(100);
free(p);
```

在嵌入式开发中，遵循MISRA标准，为了程序可靠性等，尽量避免使用动态内存分配。

### 头文件

 头文件包含了 C 函数声明和宏定义，可以被多个源文件引用。 

使用预处理指令`#include`来进行头文件引用。

`#include <file>` 

当后面是<> 符号时，会在系统目录进行头文件搜索。

`#include "file"` 

当后面是”“ 符号时，会在用户目录进行头文件搜索。

当遇到 **#include** 指令时，预处理过程会将指定的文件原样的插入到当前位置。

* 单次引用

为了避免头文件引入多次而造成的一些错误，可以采用#ifndef 的方式或者#pragma的方式实现头文件的单次包含。

```c
#ifndef XXX
#define XXX
...
#endif
```

当第一次include该文件时，因为没有定义XXX，所以进入内部，定义了XXX，当第二次include时，XXX已经被定义，所以就不会再进入了。

为了保证每个头文件的XXX都独一无二，所以一般可以采用文件路径名称的方式来定义，例如

`__PATH1_PATH2_FILENAME` 之类的名称

### 数据结构

#### 抽象数据类型ADT

利用纯C语言实现OOP需要很高的技巧以及对C语言很深入的理解，网上能找到不少深入的教程，这里不过多说明。

比较好的参考资料：https://www.state-machine.com/doc/AN_OOP_in_C.pdf

#### 链表

```c
#ifndef INC_H_LIST_H_
#define INC_H_LIST_H_
#include "h_def.h"

struct HNode
{
	struct HNode* next;
};
struct HList
{
	struct HNode* head;
	struct HNode* tail;
	bool (*add)(struct HList *list, struct HNode* node);
	bool (*remove)(struct HList *list, struct HNode* node);
    void (*travel)(struct HList *list, void (*func)(struct HNode* node));
};
typedef struct HList HList;
#define HNODE(x) ((struct HNode *)x)
void list_init(HList *list);
#endif /* INC_H_LIST_H_ */
```

h_list.h

```c
/*
 * h_list.c
 *
 *  Created on: 2020年4月28日
 *      Author: winxos
 */
#include "../inc/h_list.h"
static bool list_add(struct HList *list, struct HNode* node)
{
    if(list->head==NULL)
    {
        list->head=node;
        list->tail=node;
    } else{
        list->tail->next=node;
        list->tail=node;
    }
    return 0;
}

static bool list_remove(struct HList *list, struct HNode* node)
{
    struct HNode* p=list->head;
    while(p)
    {
        if(p->next==node)
        {
        	p->next = p->next->next;
        	break;
        }
        p=p->next;
    }
    return 0;
}
static void list_travel(struct HList *list, void (*func)(struct HNode* node))
{
    struct HNode* p=list->head;
    while(p)
    {
        func(p);
        p=p->next;
    }
}
void list_init(struct HList *list)
{
    list->tail=NULL;
    list->head=NULL;
    list->add = list_add;
    list->remove =  list_remove;
    list->travel = list_travel;
}
```

h_list.c

使用方法如下：

1.建立任意名称结构体，第一个元素为`struct HNode super;`

如`struct HDevice`这样

```c
#define DEVICE_NAME_MAX 7
struct HDevice{
	struct HNode super;
	char name[DEVICE_NAME_MAX+1];
	void(*init)(struct HDevice*);
	void(*loop)(struct HDevice*);
	void(*tick)(struct HDevice*);
	void* device;
};
typedef struct HDevice HDevice;
```

然后，使用时

```c
static void _init(struct HNode* node)
{
	struct HDevice *device=(struct HDevice *)node;
	device->init(device);
}
void test()
{
   	HList devices;
    list_init(&devices);
    struct HDevice device;
    devices.add(&devices,HNODE(device));
	devices.travel(&devices,_init); 
}
```

上面这种将HNode结构体放置于另一个结构体头部，方式实际上可以理解为是一种继承的实现，通过指针的强制转换，就可以实现基类的功能，利用这个特性就实现了C语言的数据结构OOP。

#### 队列
#### 二叉树

#### 红黑树

#### B树

#### B+树

### 标准库使用

