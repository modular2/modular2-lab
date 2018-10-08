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
### RPC的串口连接
Modular-2的Mbed RPC接口，可使用电脑通过终端程序USB串口连接。<br>
终端设置为：CR+LF, 9600, 8数据位，无奇偶位，1停止位，本地回显。<br>
### RPC接口库（mbed-rpc）
使用mbed-rpc的接口遵循以下命令格式：<code>RPC::call(buf, outbuf);</code><br>
外部调用命令如下：

|命令|	返回结果|	示例|
| ------| ------ | :------: |
|Enter	|返回所有可用的RPC对象名| |	
|"/<对象名> RPC"|	返回该对象下可用的RPC方法| | 	
|"<对象名>/<方法名> <参数(多参数空格隔开)>"	|使用输入的参数执行该对象的方法| |	

注意!<br>
如果你要使用你自定义的对象，请确保mbed已经定义该对象。<br>
方法没有参数时，需要加空格做为空参数。<br>
"RPCobject/read " 执行，"RPCobject/read" 不执行。<br>
### RPC接口库主要类
* RPCFunction – 通过RPC调用mbed 自定义函数的类；
* RPCVariable – 使用RPC读写mbed自定义变量的类；
* SerialRPCInterface – 使用RPC调用串口的类。
## 实验内容

