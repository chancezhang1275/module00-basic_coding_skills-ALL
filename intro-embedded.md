# 嵌入式的基本概念

author | Chance

contact | 603718892@qq.com

date| 26/ 02/ 22

> 这里是一个过去和最近学习的汇总, 不保证概念的正确性. 

[TOC]

## 0. 资料推荐

> 个人兴趣偏向嵌入式+人工智能, 这里每一项的推荐遵循由易到难顺序

书籍

1. 数字设计和计算机体系结构 ARM版本 哈里斯
2. 嵌入式C语言自我修养 从芯片, 编译器到操作系统 王利涛
3. ARM Cortex-M3与Cortex-M4权威指南 Joseph Yiu
4. 手把手教你设计CPU -- RISC-V处理器 胡振波
5. 智能计算系统 陈云霁
6. AI嵌入式系统 算法优化与实现 应忍冬 刘佩林 -> 可对比 [src](https://mp.weixin.qq.com/s?__biz=MzI4MDYzNzg4Mw==&mid=2247548328&idx=1&sn=f09db2ddeca2ee77d6502e9262f8c2f4&chksm=ebb7057cdcc08c6af5a3d5ec14780bf73e722b874798b344b1690513d6b1049e01ccffbcbe68&mpshare=1&scene=23&srcid=0214ILEBrC8ijEXDkU0a5Pnv&sharer_sharetime=1644835616481&sharer_shareid=a0f9c3c1132e58ec0f00a1617db2639b#rd) 学习



科普

1. AI sys [1](https://mp.weixin.qq.com/s/PQLQ0nN0fM4PPkhEUXfOmw), [2](https://mp.weixin.qq.com/s?__biz=MzU1OTEwNDkwNw==&mid=2247487618&idx=1&sn=368efa763499630046703226139ce64c&chksm=fc1d0666cb6a8f70f24aee1f90b9e47d8c6c1ca710d4934d9f3cc549ec0964d5bb7948f939a2&mpshare=1&scene=23&srcid=0226TrsGIUxiAOMUJ1iBB0kW&sharer_sharetime=1645815545511&sharer_shareid=a0f9c3c1132e58ec0f00a1617db2639b#rd) 



课程

1. 基于STM32CubeMX和HAL驱动库的嵌入式系统设计 UESTC [src](https://www.icourse163.org/course/UESTC-1207429802?outVendor=zw_mooc_pclszykctj_) 
2. 嵌入式软件设计 DUT [src](https://www.icourse163.org/course/DUT-1002607070?outVendor=zw_mooc_pclszykctj_) 
3. 编译原理 NUDT [src](https://www.icourse163.org/course/NUDT-1003101005?outVendor=zw_mooc_pclszykctj_) 
4. 龙芯杯 培训资料 [src](https://www.bilibili.com/video/BV1sZ4y1u7Fc?spm_id_from=333.337.search-card.all.click) 



项目

1. 智能计算系统-实验 [src](https://csdiy.wiki/%E4%BA%BA%E5%B7%A5%E6%99%BA%E8%83%BD/CYJ/) 
2. 一生一芯 [src](https://docs.ysyx.org/) 
3. 数字IC/数字电路/FPGA设计_从入门到精通 [src](https://ke.qq.com/course/3133628) 
4. H264 Encoder IP [src](http://openasic.org/category/1/announcements) 
5. ncnn 腾讯 nihui [src](https://github.com/Tencent/ncnn) 

> 注: 一生一芯片的文档还在建设中



## 1. 嵌入式的上上下下

### 1.1 物理上 + 逻辑上

> 这部分上次分享已经讲了, 可参考嵌C修养Chap02

物理上 | 材料 -> 制造 -> 封装

逻辑上 | 逻辑门 -> 微结构 -> CPU架构 + 总线 + 外围电路



### 1.2 代码视角

> 这里结合编译原理讲解, 大致对应嵌C修养Chap04
>
> 参考: 编译原理 NUDT [src](https://www.icourse163.org/course/NUDT-1003101005?outVendor=zw_mooc_pclszykctj_) 

编译 vs 解释

编译: 高级语言程序经过编译程序, 转换为机器语言程序, 在机器上运行, 从而得到结果. e.g. CPP

解释: 边解释边执行源程序. e.g. Python



高级语言 -> 机器语言 vs. 英语 -> 中文

过程: 

1. 词法分析: 识别句子中的一个个单词
2. 语法分析: 分析句子的语法结构
3. 中间代码产生: 根据句子的含义进行初步翻译
4. 优化: 对译文进行修饰
5. 目标代码产生: 写出翻译好的译文

这里, 目标代码主要有三种形式

- 汇编指令代码: 需要汇编器将其汇编, 转为`0101`这种机器码才能执行

- 绝对指令代码: 可直接运行, 指令的地址部分是绝对的地址

- 可重新定位指令代码: 需要链接才能转换为绝对指令代码

  代码中的地址是相对地址, 可以从内存的任何位置进行装载, 将代码中的相对地址加上装入的起始地址, 就能得到最后的绝对地址, 从而实现正确的访问 (链接过程)

  作用: 很好地支持模块化地开发, 使得每个程序模块都能够独立开发和编译, 然后通过链接程序, 将各个模块链接成可以执行的程序

  e.g. 

  ```assembly
  ; b = a + 2
  
  MOV a, R1 ; 0001 01 00 00000000*
  ADD #2, R1 ; 0011 01 10 00000010
  MOV R1, b ; 0100 01 00 00000100*
  ; 操作码 寄存器操作数 第二个操作数标志 第二个操作数(地址 or 立即数) *只起标识作用, 表明是相对地址
  ```

  装配时, `L=00001111`, 那么可执行文件为

  ```
  0001 01 00 00001111
  0011 01 10 00000010
  0100 01 00 00010011
  ```



## 2. 嵌入式系统

> 这里参考 基于STM32CubeMX和HAL驱动库的嵌入式系统设计 [src](https://www.icourse163.org/course/UESTC-1207429802?outVendor=zw_mooc_pclszykctj_) 

### 2.1 嵌入式系统硬件

嵌入式系统硬件 == 处理器 + 外围电路(内置 + 外置)

#### 2.1.1 处理器的名称

- CPU | 中央处理器

- MCU | 微控制器, CPU加一些内置外设, 可以跑简单RTOS如uCOS-ii

- MPU | 微处理器, 更好的CPU加更多的外设, 有MMU/MPU, 可以跑Linux

- SoC | 片上系统, 这个名词一般在IC设计领域用得多, 除了常见的外设外, 再集成了各种DSP, ISP, NPU, Encoder, FPGA

#### 2.1.2 处理器-关于STM32和OpenMV的介绍

- ARM是架构公司, 提供处理器的IP, ARM型号的部分表格

  |  架构   |                 处理器                 |                指令集                |   NEON   |           OS            |                  例子                   |
  | :-----: | :------------------------------------: | :----------------------------------: | :------: | :---------------------: | :-------------------------------------: |
  |  ARMv6  |                M0+/0/1                 |              32位指令集              |  不支持  |                         |                                         |
  |  ARMv7  |       A性能<br>R实时<br>M3/4平衡       |    32位指令集A32<br>16位指令集T16    | 可选支持 |         uCOS-ii         | STM32 M3-F1<br>M4-F4, L432<br>M7-OpenMV |
  | ARMv8-A | A32/35节能<br>A53平衡<br>A57/72/73性能 | 64位指令集AArch64<br>兼容32位AArch32 | 默认支持 | Linux<br>Android<br>iOS |               ZYNQ AXU4EV               |

  注: M4内核因为没有MMU(存储器管理单元)或MPU(存储器保护单元), 所以不能跑通用的OS如Linux, 但可以运行精简的RTOS如uCOS-ii

- Micro-Python是基于ARM结构的编译器和runtime environment;  

- ST使用ARM的IP设计MCU; 

- OpenMV创始人使用Micro-Python为STM32 H7开发了一套主要与视觉相关的HAL, 便于使用. 

#### 2.1.3 外围电路

- 环境感知 | 温湿度传感器, 加速度传感器, 压力传感器, ...
- 接口 | A/D接口, 同步/异步串口, USB接口, ...
- 存储 | SRAM, DRAM, NANA/Nor Flash
- 人机交互 | 指示灯/ 数码管, 液晶显示屏, 键盘



### 2.2 嵌入式系统软件

```
应用程序
(操作系统)
(硬件抽象层HAL)
驱动程序
---
嵌入式硬件平台
```

使用HAL层典型: Arduino, Mbed, Micro-Python(OpenMV)

> BSP板级支持包: HAL的一种实现形式; 
>
> 例如`BSP_LED_Init`, `BSP_LED_On`, `BSP_LED_Off`等接口; 

> STM32的学习: 寄存器版本 vs HAL版本
>
> 寄存器: 更了解细节, 但是STM32外设多, 需要用户了解每个寄存器功能和寄存器每一位的定义 -> 难顶
>
> 固件库: 会调用就行, 但是程序的效率低一点, 不过现在处理器性能普遍较快, 不影响



### 2.3 嵌入式编程模式

#### 2.3.1 前后台模式

```c
 while(1) do{ //后台
  	Task1
  	Task2
  	Task3
          ...//中断(前台)
      Task3
  	Task4
          ISR
          	ISR//中断嵌套(前台)
          ISR
      Task4
  }
```

硬中断: 电信号 -> 中断控制器 -> 处理器引脚 -> 处理器停止目前任务, 跳转中断程序处理入口点处理中断

```c
 /***
  温度采集系统 - 前后台编程实现
  1. 每隔1s进行温度的采集; 
  2. 温度显示在数码管上; 
  3. 可以通过键盘设置温度上/下限; 
  4. 温度超过上/下限, 声光报警; 
  ***/
  
  int main(){
      System Initialize;
      while(1){
          if( Flag1s == 1 )
              GetTemperature();
          if( Flag20ms == 1)
              DispTemperature();
          if( Flag10ms == 1)
              ReadKey();
          if( AlarmFlag == 1)
              SendAlarm();
      }
  }
  // 这里的三个标志Flag1s, Flag20ms, Flag10ms都由定时器中断产生;
  // AlarmFlag标注由温度采集程序产生标志;
```

即用户需要编写: 

1. 定时中断程序
2. 定义标志变量
3. 判断标志变量并清除

> 这里可以把巡线的任务作为后台, 检测到其它信号量就执行中断. 

#### 2.3.2 操作系统模式

暂时不谈, 原因: 此次项目硬件平台为OpenMV, 不支持Linux系统. 

> OpenMV的HAL是Micro-Python, 足够此次任务; 
>
> 但是控制部分的STM32F1可以尝试使用uCOS-ii;
>
> 需要的时候再进行讲解; 



