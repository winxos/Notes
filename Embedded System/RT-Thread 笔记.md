# RT-Thread 笔记
> wvv 20200227

[toc]
### RT-Thread Studio 使用笔记

### BSP开发
#### 硬件资源宏配置
#### 时钟配置
```c
void SystemClock_Config(void)
{
  RCC_OscInitTypeDef RCC_OscInitStruct = {0};
  RCC_ClkInitTypeDef RCC_ClkInitStruct = {0};

  /** Initializes the CPU, AHB and APB busses clocks 
  */
  RCC_OscInitStruct.OscillatorType = RCC_OSCILLATORTYPE_HSE;
  RCC_OscInitStruct.HSEState = RCC_HSE_ON;
  RCC_OscInitStruct.HSEPredivValue = RCC_HSE_PREDIV_DIV1;
  RCC_OscInitStruct.HSIState = RCC_HSI_ON;
  RCC_OscInitStruct.PLL.PLLState = RCC_PLL_ON;
  RCC_OscInitStruct.PLL.PLLSource = RCC_PLLSOURCE_HSE;
  RCC_OscInitStruct.PLL.PLLMUL = RCC_PLL_MUL9;
  if (HAL_RCC_OscConfig(&RCC_OscInitStruct) != HAL_OK)
  {
    Error_Handler();
  }
  /** Initializes the CPU, AHB and APB busses clocks 
  */
  RCC_ClkInitStruct.ClockType = RCC_CLOCKTYPE_HCLK|RCC_CLOCKTYPE_SYSCLK
                              |RCC_CLOCKTYPE_PCLK1|RCC_CLOCKTYPE_PCLK2;
  RCC_ClkInitStruct.SYSCLKSource = RCC_SYSCLKSOURCE_PLLCLK;
  RCC_ClkInitStruct.AHBCLKDivider = RCC_SYSCLK_DIV1;
  RCC_ClkInitStruct.APB1CLKDivider = RCC_HCLK_DIV2;
  RCC_ClkInitStruct.APB2CLKDivider = RCC_HCLK_DIV1;

  if (HAL_RCC_ClockConfig(&RCC_ClkInitStruct, FLASH_LATENCY_2) != HAL_OK)
  {
    Error_Handler();
  }
}

```
### 内核相关

