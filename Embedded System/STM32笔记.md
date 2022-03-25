# 嵌入式软件笔记

> wvv 20191216

[TOC]

### STM32

#### STM32命名规则

#### STM32产品线

#### 开发环境搭建

##### STM32CubeIDE

##### STM32CubeMonitor

##### ST-LINK Utility

### 模块调试经验

#### GY-49/max44009 I2C光照模块

```c
uint8_t data[2]={0,0};
HAL_I2C_Mem_Read(&hi2c1, 0x94, 0x03, I2C_MEMADD_SIZE_8BIT, data, 1, 0xff);
HAL_Delay(10);
HAL_I2C_Mem_Read(&hi2c1, 0x94, 0x04, I2C_MEMADD_SIZE_8BIT, &data[1], 1, 0xff);
int Exponent =  (data[0] & 0xF0) >> 4;
int Mantissa = ((data[0] & 0x0F) << 4) | (data[1] &0x0F);
int Lux = Mantissa * pow(2, Exponent)  * 0.045;
log_out(Lux);
```

注意：hal库中，地址需要乘以2，最低位是读写标志位。

注意：i2c 中scl,sda需要上拉电阻！！！

注意：i2c设备有的3.3v，有的5v，上拉电阻不可共用！！！电压不可混用。

#### Nokia5110 LCD SPI驱动

Nokia 5110采用的PCD8544驱动芯片，SPI驱动时序

硬件SPI驱动速度较快，也很方便，stm32 hal库驱动代码如下：

```c
#include <stm32f1xx_hal.h>
#include <main.h>
#include "font.h"
#include "lcd5110.h"

extern SPI_HandleTypeDef hspi1;
#define LCD_R 6
#define LCD_C 84
#define LCD_BUF_SZ (LCD_R*LCD_C)
static uint8_t _buf[LCD_BUF_SZ] = { 0 };

void write_byte(uint8_t data)
{
    HAL_SPI_Transmit(&hspi1, &data, 1, 0xff);
}
void write_cmd(uint8_t data)
{
    HAL_GPIO_WritePin(lcd_ce_GPIO_Port, lcd_ce_Pin, GPIO_PIN_RESET);
    HAL_GPIO_WritePin(lcd_dc_GPIO_Port, lcd_dc_Pin, GPIO_PIN_RESET);
    write_byte(data);
    HAL_GPIO_WritePin(lcd_ce_GPIO_Port, lcd_ce_Pin, GPIO_PIN_SET);
}
void write_data(uint8_t data)
{
    HAL_GPIO_WritePin(lcd_ce_GPIO_Port, lcd_ce_Pin, GPIO_PIN_RESET);
    HAL_GPIO_WritePin(lcd_dc_GPIO_Port, lcd_dc_Pin, GPIO_PIN_SET);
    write_byte(data);
    HAL_GPIO_WritePin(lcd_ce_GPIO_Port, lcd_ce_Pin, GPIO_PIN_SET);
}
void lcd5110_reset()
{
    HAL_GPIO_WritePin(lcd_rst_GPIO_Port, lcd_rst_Pin, GPIO_PIN_RESET);
    HAL_Delay(1);
    HAL_GPIO_WritePin(lcd_rst_GPIO_Port, lcd_rst_Pin, GPIO_PIN_SET);
}
void lcd5110_refresh()
{
    write_cmd(0x40);
    write_cmd(0x80);
    HAL_GPIO_WritePin(lcd_ce_GPIO_Port, lcd_ce_Pin, GPIO_PIN_RESET);
    HAL_GPIO_WritePin(lcd_dc_GPIO_Port, lcd_dc_Pin, GPIO_PIN_SET);
    HAL_SPI_Transmit(&hspi1, _buf, LCD_BUF_SZ, 0xffff);
    HAL_GPIO_WritePin(lcd_ce_GPIO_Port, lcd_ce_Pin, GPIO_PIN_SET);
}
void lcd5110_clear()
{
    memset(_buf,0,LCD_BUF_SZ);
}
void lcd5110_pixel(uint8_t x, uint8_t y, uint8_t mode)
{
    if (x < 0 || x > 83 || y < 0 || y > 47)
    {
        return;
    }
    if(mode==LCD5110_PIXEL_MODE_PAINT)
    {
        _buf[(y >> 3) * 84 + x] |= 1 << (y & 0x07);
    }
    else
    {
        _buf[(y >> 3) * 84 + x] &=~( 1 << (y & 0x07));
    }
}
void lcd_putchar(char c, uint8_t x, uint8_t y)
{
    if (x < 0 || y < 0 || x >= 84 || y >= 6)
    {
        return;
    }
    const uint8_t *p = &font5x8[(c - ' ') * 5];
    for (int i = x; (i < x + 5) && i < 84; i++)
    {
        _buf[y * 84 + i] = *p;
        p++;
    }
}
void lcd5110_str(char* str, uint8_t x, uint8_t y)
{
    int offset = 0;
    while (*str)
    {
        lcd_putchar(*str, x + offset, y);
        offset += 6;
        str++;
    }
}
void lcd5110_init()
{
    lcd5110_reset();
    write_cmd(0x21);
    write_cmd(0x14);    /*bias*/
    write_cmd(0xba);    /*Vop*/
    write_cmd(0x20);
    write_cmd(0x0c);
    lcd5110_refresh();
}
```

