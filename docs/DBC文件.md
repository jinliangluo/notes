# DBC文件



## 1. DBC定义

​	DBC（data base CAN）是汽车ECU间进行CAN通讯的报文内容;



## 2. DBC文件查看

​	DBC是文本文件，可以用记事本打开，一般都用CANdb++，可以更方便的查看和编辑。	



## 3. DBC文件组成

​	DBC是由一系列的Message和Signal组成，文件定义了Message和Signal的属性，可参考vector文档（回复“DBC文档”获取）。下面介绍几个重要的关键字：

### 3.1 BO_

`BO_` 是对message的定义;

+ 格式:  `BO_ ID Name: DLC Transmiter`
+ 例子: `BO_ 100 ESP_01:8 ESP`
+ 释义: 发送方=ESP，帧名称=ESP_01，帧ID=0x64，报文长度=8个字节



### 3.2 SG_

`SG_`是对Signal的定义。

+ 格式： `SG_ Name : StartBit | Length @ ByteOrder SignedFlag (Factor,Offset) [Minimum | Maximum] "Unit" Receiver1,Receiver2`
+ 例子：SG_ VehSpd : 7|16@0+ (0.01,0) [0|655.35] "km/h" ECM.TCM
+ 释义：信号名称=VehSpd，起始地址=7，长度=16，字节顺序=MSB（大端），符号位=无符号，系数=0.01，偏移=0，最小值=0，最大值=655.35，单位=km/h，接收方=ECM和TCM



### 3.3 VAL_

VAL_是对Signal枚举值的定义。

+ 格式：VAL_ ID Name key1 "value1" key2 "value2" ;
+ 例子：VAL_ 100 VehSpdValid 1 "Valid" 0 "Invalid" ;
+ 释义：帧ID=0x64，信号名称=VehSpdValid，枚举值（0x0=Invalid，0x1=Valid）