#### 线程
[官方文档](https://www.rt-thread.org/document/site/programming-manual/thread/thread/)
在 RT-Thread 中，线程包含五种状态：
初始状态、挂起状态、就绪状态、运行状态、关闭状态
RT-Thread 最大支持 256 个线程优先级 (0\~255)，数值越小的优先级越高，0 为最高优先级。在一些资源比较紧张的系统中，可以根据实际情况选择只支持 8 个或 32 个优先级的系统配置；对于 ARM Cortex-M 系列，普遍采用 32 个优先级。
空闲线程是系统创建的最低优先级的线程，线程状态永远为就绪态。当系统中无其他就绪线程存在时，调度器将调度到空闲线程，它通常是一个死循环，且永远不能被挂起。
在系统启动时，系统会创建 main 线程，它的入口函数为 main_thread_entry()，用户的应用入口函数 main() 就是从这里真正开始的，系统调度器启动后，main 线程就开始运行。
```c
rt_thread_t rt_thread_create(const char* name,
                            void (*entry)(void* parameter),
                            void* parameter,
                            rt_uint32_t stack_size,
                            rt_uint8_t priority,
                            rt_uint32_t tick);
rt_err_t rt_thread_delete(rt_thread_t thread);
rt_err_t rt_thread_init(struct rt_thread* thread,
                        const char* name,
                        void (*entry)(void* parameter), void* parameter,
                        void* stack_start, rt_uint32_t stack_size,
                        rt_uint8_t priority, rt_uint32_t tick); /* 静态方式 */
rt_err_t rt_thread_startup(rt_thread_t thread);
rt_thread_t rt_thread_self(void);
rt_err_t rt_thread_yield(void);
rt_err_t rt_thread_sleep(rt_tick_t tick);
rt_err_t rt_thread_delay(rt_tick_t tick);
rt_err_t rt_thread_mdelay(rt_int32_t ms);
rt_err_t rt_thread_suspend (rt_thread_t thread);
rt_err_t rt_thread_resume (rt_thread_t thread);
rt_err_t rt_thread_idle_sethook(void (*hook)(void));
rt_err_t rt_thread_idle_delhook(void (*hook)(void));
void rt_scheduler_sethook(void (*hook)(struct rt_thread* from, struct rt_thread* to));
```
#### 定时器
[官方文档](https://www.rt-thread.org/document/site/programming-manual/timer/timer/#rt-thread)
RT-Thread 的定时器提供两类定时器机制：第一类是单次触发定时器，这类定时器在启动后只会触发一次定时器事件，然后定时器自动停止。第二类是周期触发定时器，这类定时器会周期性的触发定时器事件，直到用户手动的停止，否则将永远持续执行下去。
RT-Thread 的定时器可以分为 HARD_TIMER 模式与 SOFT_TIMER 模式。
```c
struct rt_timer
{
    struct rt_object parent;
    rt_list_t row[RT_TIMER_SKIP_LIST_LEVEL];  /* 定时器链表节点 */

    void (*timeout_func)(void *parameter);    /* 定时器超时调用的函数 */
    void      *parameter;                         /* 超时函数的参数 */
    rt_tick_t init_tick;                         /* 定时器初始超时节拍数 */
    rt_tick_t timeout_tick;                     /* 定时器实际超时时的节拍数 */
};
typedef struct rt_timer *rt_timer_t;
rt_timer_t rt_timer_create(const char* name, void (*timeout)(void* parameter),
                                            void* parameter, rt_tick_t time, rt_uint8_t flag);
rt_err_t rt_timer_delete(rt_timer_t timer);
void rt_timer_init(rt_timer_t timer, const char* name, void (*timeout)(void* parameter), 
                                void* parameter, rt_tick_t time, rt_uint8_t flag);
rt_err_t rt_timer_start(rt_timer_t timer);
rt_err_t rt_timer_stop(rt_timer_t timer);
```
#### 信号量
```c
struct rt_semaphore
{
   struct rt_ipc_object parent;  /* 继承自 ipc_object 类 */
   rt_uint16_t value;              /* 信号量的值 */
};
/* rt_sem_t 是指向 semaphore 结构体的指针类型 */
typedef struct rt_semaphore* rt_sem_t;
rt_sem_t rt_sem_create(const char *name, rt_uint32_t value, rt_uint8_t flag);
rt_err_t rt_sem_init(rt_sem_t sem, const char *name, rt_uint32_t value, rt_uint8_t flag);
rt_err_t rt_sem_take (rt_sem_t sem, rt_int32_t time);
rt_err_t rt_sem_trytake(rt_sem_t sem);
rt_err_t rt_sem_release(rt_sem_t sem);
```
#### 互斥量
```c
struct rt_mutex
    {
        struct rt_ipc_object parent;                /* 继承自 ipc_object 类 */

        rt_uint16_t          value;                   /* 互斥量的值 */
        rt_uint8_t           original_priority;     /* 持有线程的原始优先级 */
        rt_uint8_t           hold;                     /* 持有线程的持有次数   */
        struct rt_thread    *owner;                 /* 当前拥有互斥量的线程 */
    };
 /* rt_mutext_t 为指向互斥量结构体的指针类型  */
typedef struct rt_mutex* rt_mutex_t;
rt_mutex_t rt_mutex_create (const char* name, rt_uint8_t flag);
rt_err_t rt_mutex_init (rt_mutex_t mutex, const char* name, rt_uint8_t flag);
rt_err_t rt_mutex_take (rt_mutex_t mutex, rt_int32_t time);
rt_err_t rt_mutex_release(rt_mutex_t mutex);
```
#### 事件集
事件集主要用于线程间的同步，与信号量不同，它的特点是可以实现一对多，多对多的同步。即一个线程与多个事件的关系可设置为：其中任意一个事件唤醒线程，或几个事件都到达后才唤醒线程进行后续的处理；
```c
struct rt_event
{
    struct rt_ipc_object parent;    /* 继承自 ipc_object 类 */

    /* 事件集合，每一 bit 表示 1 个事件，bit 位的值可以标记某事件是否发生 */
    rt_uint32_t set;
};
/* rt_event_t 是指向事件结构体的指针类型  */
typedef struct rt_event* rt_event_t;
rt_event_t rt_event_create(const char* name, rt_uint8_t flag);
rt_err_t rt_event_delete(rt_event_t event);
rt_err_t rt_event_init(rt_event_t event, const char* name, rt_uint8_t flag);
rt_err_t rt_event_send(rt_event_t event, rt_uint32_t set);
rt_err_t rt_event_recv(rt_event_t event, rt_uint32_t set, rt_uint8_t option, 
                                    rt_int32_t timeout, rt_uint32_t* recved);
```
#### 邮箱

#### 消息队列

#### 信号

### 基础知识

#### 串口UART的使用
在工程drivers/board.h文件中，开头几行，通过宏定义来开启UART
例如 
```c
#define BSP_USING_UART1
#define BSP_USING_UART2
#define BSP_UART1_RX_USING_DMA
```
然后在接下来的代码中记得修改对应io口
```c
#if defined(BSP_USING_UART2)
#define UART2_TX_PORT       GPIOA
#define UART2_RX_PORT       GPIOA
#define UART2_TX_PIN        GPIO_PIN_2
#define UART2_RX_PIN        GPIO_PIN_3
#endif
```
配置好后就可以使用了。
使用过程如下
```c
    rt_device_t serial = rt_device_find("uart1");
    struct serial_configure config = RT_SERIAL_CONFIG_DEFAULT;  /* 初始化配置参数 */
    config.baud_rate = BAUD_RATE_115200;        //修改波特率为 9600
    config.data_bits = DATA_BITS_8;           //数据位 8
    config.stop_bits = STOP_BITS_2;           //停止位 1
    config.parity    = PARITY_NONE;           //无奇偶校验位
    rt_device_control(serial, RT_DEVICE_CTRL_CONFIG, &config);
    rt_device_open(serial, RT_DEVICE_FLAG_DMA_RX);  //DMA 方式
```
默认串口配置为
```c
#define RT_SERIAL_CONFIG_DEFAULT           \
{                                          \
    BAUD_RATE_115200, /* 115200 bits/s */  \
    DATA_BITS_8,      /* 8 databits */     \
    STOP_BITS_1,      /* 1 stopbit */      \
    PARITY_NONE,      /* No parity  */     \
    BIT_ORDER_LSB,    /* LSB first sent */ \
    NRZ_NORMAL,       /* Normal mode */    \
    RT_SERIAL_RB_BUFSZ, /* Buffer size */  \
    0                                      \
}
```
如果采用默认配置的话，则不需要调用rt_device_control进行串口配置。
串口接收通过
```c
rt_device_set_rx_indicate(serial, uart_input);
```
来设置回调函数，回调函数如下
```c
static rt_err_t uart_input(rt_device_t dev, rt_size_t size)
{
    return RT_EOK;
}
```
### 组件
#### FinSH
#### ulog
#### utest
### 应用笔记
#### 与HLDevices框架的整合
HLDevices为自研的组件层，采用HLDevices框架可以重复利用很多代码，而且使用也非常简单。
HLDevices需要系统调用init，loop，tick三个函数来运行框架，
在RTT中，我们可以在初始化的时候调用init，
利用定时器产生一个毫秒级信号产生tick，
然后在主线程中调用loop
示例代码如下
```c
#include <rtthread.h>
#include <board.h>
#include <rtdevice.h>
#define DBG_TAG "main"
#define DBG_LVL DBG_LOG
#include <rtdbg.h>
#include <hlib.h>
#define LED0_PIN    GET_PIN(C, 13)
struct HLDevice dtasks={"task",(void*)10,&hl_tasks_ops};
#define HLIB_HEAP_SIZE 1000
static u8 _head_buf[HLIB_HEAP_SIZE]={0};
HLib _lib={
        .pool=&_head_buf,
        .pool_size=HLIB_HEAP_SIZE
};
void t1(void *p)
{
    static u8 value=0;
    LOG_I("t");
    rt_pin_write(LED0_PIN, value);
    value=1-value;
}
void t2(void *p)
{
    LOG_I("x");
}
void ticks(void *p)
{
    hl_devices_tick();
}
int main(void)
{
    rt_pin_mode(LED0_PIN, PIN_MODE_OUTPUT);
    hl_lib_init(&_lib);
    hl_device_register(&dtasks);
    hl_devices_init();
    rt_timer_t t=rt_timer_create("tick", ticks,
                             RT_NULL, 10,
                             RT_TIMER_FLAG_PERIODIC);
    rt_timer_start(t);
    hl_tasks_once_add(t2, NULL, 300);
    hl_tasks_add(t1,NULL,1000,0);
    while (1)
    {
        hl_devices_loop();
        rt_thread_mdelay(10);
    }
    return RT_EOK;
}
```
上述代码初始化了HLib，产生了10ms的tick触发，
简单的调用了HLDevices中的hl_tasks_add，hl_tasks_once_add
执行了循环任务和单次执行任务。
#### uart dma 空闲中断收发

#### 事件集示例
本代码模拟了多个事件条件同时满足才进行下一步操作。
```c
#define LED0_PIN    GET_PIN(C, 13)
#define PIN1    GET_PIN(B, 1)
#define PIN2    GET_PIN(B, 2)

struct rt_timer tt1,tt2;
struct rt_event event;
#define EVENT_FLAG1 (1 << 1)
#define EVENT_FLAG2 (1 << 2)

static rt_uint8_t ct1,ct2;
void t1(void *p)
{
    static rt_uint8_t last_p1;
    if (last_p1 == rt_pin_read(PIN1))
    {
        ct1++;
        if(ct1>10)
        {
            rt_timer_stop(p);
            rt_event_send(&event, EVENT_FLAG1);
        }
    }
    last_p1 = rt_pin_read(PIN1);
    LOG_I("t1:%p %d %d",p,last_p1,ct1);
}
void t2(void *p)
{
    static rt_uint8_t last_p2;
    if (last_p2 == rt_pin_read(PIN2))
    {
        ct2++;
        if(ct2>20)
        {
            rt_timer_stop(p);
            rt_event_send(&event, EVENT_FLAG2);
        }
    }
    last_p2 = rt_pin_read(PIN2);
    LOG_I("t2:%p %d %d",p,last_p2,ct2);

}
void wait(void *p);
void start_t()
{
    rt_thread_t tt= rt_thread_create("tt",
                       wait,RT_NULL,512,20,10);
    rt_thread_startup(tt);
    ct1=0,ct2=0;
    rt_timer_start(&tt1);
    rt_timer_start(&tt2);
}
void wait(void *p)
{
    rt_uint32_t e;
    if (rt_event_recv(&event, (EVENT_FLAG1 | EVENT_FLAG2),
                      RT_EVENT_FLAG_AND | RT_EVENT_FLAG_CLEAR,
                      RT_WAITING_FOREVER, &e) == RT_EOK)
    {
        LOG_I("WAIT OK");
        rt_thread_delay(2000);
    }
}
int main(void)
{
    rt_pin_mode(LED0_PIN, PIN_MODE_OUTPUT);
    rt_pin_mode(PIN1, PIN_MODE_INPUT_PULLUP);
    rt_pin_mode(PIN2, PIN_MODE_INPUT_PULLUP);
    rt_timer_init(&tt1,"t1", t1,
                &tt1, 500, RT_TIMER_FLAG_PERIODIC);
    rt_timer_init(&tt2,"t2", t2,
                &tt2, 500, RT_TIMER_FLAG_PERIODIC);
    rt_event_init(&event, "event", RT_IPC_FLAG_FIFO);
    LOG_I("t1:%p t2:%p",&tt1,&tt2);
    start_t();
    while (1)
    {
        rt_thread_mdelay(10);
    }
    return RT_EOK;
}
```
解释：
调用start_t函数时，创建后台线程wait，等待信号量EVENT_FLAG1和EVENT_FLAG2同时触发，
同时启动周期性定时器任务tt1，tt2，定时器tt1，tt2的传入参数就是自身对应定时器的句柄，
当tt1，tt2各自满足任务结束条件时，设置对应信号量，
当tt1，tt2都完成后，先前处于等待的wait线程满足触发条件，执行下一步任务，线程退出。

#### StateMachine示例

##### rtt_main.c

```c
#include <rtthread.h>

#define DBG_TAG "main"
#define DBG_LVL DBG_LOG
#include <rtdbg.h>
#include <hlib.h>
struct rt_mailbox mb;
char mb_pool[64];
void mb_push(int n)
{
    rt_mb_send(&mb, n);
}
__weak_symbol void ticks()
{

}
void setup();
extern HLState *me;
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
    setup();
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

##### app.c

 ```c
#include <rtthread.h>

#define DBG_TAG "app"
#define DBG_LVL DBG_LOG
#include <rtdbg.h>
#include <hlib.h>
void mb_push(int n);
enum {
    SIG_READY,
    SIG_TICK
};
struct HLLED{
    HLState super;
    int a;
};
struct HLLED led;
HLState *me=(HLState *)&led;
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
void setup()
{
    NEXT(init);
    mb_push(SIG_READY);
}
 ```

##### hl_state.h

```c
#ifndef INC_HL_STATE_H_
#define INC_HL_STATE_H_
#include "typedef.h"
enum {
	SRET_OK
};
typedef u8 SRET;
typedef u32 Sig;
struct HLState;
typedef SRET (*pFunState)(struct HLState * me);
struct HLState{
	pFunState state;
	Sig evt;
};
typedef struct HLState HLState;
#define NEXT(_target) (me->state = _target)
void hl_state_execute(HLState * me);
```

##### hl_state.c

```c
#include "../inc/hl_state.h"
void hl_state_execute(HLState * me)
{
	if(me->state)
	{
		me->state(me);
	}
}
```

#### GY-49 I2C操作

```c
static rt_err_t read_regs(struct rt_i2c_bus_device *bus)
{
    struct rt_i2c_msg msgs[4];
    uint8_t cmd[2]={0x03,0x04};
    uint8_t b[2]={0};
    msgs[0].addr = 0x4A;     /* 从机地址 */
    msgs[0].flags = RT_I2C_WR ;     /* 读标志 */
    msgs[0].buf = cmd;             /* 读写数据缓冲区指针　*/
    msgs[0].len = 1;             /* 读写数据字节数 */
    msgs[1].addr = 0x4A;     /* 从机地址 */
    msgs[1].flags = RT_I2C_RD | RT_I2C_NO_READ_ACK  ;     /* 读标志 */
    msgs[1].buf = b;             /* 读写数据缓冲区指针　*/
    msgs[1].len = 1;             /* 读写数据字节数 */
    msgs[2].addr = 0x4A;     /* 从机地址 */
    msgs[2].flags = RT_I2C_WR ;     /* 读标志 */
    msgs[2].buf = &cmd[1];             /* 读写数据缓冲区指针　*/
    msgs[2].len = 1;             /* 读写数据字节数 */
    msgs[3].addr = 0x4A;     /* 从机地址 */
    msgs[3].flags = RT_I2C_RD | RT_I2C_NO_READ_ACK  ;     /* 读标志 */
    msgs[3].buf = &b[1];             /* 读写数据缓冲区指针　*/
    msgs[3].len = 1;             /* 读写数据字节数 */
    /* 调用I2C设备接口传输数据 */
    if (rt_i2c_transfer(bus, &msgs,4) == 4)
    {
        int e =  (b[0] & 0xF0) >> 4;
        int m = ((b[0] & 0x0F) << 4) | (b[1] &0x0F);
        for(uint8_t i=0;i<e;i++)
        {
            m=m*2;
        }
        int Lux = m * 0.045;
        log_out(Lux);
        return RT_EOK;
    }
    else
    {
        return -RT_ERROR;
    }
}
```

GY-49 为高精度光照传感器，采用MAX44009芯片，采用I2C通信。

设备总线地址0x4A

光照值寄存器0x03, 0x04

i2c时序和数值换算公式和其他寄存器参考芯片手册。

RTT中，最新i2c操作使用rt_i2c_transfer接口。

注意，在时序中可以看出最后一个读操作无应答信号，所以flags加上RT_I2C_NO_READ_ACK。