该驱动采用全屏缓存定期刷新方式，经过测试，全屏刷新频率能达到数百帧。

完整代码查看：https://github.com/winxos/stm32_hal_lcd5110_hw_spi

### 相关技巧

#### 静态库编写和使用

#### 项目文件结构规划

#### 日志系统设计

```c
#define LOG_DETAIL 0
#if LOG_DETAIL != 0
#define LOG_LINE(lvl, fmt, ...)                                              \
    do                                                                       \
    {                                                                        \
        printf("[%10lu][" lvl "][%s:%d]" fmt "\r\n", time,__FUNCTION__, __LINE__, ##__VA_ARGS__); \
    } while (0)
#else

#define LOG_LINE(lvl, fmt, ...)                                   \
    do{                                                           \
        printf("[%10lu][" lvl "]" fmt "\r\n", time, ##__VA_ARGS__); \
    } while (0)
#endif

#define log_e(fmt, ...) LOG_LINE("E", fmt, ##__VA_ARGS__)
#define log_d(fmt, ...) LOG_LINE("D", fmt, ##__VA_ARGS__)
#define log_i(fmt, ...) LOG_LINE("I", fmt, ##__VA_ARGS__)
#define log_hex(header, data, len)                         \
        do{                                                \
        printf("[%10lu][HEX][]" header "]\r\n", time);    \
        for(uint8_t i = 0;i<len;i++)printf("%04X ",data[i]);\
        printf("\r\n");\
        }while(0)
```

这个简易的日志系统就可以通过log_e等方式进行可变参数的调用。

#### printf 重定向细节

```c
static uint8_t _buf[80];
int _write(int file, char *ptr, int len)
{
    while (huart1.gState != HAL_UART_STATE_READY)
        ;
    memcpy(_buf, ptr, len);
    HAL_UART_Transmit_DMA(&huart1, _buf, len);
    return len;
}
```

重载_write时，注意如果使用中断或者dma模式，需要通过查询状态防止重入，还要注意将数据拷贝一份再发送，因为如果不拷贝，发送未完成时又执行了发送，就会覆盖上一个数据。

上面的例程代码是，重载printf到串口1

#### 软串口实现

软串口的核心是实现高精度的微妙延时函数，才能实现误码率小。

一般采用定时器来实现。

```c
void delay_us(uint16_t us)
{
    TIM4->CNT = us;
    TIM_Cmd(TIM4, ENABLE);
    while (us > 4)
    {
        us = TIM4->CNT;
    }
    TIM_Cmd(TIM4, DISABLE);
}

void delay_ms(uint32_t t)
{
    for (uint32_t i = 0; i < t; i++)
    {
        delay_us(1000);
    }
}
/**
 *gpio simulate uart with baud
 */
void soft_tx_byte(GPIO_TypeDef *GPIOx, uint16_t GPIO_Pin, uint32_t baud, uint8_t value)
{
    uint16_t delay = 1000000 / baud;
    GPIO_SetBits(GPIOx, GPIO_Pin);     //发送起始位
    GPIO_ResetBits(GPIOx, GPIO_Pin);
    delay_us(delay);
    for (uint8_t i = 0; i < 8; i++)
    {
        if ((value & (1 << i)) != 0)
        {
            GPIO_SetBits(GPIOx, GPIO_Pin);
        }
        else
        {
            GPIO_ResetBits(GPIOx, GPIO_Pin);
        }
        delay_us(delay);
    }
    GPIO_SetBits(GPIOx, GPIO_Pin);
    delay_us(delay);
}
int __io_putchar(int ch)
{
    soft_tx_byte(GPIOC, GPIO_Pin_13, 9600, ch); //log
    return 0;
}
```

