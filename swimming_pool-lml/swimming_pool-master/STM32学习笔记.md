# STM32学习笔记

* 安装KEIL5
* 安装STM32芯片包
* 用DAP仿真器下载程序
  * 硬件连接
    * 用USB线连接，开发板必须要**供电**
* 用串口下载程序
  * 下载CH340驱动
  * 打开mcuisp软件
    * 搜索串口，设置波特率115200
    * 选择HEX文件
    * 校验，编程后执行
    * **DTR低电平复位，RTS高电平进入bootloader**
    * 开始编程
* ISP简介

> ISP在系统可编程，指电路板上的空白器件可以编程写入最终用户代码，而不需要从电路板上取下器件，已经编程的器件也可以用ISP方式擦除或再编程。

* BOOT配置

| BOOT0 | BOOT1 |  启动方式  |
| :---: | :---: | :--------: |
|   0   |   X   | 内部FLASH  |
|   1   |   0   | 系统存储器 |
|   1   |   1   |  内部SRAM  |

* STM32--ST公司开发的32位微控制器

|           | STM32   F   103   V   E   T   6                              |
| :-------: | :----------------------------------------------------------- |
|   家族    | STM32表示32bit的MCU                                          |
| 产品类型  | F表示基础型                                                  |
| 具体特性  | 基础型                                                       |
| 引脚数目  | V表示100pin，其他常用的为C表示48，R表示64，Z表示144，B表示208，N表示216 |
| FLASH大小 | E表示512KB，其他常用的为C表示256，E表示512，I表示2048        |
|   封装    | T表示QFP封装，这个是最常用的封装                             |
|   温度    | 6表示温度等级为A：-40~85度                                   |

* 选择合适的MCU
* 分配原理图IO
  * 引脚分类
* 寻找IO功能说明
  * 参考手册
  * 数据手册
* STM芯片
  1. ICode总线--**取指**
  2. 驱动单元
     * DCode总线--**取数**
     * 系统总线--**读写寄存器**
     * DMA总线--**传输数据**
  3. 被动单元
     * 内部的闪存存储器--**FLASH**
     * 内部的SRAM
     * FSMC--**扩展静态内存**
     * AHB到APB的桥
       * 从AHB总线延伸出APB1,APB2两条总线--挂载GPIO、串口、I2C、SPI等外设
       * 学习重点--**学会编程这些外设去驱动外部的各种设备**
* 存储器映射--给存储器**分配地址**的过程
* 存储器功能分类--4GB-8块

|    序号    |             用途             |              地址范围              |
| :--------: | :--------------------------: | :--------------------------------: |
| **Block0** |  **Code**（设计片内FLASH）   | **0x0000 0000~0x1FFF FFFF(512MB)** |
| **Block1** |   **SRAM**（设计片内SRAM）   | **0x2000 0000~0x3FFF FFFF(512MB)** |
| **Block2** | **片上外设**（设计片内外设） | **0x4000 0000~0x5FFF FFFF(512MB)** |
|   Block3   |      FSMC的bank1~bank2       |   0x6000 0000~0x7FFF FFFF(512MB)   |
|   Block4   |      FSMC的bank3~bank4       |   0x8000 0000~0x9FFF FFFF(512MB)   |
|   Block5   |          FSMC寄存器          |   0xA000 0000~0xBFFF FFFF(512MB)   |
|   Block6   |           没有使用           |   0xC000 0000~0xDFFF FFFF(512MB)   |
|   Block7   |      Cortex-M3内部外设       |   0xE000 0000~0xFFFF FFFF(512MB)   |



* Block2内部区域功能划分

|   块   |   用途说明   |        地址范围         |
| :----: | :----------: | :---------------------: |
|        | APB1总线外设 | 0x4000 0000~0x4000 77FF |
| Block2 | APB2总线外设 | 0x4001 0000~0x4001 3FFF |
|        | AHB总线外设  | 0x4001 8000~0x5003 FFFF |

* 寄存器映射

  * 通过绝对地址访问内存单元--易出错
  * 通过寄存器别名访问内存单元

  ```c++
  1 // GPIOB 端口全部输出高电平
  2 #define GPIOB_ODR			*(unsigned int*)(GPIOB_BASE+0X0C)
  3 GPIOB_ODR=0XFF;
  ```


* **C语言对寄存器的封装**

> **总线和外设基址宏定义**

