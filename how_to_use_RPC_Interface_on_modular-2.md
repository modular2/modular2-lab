# RPC接口相关实验
## RPC概述
### 什么是RPC？
Remote Procedure Call (RPC)远程过程调用允许计算机程序在另一台计算机上执行子程序。它通常用于计算设备的网络中。在Mbed系统中，您可以通过在终端或浏览器上简单地调用主机上的变量或函数的名称来操纵变量并在Mbed上执行子程序。<br>
RPC范例允许用其他语言编写的程序与Mbed通信。例如，可以使用Python、Java、Matlab等语言中的库，这使得基于GUI的命令在MBED上执行。
### 如何快速实现RPC接口
1. Modular-2的Mbed RPC接口，可使用电脑通过终端程序USB串口连接。<br>
终端设置为：CR+LF, 9600, 8数据位，无奇偶位，1停止位，本地回显。<br>
2. Modular-2的Mbed RPC还能够通过RPC接口库与主流编程言进行连接。<br>
这些库允许其他程序通过RPC直接通信到Mbed，而不需要传输设置或格式化RPC命令。相关库可用于Matlab、LabVIEW、Python、Java和.NET。
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
RPC接口库主要类:
* RPCFunction – 通过RPC调用mbed 自定义函数的类；
* RPCVariable – 使用RPC读写mbed自定义变量的类；
* SerialRPCInterface – 使用RPC调用串口的类。