上述代码实现软串口重定向到printf。

#### 简易任务队列的实现

```c
typedef void (*pTask)(void*);
typedef struct
{
    pTask task;
    void *p;
    uint32_t t;
    uint32_t cyc;
} TaskInfo;

#define TASKS_MAX_COUNT (3)
static TaskInfo tasks_buf[TASKS_MAX_COUNT] = {0};
/**
 *
 */
uint8_t async_task_add(pTask task, void *p, uint32_t t)
{
    for (uint8_t i = 0; i < TASKS_MAX_COUNT; i++)
    {
        if(tasks_buf[i].task == NULL)
        {
            tasks_buf[i].task = task;
            tasks_buf[i].p = p;
            tasks_buf[i].t = t;
            tasks_buf[i].cyc = 0;
            return TRUE;
        }
    }
    return FALSE;
}
/**
 *
 */
uint8_t repeat_task_add(pTask task, void *p, uint32_t t, uint32_t cyc)
{
    for (uint8_t i = 0; i < TASKS_MAX_COUNT; i++)
    {
        if(tasks_buf[i].task == NULL)
        {
            tasks_buf[i].task = task;
            tasks_buf[i].p = p;
            tasks_buf[i].t = t;
            tasks_buf[i].cyc = cyc;
            return i;
        }
    }
    return 0xff;
}
/**
 *
 */
uint8_t repeat_task_del(uint8_t id)
{
    if(id < TASKS_MAX_COUNT)
    {
        tasks_buf[id].task = NULL;
        return TRUE;
    }
    return FALSE;
}
/**
 *
 */
void async_task_tick()
{
    for (uint8_t i = 0; i < TASKS_MAX_COUNT; i++)
    {
        if(tasks_buf[i].task != NULL)
        {
            if(tasks_buf[i].t > 0)
            {
                tasks_buf[i].t --;
            }
        }
    }
}
/**
 *
 */
void async_task_run()
{
    for (uint8_t i = 0; i < TASKS_MAX_COUNT; i++)
    {
        if (tasks_buf[i].task != NULL)
        {
            if(tasks_buf[i].t == 0)
            {
                tasks_buf[i].task(tasks_buf[i].p);
                if(tasks_buf[i].cyc != 0)
                {
                    tasks_buf[i].t = tasks_buf[i].cyc; //repeat
                }
                else
                {
                    tasks_buf[i].task = NULL; //once
                }
            }
        }
    }
}
/**
 *
 */
void async_task_clear()
{
    memset(tasks_buf,0,sizeof(tasks_buf));
}
```

#### 系统架构确定

1. RAM小于10K，基本没必要用OS，系统自身开销不小。
2. 硬件设计应该综合考虑成本、备货容易程度，替换代价，扩展性等，没必要过度冗余设计。
3. 裸机优先考虑状态机框架，参考QPFramework。
4. 先设计好日志调试系统，确定日志输出方式，然后先实现可靠的通信模块代码。

#### IAP系统设计

#### ITM调试系统设计

利用 ITM_SendChar 重载 printf

#### CAN链路层调试经验

1. 注意TJA1051芯片的朝向，丝印层无明显标记
2. 正常情况H，L电平都为2.5v
3. 不需要先隔离再到TJA1051，而是TJA1051之后再用防雷隔离器输出
4. 注意CAN引脚默认和USB共用，要重新映射
5. CAN波特率计算，APB1/(Prescaler*(bs1+bs2+rsjw))
6. 注意cubemx中，RX0中断是和USB低优先级中断共用，RX0,RX1对应的FIFO0和FIFO1中断，SCE中断是错误中断
7. HAL库正常接收时，实现回调`void HAL_CAN_RxFifo0MsgPendingCallback(CAN_HandleTypeDef *hcan)`就可以
8. 过滤器注意MASK还有FilterBank不要设置错了，初始化流程为，配置过滤器 -> 启动CAN -> 开启CAN中断

