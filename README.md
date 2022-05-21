# Cubli_Mini项目

本项目ID是来源于苏黎世联邦理工学院的Cubli，电机驱动使用的是Simple FOC。低成本、小型化、简化后作为我的首个开源项目

**项目完全开源，任何人都可以根据本Github内容自行白嫖，当然也可以顺手点个Star**

### 介绍

本项目在原版Cubli的基础上去掉主动刹车，缩小体积和一定的简化。需要手动调整到平衡点附近，才可以实现边和点平衡。Cubli_Mini是一个10x10x10CM的立方体，集电机驱动、充放电一体化。满电状态下，点平衡在无扰动下续航≥5小时。

项目成本：≤800RMB

**视频介绍：**[https://www.bilibili.com/video/BV1UR4y1c74B?spm_id_from=333.999.0.0](https://www.bilibili.com/video/BV1UR4y1c74B?spm_id_from=333.999.0.0)

YouTube：[https://youtu.be/JwiJBd6I_vY](https://youtu.be/JwiJBd6I_vY)

![Image](https://github.com/ZhaJiHu/Cubli_Mini/blob/master/5.Doc/Pic/Cubli_Mini.JPG)

### 资料更新说明（2022/5/21）

- /6.Process/钣金件/，增加对应的step文件
- /6.Process/3D打印/，增加对应的step文件
- **安全说明**
  - **动量轮比较危险，要注意安全，手远离动量轮**
  - /2.Firmware/MCU1/control/cubli_mini.h，修改VOLTAGE_LIMIT的电压可以限制飞轮的最大输出，开始使用建议降低该参数，可以从4，5V开始

### 资料更新说明（2022/5/19)

- 最新发现动量轮有两个角忘了倒角了，理论上会影响大转速下的动平衡，因此对此进行修改。已经下单打样的同学也可以正常使用，因为电机KV值很低，我现在使用的也是漏倒角的动量轮。更新了/4.Model/Cubli_Mini.STEP文件。并且更新了/6.Process/钣金件/YC.JGJ.000002.4.*，文件版本号更新到.4
- 还有说一下结构件的采购数量和采购链接都在/6.Process/BOM/结构BOM.xls
- 物料采购的时候注意编码器模块需要3套物料

### 资料说明

- 3D模型设计源文件
- 主控模块、IMU模块、编码器模块、下载模块硬件工程
- 主控1、主控2软件工程
    - 主控模块使用两颗ESP32
- 网络调试助手
    - WIFI调参使用
- 各类文档说明
    - 对项目的一些说明，如调参，按键操作，控制原理等
- 打样文件
    - STL格式3D打印文件
    - DWG格式钣金文件
    - BOM表
        - 结构BOM表
        - PCB元器件BOM表

### 结构设计

由于我只学了一周的SW，因此有些零件会漏画一些内容，如螺丝螺纹，铜柱内螺纹啥的

外框等钣金件采用的是成本较低的玻纤板，在保证刚性的情况下降低成本。动量轮采用的是304不锈钢，可以在小体积下提供较大的转动惯量。辅以少量3D打印件组成。连接件为了美观使用的是阳极氧化铝柱，当然也可以自行使用成本更低的铜柱。整体结构件成本为350块以内，铝柱更换铜柱可以降低到290以内

**注意事项：**

- Cubli_Mini的八个角点，是由两种不同的角组成，名字为角1和角2，各4个
- 外框和角的连接处的结构不是45°对称，所以是有安装方向的，详情看DoC/结构和安装说明

### 电路模块

**主控模块：**

- 主控模块集成了3个电机驱动电路，电池充放电路，降压稳压电路，CAN通讯电路
- 由于单颗ESP32只有6路电机PWM，因此使用了两颗ESP32模块，使用了CAN作为MCU间的通讯方式，由于两颗ESP32都是集成在同一个主板上，因此为了节省IC成本，可以把CAN修改为IIC，UART，SPI等各类短路径通讯方式，但是需要修改主控软硬件，有想法的同学可以自行修改
- 电机驱动电路参考Simple FOC开源电路，Simple FOC链接：[https://github.com/simplefoc](https://github.com/simplefoc)。电池充放电参考：[https://oshwhub.com/qqj1228/zi-ping-heng-di-lai-luo-san-jiao_10-10-ban-ben](https://oshwhub.com/muyan2020/zi-ping-heng-di-lai-luo-san-jiao_10-10-ban-ben_copy)

**IMU模块：**

- 使用最为常见的MPU6050
- 为了防止应力，对IMU所在区域挖槽，减少安装应力。但使用了3颗螺丝硬连接，并不能避免应力问题，改进方案为使用软垫片并只拧紧一个螺丝进一步减少应力
- 由于IMU安装要求比较高，因此在贴片时，尽量保证MPU6050与PCB平行，且位于焊盘正中央

**编码器模块：**

- 采用的是成本较低的AS5600，在低速情况下便宜好用

**自动下载模块：**

- 为了尽量减少主控的面积，因此单独把自动下载电路摘出来，参考官方自动下载电路

**注意事项：**

- 正常情况下，自动下载模块焊接排针，主控不焊接任何排针/排母

### 固件说明

Cubli_Mini的核心内容为Simple FOC电机驱动库以及点平衡控制算法，运行在MCU1中。MCU2只是使用了Simple FOC库对电机2和电机3的控制，利用CAN反馈和接收控制命令。

**MCU1固件说明：**

- bsp：基于arduino库封装了一下LED，KEY，ADC，非易失储存等内容
- comm：公用类，目前只封装了一个时间类，获取时间差，非延时等待等简单内容
- control：核心库，内容为平衡算法和调参接口，调参参考Simple FOC格式。为方便使用，使用的是字符串，不使用传统的16进制协议
- main：main文件，驱动的初始化和FreeRTOS任务创建
    - ESP32官方把WIFI，蓝牙类运行在Core0。因此在创建任务中，把WIFI任务创建在Core0中，其他则创建在Core1。只有运行于不同核心任务的数据交互我才加了互斥锁。

**注意事项：**

- 目前WIFI SSID和PSW，TCP Server IP Port全部是写死在固件里面，因此要想使用WIFI调参需要修改这些内容，详情请看Doc/使用和调参说明

### 打样说明

考虑到很多同学可能是第一次进行结构打样，因此提供了打样文件

**3D打印：**

- 文件位于：/6.Process/**3D打印**，该文件为STL格式，打样数量请查看同目录下的<**3D打印加工说明**>，可以在未来工厂或者嘉立创打样，首选光固化
- 文件名称使用的是料号，料号方便管理，尾号没有.X都代表为第一版

**钣金件打样：**

- 文件位于：/6.Process/**钣金件**，**每个物料的都有对应的PDF和DWG文件，PDF文件方便查看，DWG文件用于激光切割/板材加工**。打样数量请查看同目录下的<**钣金加工说明**>，可以找淘宝给图纸直接加工。由于加工精度要求不高，个人用途下，公差全看加工厂的心情
- 也可以直接找到对应的step文件给淘宝加工

**PCB打样：**

- 从硬件工程拿着PCB文件出门左转嘉立创白嫖

**BOM表：**

- 文件位于：/6.Process/**BOM**
- 结构BOM表，里面列有需要购买的**结构物料和链接**
- PCB BOM表，里面列有需要购买的元器件物料，也可以自行找到AD工程导出
- **注意事项：**
    - PCB BOM表是从AD源文件导出，工程会有重复用料，需要注意
    - **注意编码器需要3块，需要采购3套物料**
    - DRV8313、ESP32、AS5600、IMU可以去淘宝购买，其他右转立创商城BOM表下单。
    - **注意贴片型铝电解电容的封装，主控板有限高区域，限高高度为8mm**
    - 如文件有遗漏的地方望提醒

**写给小白的白嫖教程：**

- 文件位于：/Doc/写给小白的白嫖教程.pdf，手把手教你如何完成本项目的白嫖

### 控制算法

Cubli_Mini使用的是LQR来进行控制，也可以使用串级PID来进行控制。代码位于/需要修改/MCU1/control/cubli_mini

**边平衡**：

- 可以参考一阶倒立摆进行建模和求解参数。由于建立模型的参数比较难准确的获得，我求解得到的参数和最佳参数有些许出入。因此本项目大多数使用玄学调参工程师来整定参数

**点平衡：**

- 在边平衡的基础上添加对X，Z轴的控制。采用与边平衡一样的控制方法，详情看Doc/控制原理说明。

**研究型：**

- 请查看原版Cubli的论文

### 调参

**调节方法：**

- 可以使用UART和WIFI进行调参，WIFI需要连接到路由器，Cubli_Mini为TCP客户端

**调节命令：**

- 详情命令请查看Doc/调参命令说明