```c++
1 /*外设基地址*/
2 #define PERIPH_BASE			((unsigned int)0x40000000)
3 /*总线基地址*/
4 #define APB1PERIPH_BASE		(PERIPH_BASE)
5 #define APB2PERIPH_BASE		(PERIPH_BASE+0X00010000)
6 #define AHBPERIPH_BASE		(PERIPH_BASE+0X00020000)
7 /*GPIO外设基地址*/
8 #define GPIOA_BASE			(APB2PERIPH_BASE+0X0800)
9 #define GPIOB_BASE			(APB2PERIPH_BASE+0X0C00)
10 #define GPIOC_BASE			(APB2PERIPH_BASE+0X1000)
11 #define GPIOD_BASE			(APB2PERIPH_BASE+0X1400)
12 #define GPIOE_BASE			(APB2PERIPH_BASE+0X1800)
13 #define GPIOF_BASE			(APB2PERIPH_BASE+0X1C00)
14 #define GPIOG_BASE			(APB2PERIPH_BASE+0X2000)
15 /* 寄存器基地址，以GPIOB为例*/
16 #define GPIOB_CRL			(GPIOB_BASE+0X00)
17 #define GPIOB_CRH			(GPIOB_BASE+0X04)
18 #define GPIOB_IDR			(GPIOB_BASE+0X08)
19 #define GPIOB_ODR			(GPIOB_BASE+0X0C)
20 #define GPIOB_BSRR			(GPIOB_BASE+0X10)
21 #define GPIOB_BRR			(GPIOB_BASE+0X14)
22 #define GPIOB_LCKR			(GPIOB_BASE+0X18)
```

> **使用指针控制BSRR寄存器**

```c++
1 /*控制GPIOB 引脚0输出低电平（BSRR寄存器的BRO置1）*/
2 *(unsigned int*)GPIOB_BSRR=(0X01<<(16+0));
3 /*控制GPIOB 引脚0输出高电平（BSRR寄存器的BSO置1）*/
4 *(unsigned int*)GPIOB_BSRR=0X01<<0;
5 unsigned int temp;
6 /*读取GPIOB端口所有引脚的电平（读IDR寄存器）*/
7 temp=*(unsigned int*)GPIOB_IDR;
```

> **使用结构体对GPIO寄存器组的封装**

```c++
1 typedef unsigned int uint32_t;/*无符号32位变量*/
2 typedef unsigned short int uint16_t;/*无符号16位变量*/
3 /*GPIO寄存器列表*/
4 typedef struct{
5		uint32_t CRL;	/*GPIO端口配置低寄存器	地址偏移：0x00*/
6		uint32_t CRH;	/*GPIO端口配置高寄存器	地址偏移：0x04*/
7		uint32_t IDR;	/*GPIO数据输入寄存器	地址偏移：0x08*/
8		uint32_t ODR;	/*GPIO数据输出寄存器	地址偏移：0x0C*/
9		uint32_t BSRR;	/*GPIO位设置/清除寄存器	地址偏移：0x10*/
10		uint32_t BRR;	/*GPIO端口位清除寄存器	地址偏移：0x14*/
11		uint16_t LCKR;	/*GPIO端口配置锁定寄存器	地址偏移：0x18*/
12 }GPIO_TypeDef;
```

> **通过结构体指针访问寄存器**

```c++
1 GPIO_TypeDef*GPIOx;		//定义一个GPIO_TypeDef型结构体指针GPIOx
2 GPIOx=GPIOB_BASE;			//把指针地址设置为宏GPIOB_BASE地址
3 GPIOx->IDR=0xFFFF;
4 GPIOx->ODR=0xFFFF;
5 uint32_t temp;
6 temp=GPIOx->IDR;			//读取GPIOB_IDR寄存器的值到变量temp中

```

> **定义好GPIO端口首地址指针**

```c++
1 /*使用GPIO_TypeDef把地址强制转换成指针*/
2 #define GPIOA				((GPIO_TypeDef*)GPIOA_BASE)
3 #define GPIOB				((GPIO_TypeDef*)GPIOB_BASE)
4 #define GPIOC				((GPIO_TypeDef*)GPIOC_BASE)
5 #define GPIOD				((GPIO_TypeDef*)GPIOD_BASE)
6 #define GPIOE				((GPIO_TypeDef*)GPIOE_BASE)
7 #define GPIOF				((GPIO_TypeDef*)GPIOF_BASE)
8 #define GPIOG				((GPIO_TypeDef*)GPIOG_BASE)
9 #define GPIOH				((GPIO_TypeDef*)GPIOH_BASE)
10 /*使用定义好的宏直接访问*/
11 /*访问GPIOB端口的寄存器*/
12 GPIOB->BSRR=0xFFFF;			//通过指针访问并修改GPIOB_BSRR寄存器
13 GPIOB->CRL=0xFFFF;			//修改GPIOB_CRL寄存器
14 GPIOB->ODR=0xFFFF;			//修改GPIOB_ODR寄存器
15 uint32_t temp;
16 temp=GPIOB->IDR;				//读取GPIOB_IDR寄存器的值到变量temp中
17 /*访问GPIOA端口的寄存器*/
18 GPIOA->BSRR=0xFFFF;			
19 GPIOA->CRL=0xFFFF;			 
20 GPIOA->ODR=0xFFFF;			 
21 uint32_t temp;
22 temp=GPIOA->IDR;				//读取GPIOA_IDR寄存器的值到变量temp中
 
```