```c
#include "can.h"
#include <stm32f1xx_hal.h>
uint16_t id=0x0601;
void can_send(uint8_t *data,uint8_t sz)
{
    CAN_TxHeaderTypeDef tx_header;
    uint32_t mail_id;
    tx_header.RTR = CAN_RTR_DATA;
    tx_header.IDE = CAN_ID_STD;
    tx_header.StdId=id;
    tx_header.TransmitGlobalTime = DISABLE;
    tx_header.DLC = sz;
    if (HAL_CAN_AddTxMessage(&hcan, &tx_header, data, &mail_id) != HAL_OK)
    {
       /* Transmission request Error */
       Error_Handler();
    }
}
void can_ack(uint16_t id)
{
    CAN_TxHeaderTypeDef tx_header;
    uint8_t tx[] = {0x00};
    uint32_t mail_id;
    tx_header.RTR = CAN_RTR_REMOTE;
    tx_header.IDE = CAN_ID_STD;
    tx_header.StdId=id;
    tx_header.TransmitGlobalTime = DISABLE;
    if (HAL_CAN_AddTxMessage(&hcan, &tx_header, tx, &mail_id) != HAL_OK)
    {
       /* Transmission request Error */
       Error_Handler();
    }
}
void HAL_CAN_RxFifo0MsgPendingCallback(CAN_HandleTypeDef *hcan)
{
    CAN_RxHeaderTypeDef   rx_header;
    uint8_t    rx[8];
    if (HAL_CAN_GetRxMessage(hcan, CAN_RX_FIFO0, &rx_header, rx) != HAL_OK)
    {
    /* Reception Error */
        Error_Handler();
    }
    if ((rx_header.StdId == id) && (rx_header.IDE == CAN_ID_STD) && rx_header.DLC == 2)
    {
        if(rx[0]==0x01)
        {
            HAL_GPIO_WritePin(LED_GPIO_Port, LED_Pin, rx[1]);
            can_ack(0x602);
        }
    }
}
void setup()
{
    CAN_FilterTypeDef  sFilterConfig;
    sFilterConfig.FilterBank = 0;
    sFilterConfig.FilterMode = CAN_FILTERMODE_IDMASK;
    sFilterConfig.FilterScale = CAN_FILTERSCALE_32BIT;
//    sFilterConfig.FilterIdHigh = 0x601<<5;
    sFilterConfig.FilterIdHigh = 0x0000;
    sFilterConfig.FilterIdLow = 0x0000;
    sFilterConfig.FilterMaskIdHigh = 0x0000;
    sFilterConfig.FilterMaskIdLow = 0x0000;
    sFilterConfig.FilterFIFOAssignment = CAN_FILTER_FIFO0;
    sFilterConfig.FilterActivation = ENABLE;

    if (HAL_CAN_ConfigFilter(&hcan, &sFilterConfig) != HAL_OK)
    {
        Error_Handler();
    }
    if (HAL_CAN_Start(&hcan) != HAL_OK)
    {
        Error_Handler();
    }
    if (HAL_CAN_ActivateNotification(&hcan, CAN_IT_RX_FIFO0_MSG_PENDING) != HAL_OK)
    {
        Error_Handler();
    }
    while(1)
    {
        HAL_Delay(1000);
    }
}
```

完整代码查看：https://github.com/winxos/stm32_hal_can_minimal_demo

#### 可靠串口bsp实现

