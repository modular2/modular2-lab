# 实验-8 Mbed的串口RPC调用
## 实验目的
了解远程过程调用(RPC)的实现机制，实验使用modular-2通过串口进行RPC调用。
## 实验设备
### 硬件：
+ Modular-2一台（V1.3）
+ PC电脑一台
### 软件：
+ WINDOWS 7操作系统
+ SecureCRT 7终端仿真程序
## 实验原理
### 什么是RPC？
Remote Procedure Call (RPC)远程过程调用允许计算机程序在另一台计算机上执行子程序。它通常用于计算设备的网络中。在Mbed系统中，您可以通过在终端或浏览器上简单地调用主机上的变量或函数的名称来操纵变量并在Mbed上执行子程序。

RPC范例允许用其他语言编写的程序与Mbed通信。例如，可以使用Python、Java、Matlab等语言中的库，这使得基于GUI的命令在MBED上执行。

### 如何快速实现RPC接口
Modular2的Mbed RPC接口，可使用电脑通过终端程序USB串口连接。<br>
终端设置为：CR+LF, 9600, 8数据位，无奇偶位，1停止位，本地回显。

Modular2 的Mbed RPC还能够通过RPC接口库与主流编程言进行连接。这些库允许其他程序通过RPC直接通信到Mbed，而不需要传输设置或格式化RPC命令。相关库可用于Matlab、LabVIEW、Python、Java和.NET。

### RPC接口库（mbed-rpc）
使用mbed-rpc的接口遵循以下命令格式：```RPC::call(buf, outbuf);```

外部调用命令如下：

|命令|返回结果|
| :------:| :------:|
|Enter	|返回所有可用的RPC对象名|
|"/<对象名> RPC"|	返回该对象下可用的RPC方法|
|"<对象名>/<方法名> <参数(多参数空格隔开)>"	|使用输入的参数执行该对象的方法|

注意!<br>
如果你要使用你自定义的对象，请确保mbed已经定义该对象。<br>
方法没有参数时，需要加空格做为空参数。<br>
"RPCobject/read " 执行，"RPCobject/read" 不执行。<br>
### RPC接口库主要类
* RPCFunction – 通过RPC调用mbed 自定义函数的类；
* RPCVariable – 使用RPC读写mbed自定义变量的类；
* SerialRPCInterface – 使用RPC调用串口的类。
## 实验内容
### 代码
```
#include "mbed.h"
#include "mbed_rpc.h"
// RpcDigitalOut实例化PC_6引脚，更多请查阅RpcClasses.h
RpcDigitalOut myled(PC_6,"myled");

Serial pc(USBTX, USBRX);//串口申明
int main() {
    
    char buf[256], outbuf[256];
    while(1) {
        pc.gets(buf, 256);        //RPC接口调用
        RPC::call(buf, outbuf); 
        pc.printf("%s\n", outbuf);
    }
}
```
### 串口终端SecureCRT设置
运行SecureCRT设置为串口模式，选择Modular-2在系统中生成的串口，通过数据线连接Modular-2，打开互动窗口后，进行SecureCRT会话设置。
### RPC串口命令调用
<<<<<<< HEAD
<<<<<<< HEAD
使用串口终端发送以下RPC命令将打开Modular2的LED6绿灯。<br>
```
/myled/write 0
/myled/write 1
 ```
![RPC串口命令调用](/modular2-lab/raw/master/screenshots/rpc_over_serial_command.jpg)
=======
使用串口终端发送以下RPC命令将打开Modular2的LED6绿灯。
```/ myled /write 0 ```

>>>>>>> parent of 6f02190... Update Lab8_RPC_Over_Serial.md
=======
使用串口终端发送以下RPC命令将打开Modular2的LED6绿灯。
```/ myled /write 0 ```

>>>>>>> parent of 6f02190... Update Lab8_RPC_Over_Serial.md
