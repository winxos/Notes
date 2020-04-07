# 嵌入式系统的事件驱动式状态机C语言实现

> wvv 20200407

[toc]

### 前言

本文设计了一种最简单的事件驱动式状态机，采用C语言实现，非常简短，具备极强的移植性与实用性。

本文状态机框架，特别适合用于中小型项目的项目架构和业务逻辑流处理。

### 核心概念

* 状态机

宏观上来讲，每个项目都可以分解为一个个的子任务，每个子任务都可以看成是一个状态，系统任何时候都处于某个状态，通过某些事件触发实现状态改变或者转移。

* 事件

事件可以是外部任何输入，或者内部软件自身产生的信号。

* 事件队列

任何的事件产生后都放入事件队列中，队列可以自己实现，也可以采用各个RTOS中的现成队列来实现。

事件队列的采用实现了事件和系统行为实现的解耦。

* 事件派发

当事件产生后，框架会把事件派发给当前状态来进行执行。当没有事件产生时，系统状态不会发生任何转移。

* C语言中的继承实现

父类声明成一个结构体，字类也声明成一个结构体，然后把一个父类声明放到第一个元素，再将自己的私有变量放到后面，使用时分别用子类和父类类型指针就可以操作不同的部分。

原理分析：因为内存中结构体成员是依次存放，所以当父类指针指向字类时就会指向其第一个元素，也就是其父类成员，这样就实现了父类结构体的重用。

在本系统中的应用：每个项目的状态机都会不同，但是其都会有共同的部分，所以就将共同的部分放到了父类中，这样状态机框架就能进行推动整个系统，不同的地方放到子类中，这样框架在使用的时候自由度就很大。

### 框架使用示例

下面代码以RT-Thread 为例，可以很容易修改为其他RTOS或者裸机运行。

#### 使用代码

```c
#include <rtthread.h>

#define DBG_TAG "main"
#define DBG_LVL DBG_LOG
#include <rtdbg.h>
#include "hl_state.h"
enum {
    SIG_READY,
    SIG_TICK
};
struct HLLED{
    HLState super;
    int a;
};
struct HLLED led;
struct rt_mailbox mb;
char mb_pool[128];
void mb_push(int n)
{
    rt_mb_send(&mb, n);
}
SRET work(HLState *me)
{
    LOG_D("work e:%d",me->evt);
    switch(me->evt)
    {
    case SIG_READY:
        LOG_D("ENTRY %d",((struct HLLED *)me)->a);
        break;
    case SIG_TICK:
        LOG_D("TICK");
        break;
    }
    return SRET_OK;
}
SRET init(HLState *me)
{
    LOG_D("init e:%d",me->evt);
    static int n=0;
    switch(me->evt)
    {
    case SIG_TICK:
        n++;
        if (n>5) {
            LOG_D("NEXT");
            ((struct HLLED *)me)->a =  5;
            NEXT(work);
            mb_push(SIG_READY);
        }
        break;
    }
    return SRET_OK;
}
void ticks(void *p)
{
    mb_push(SIG_TICK);
}
int main(void)
{
    rt_mb_init(&mb,
                "mbt",                      /* 名称是 mbt */
                &mb_pool[0],                /* 邮箱用到的内存池是 mb_pool */
                sizeof(mb_pool) / 4,        /* 邮箱中的邮件数目，因为一封邮件占 4 字节 */
                RT_IPC_FLAG_FIFO);          /* 采用 FIFO 方式进行线程等待 */
    struct rt_timer t;
    rt_timer_init(&t,"tick", ticks,
                             RT_NULL, 1000,
                             RT_TIMER_FLAG_PERIODIC);
    rt_timer_start(&t);
    LOG_D("Hello RT-Thread!");
    HLState *me=(HLState *)&led;
    NEXT(init);
    mb_push(SIG_READY);
    while (1)
    {
        if (rt_mb_recv(&mb, &me->evt, RT_WAITING_FOREVER) == RT_EOK)
        {
            hl_state_execute(me);
        }
        rt_thread_mdelay(10);
    }

    return RT_EOK;
}
```

#### 代码解析

我们从main开始分析，

* `rt_mb_init`

函数初始化了一个邮箱队列，用于接收事件，RT-Thread下的实现。

* `rt_timer_init`

函数用于产生一个定时器，每秒调用一次`ticks`函数，RT-Thread下的实现。

* `rt_timer_start`

启动定时器，RT-Thread下的实现。

* `LOG_D`

输出个调试消息，RT-Thread下的实现。

* `HLState *me=(HLState *)&led;`

创建了一个名为me的指针，指向一个结构体led。

其中结构体led声明如下：

```c
struct HLLED{
    HLState super;
    int a;
};
struct HLLED led;
```

其名称HLLED是自行任意定义，但是其第一个元素必须为`HLState`类型，根据之前的C语言中的继承实现的分析，其目的是为了继承`HLState`这样框架就可以对他进行自动推动，作为演示，HLLED中另外封装了一个自定义变量a。

`HLState *me` 声明了一个me指针，其指向了led的地址，因为类型由`HLLED`转换为了`HLState`，所以me实际上指向了led中的super成员。

* `NEXT(init);`