```c
void hl_uart_send(struct HLUart *up,u8* data,u32 len)
{
    HAL_UART_Transmit(up->huart, data, len, 0xffff);/*block sending*/
}
void hl_uart_irq(struct HLUart *up)
{
    __HAL_UART_CLEAR_IDLEFLAG((UART_HandleTypeDef*)up->huart);
    HAL_UART_DMAStop(up->huart);
    up->rx_buf.len = HL_MSG_SIZE - __HAL_DMA_GET_COUNTER(((UART_HandleTypeDef*)up->huart)->hdmarx);
    xQueueSendFromISR(up->rxq, &up->rx_buf, NULL);
}
HLUartOps _uart_ops={
        .interrupt=hl_uart_irq,
        .write=hl_uart_send
};
/*
 * create freertos queue,
 * because freertos's inner macro osMessageQDef and osMessageQ using ## symbol,
 * so you cann't pass variables through osMessageQDef or osMessageQ,
 * this macro combine the queue create procedure,
 * HANDLE is queue handle
 * SZ is the size you need
 * TYPE is the data type
 * N is the name
 * */
#define HL_QUEUE_INIT(HANDLE,SZ,TYPE,N) do{\
    osMessageQDef(N, SZ, TYPE);\
    HANDLE=osMessageCreate(osMessageQ(N), NULL);\
    }while(0)
/*
 * create unique name freertos queue,
 * using inner macro __COUNTER__ to create unique named queue each time you called,
 * HANDLE is queue handle
 * SZ is the size you need
 * TYPE is the data type
 * */
#define HL_QUEUE_INIT_UNIQUE(HANDLE,SZ,TYPE) HL_QUEUE_INIT(HANDLE,SZ,TYPE,__COUNTER__)

void hl_uart_init(void *p)
{
    struct HLUart *huart=(struct HLUart*)p;
    HL_QUEUE_INIT_UNIQUE(huart->rxq,3,struct HLMsg);/*it's very very hard to implementation*/
    HL_QUEUE_INIT_UNIQUE(huart->txq,2,struct HLMsg);
    __HAL_UART_ENABLE_IT((UART_HandleTypeDef*)huart->huart,UART_IT_IDLE);
    HAL_UART_Receive_DMA((UART_HandleTypeDef*)huart->huart,huart->rx_buf.data, HL_MSG_SIZE);
}
void hl_uart_loop(void *p)
{
    struct HLUart *up=(struct HLUart *)p;
    struct HLMsg t;
    if (xQueueReceive(up->rxq,&t,0) == pdTRUE)/*received queue*/
    {
        HAL_UART_Receive_DMA((UART_HandleTypeDef*)up->huart, up->rx_buf.data, HL_MSG_SIZE);
        struct HLFunLink * p=up->rxlist;
        while(p!=NULL)
        {
            p->executor(p->device,t.data, t.len);/*call callback link*/
            p=p->next;
        }
    }
    if (xQueueReceive(up->txq,&t,0) == pdTRUE)/*send queue*/
    {
        hl_uart_send(up, t.data, t.len);
        struct HLFunLink * p=up->txlist;
        while(p!=NULL)
        {
            p->executor(p->device,t.data, t.len);/*call callback link*/
            p=p->next;
        }
    }
}
struct HLDeviceOps uart_device_ops;
```

### 疑难问题汇编

#### Error: ST-LINK error (DEV_TARGET_NOT_HALTED)

**问题描述**

点击调式代码时出现上述错误，无法进行调试。

**关键参考**

[ST-LINK error (DEV_TARGET_NOT_HALTED)](https://community.st.com/s/question/0D50X0000B8jTfw/stlink-error-devtargetnothalted)

**问题代码**

```c
HAL_FLASH_Unlock();               //解锁Flash
FLASH_EraseInitTypeDef My_Flash;  //声明 FLASH_EraseInitTypeDef 结构体为 My_Flash
My_Flash.TypeErase = FLASH_TYPEERASE_SECTORS;
My_Flash.Sector = sector;
My_Flash.NbSectors = nb;
My_Flash.VoltageRange = VOLTAGE_RANGE_3;
uint32_t PageError = 0;
HAL_FLASHEx_Erase(&My_Flash, &PageError);  //调用擦除函数擦除
//HAL_FLASH_Lock(); //问题代码
```

**问题分析**

使用hal库操作flash时，调用了解锁函数，忘记上锁后，再调用解锁，写flash，会成此问题。

**解决办法**

将芯片擦除，修复问题代码，问题解决。

#### stlink gdb server 模式和OpenOCD模式区别

经过测试，OpenOCD 模式需要接rst引脚，gdbserver模式不需要

gdbserver模式有的时候会出现端口号被占用，无法连接，切换端口号也不行，此时切换到openocd模式可以调试。

#### stm32g系列芯片无法进入调试模式，调试时无法显示源代码

**问题描述**

硬件上boot0已经下拉接地，调式时状态异常。

**问题分析**

查看手册，发现g系列默认启动模式由 nBOOT_SEL等寄存器控制，并非由boot0的物理电平决定。

**解决办法**

在ST-LINK Utility中，点击 Target -> Option Bytes，取消nBOOT_SEL的勾选，问题解决。