函数将系统状态转移到了init，实际上就是设置了系统的初始状态为init。我们往前找到init的声明

```c
SRET init(HLState *me)
{
    ...;
}
      
```

我们发现`init`是一个函数，其参数是一个`HLState`类型指针

* `mb_push(SIG_READY);`

转移到了init后，并没有事件产生，所以系统无法推动，所以我们产生一个SIG_READY事件，放到邮箱中。

事件是一开始的时候定义了

```c
enum {
    SIG_READY,
    SIG_TICK
};
```

产生了SIG_READY事件后，系统就会自动推动，上一句中的init状态就会接收到该事件。

还有一个事件SIG_TICK，在之前定时器代码中会每秒调用ticks一次，ticks每次会产生一个SIG_TICK事件。

* `rt_mb_recv`

如果邮箱接收到事件，RT-Thread下的实现。

* `hl_state_execute`

派发事件到状态

* `rt_thread_mdelay`

毫秒级延时函数，RT-Thread下的实现。

* `init`中主要代码详解

`LOG_D("init e:%d",me->evt);`

打印消息，me->evt为事件值

```c
case SIG_TICK:
    n++;
    if (n>5) {
        LOG_D("NEXT");
        ((struct HLLED *)me)->a =  5;
        NEXT(work);
        mb_push(SIG_READY);
    }
    break;
```

上面的代码当SIG_TICK事件发生时就n加1，当n大于5时，将me指针转换为HLLED类型，然后修改了其中a成员值为5，然后执行NEXT将状态转移到work，然后触发SIG_READY事件。

* `work`中主要代码详解

```c
switch(me->evt)
{
    case SIG_READY:
        LOG_D("ENTRY %d",((struct HLLED *)me)->a);
        break;
    case SIG_TICK:
        LOG_D("TICK");
        break;
}
```

根据me中不同的事件，进行不同的行为。

当信号为SIG_READY时，打印出ENTRY 以及 a的值。

当信号为SIG_TICK时，打印出TICK。

#### 运行结果

```sh
 \ | /
- RT -     Thread Operating System
 / | \     3.1.3 build Apr  7 2020
 2006 - 2019 Copyright by rt-thread team
[D/main] Hello RT-Thread!
[D/main] init e:0
[D/main] init e:1
[D/main] init e:1
[D/main] init e:1
[D/main] init e:1
[D/main] init e:1
[D/main] init e:1
[D/main] NEXT
[D/main] work e:0
[D/main] ENTRY 5
[D/main] work e:1
[D/main] TICK
[D/main] work e:1
[D/main] TICK
...
```

#### 结果分析

我们可以看到

先打印了一次`init e:0`，说明状态init运行，信号量是0

然后打印了六次`init e:1`，说明init运行了6次，信号量是1

然后打印了`NEXT`，说明进入了if语句，状态将要发生转移

然后打印了一次`work e:0`，说明状态work已经转移成功，收到信号量0

然后打印了`ENTRY 5`，说明状态work收到了状态init中设置的a的值5，说明状态间实现了数据共享

然后连续打印`work e:1`和`TICK`，说明系统持续处于work状态和收到TICK事件

结合先前的代码解析，发现代码完全按照我们设想的方式在运行，逻辑非常的清晰，非常易用

### 框架的实现

从框架使用上来看，我们会发现他的核心之处是实现了函数指针的自动转移，同时参数又包含了全部事件以及状态机自身的数据，实现了共享，同时框架又是事件驱动式，而非传统的轮询式，极大提高了处理器利用率和实时性，框架代码实际上非常简单，主要文件如下：

**hl_state.h**

```c
#ifndef INC_HL_STATE_H_
#define INC_HL_STATE_H_
enum {
	SRET_OK
};
typedef unsigned char SRET;
typedef unsigned long Sig;
struct HLState;
typedef SRET (*pFunState)(struct HLState * me);
struct HLState{
	pFunState state;
	Sig evt;
};
typedef struct HLState HLState;
#define NEXT(_target) (me->state = _target)
void hl_state_execute(HLState * me);
#endif /* INC_HL_STATE_H_ */
```

**hl_state.c**

```c
#include "hl_state.h"
void hl_state_execute(HLState * me)
{
	if(me->state)
	{
		me->state(me);
	}
}
```

#### 注意事项

在转移时，按理应该需要传递状态机自身指针和目的状态函数，但是为了使用时可以减少一个参数，简化使用，就约定了所有状态机指针需要命名为me，这样在进行NEXT转移时才能正常运行。

### 结束语

函数转移式状态机在嵌入式系统中可以极大的优化代码结构，简化系统逻辑，传统的状态机实现相当于是一个巨大的switch语句，当项目复杂到一定程度后很难维护，而且每个状态都需要额外的定义枚举变量来区分，这样也增加了代码的重复性，同时传统的状态机是轮询式的，实时性得不到保证。

本框架优雅的实现得益于对QPFramework的源码阅读，QPFramework是个很完整的层次式状态机框架，建议搞嵌入式的同行深入了解，但大部分的中小型项目用不到这么复杂的框架，所以我对其进行了简化，只借鉴了其用法，然后实现了最简单的扁平式的事件触发状态机框架，已经可以满足自己遇到的绝大部分需求了，希望对各位有帮助。