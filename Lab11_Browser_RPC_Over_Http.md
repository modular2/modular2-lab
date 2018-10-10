# 实验-11 Mbed RPC的HTTP调用
## 实验目的
了解远程过程调用(RPC)的实现机制，实验使用modular-2通过以太网进行RPC的HTTP调用。
## 实验设备
### 硬件：
+ Modular-2一台（V1.3）
+ PC电脑一台
### 软件：
+ WINDOWS 7操作系统
+ SecureCRT 7终端仿真程序
+ chrome浏览器
## 实验原理
### 什么是RPC？
Remote Procedure Call (RPC)远程过程调用允许计算机程序在另一台计算机上执行子程序。它通常用于计算设备的网络中。在Mbed系统中，您可以通过在终端或浏览器上简单地调用主机上的变量或函数的名称来操纵变量并在Mbed上执行子程序。

RPC范例允许用其他语言编写的程序与Mbed通信。例如，可以使用Python、Java、Matlab等语言中的库，这使得基于GUI的命令在Mbed上执行。

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
### TCPSocket类
提供通过TCP发送数据流的能力。TCPSocket维护一个以connect成员函数开始的有状态连接。成功连接到服务器后，您可以使用send和recv成员函数来发送和接收数据。
构造函数接受ethernet stack指针以打开指定EthernetInterface上的套接字。如果你没有传入构造函数，那么你必须调用open来初始化套接字。
有关TCP服务器功能，请参阅TCPServer类。
## 实验内容
实现RPC的HTTP调用使Modular2 LED6闪烁。<br>
命令格式：http://<mbed URL地址>/rpc/<对象名>/<方法名> <参数(多参数空格隔开)><br>
/rpc/为自定义句柄。
### 代码
```
#include "mbed.h"
#include "EthernetInterface.h"
#include "TCPServer.h"
#include "TCPSocket.h"
#include "mbed_rpc.h"
#include <string>
using namespace std;
RpcDigitalOut myled(PC_6,"myled");
int main() {
    printf("RPC Over Http example\n");
    EthernetInterface eth;
    eth.set_network("192.168.11.122","255.255.255.0","192.168.1.1"); //设置静态IP地址
    printf("%d \n",eth.connect());//打印连接状态 0为成功
    printf("MAC Address is %s \n",eth.get_mac_address());//输出MAC地址
    printf("The Server IP address is '%s'\n", eth.get_ip_address());//输出IP地址
    TCPServer srv;
    TCPSocket clt_sock;
    SocketAddress clt_addr;    
    srv.open(&eth);/* Open the server on ethernet stack */
    srv.bind(eth.get_ip_address(), 80);/* Bind the HTTP port (TCP 80) to the server */
    printf("Listening...\n");  
    srv.listen(5);  /* 设置连接并发数为5 */
    char buf[256],outbuf[256];
    while (true) {
        srv.accept(&clt_sock, &clt_addr);        
        printf("----------------------------------------\n");
        printf("RPC Request from %s:%d\n", clt_addr.get_ip_address(), clt_addr.get_port());  //打印客户端地址及端口
        int rcount = clt_sock.recv(buf,256);
        char *RPC_STR=strstr(buf,"/rpc");//自定义RPC句柄
        //简单的请求字符处理
        string RPC_STR1,RPC_STR2,Out_RPC_STR;
        if (RPC_STR!=NULL) {
            RPC_STR=RPC_STR+4;            
            strtok(RPC_STR," HTTP");
            RPC_STR1=strtok(RPC_STR,"%");
            RPC_STR2=strtok(NULL,"%");
            if (RPC_STR2!="") {
                RPC_STR2=RPC_STR2.substr(2,(sizeof(RPC_STR2)-1));
                Out_RPC_STR=RPC_STR1+" "+RPC_STR2;
            } else {
                Out_RPC_STR=RPC_STR1+" ";
            }
            char *tbuf;
            strcpy(tbuf, Out_RPC_STR.c_str());

            RPC::call(tbuf, outbuf);//mbed_rpc 调用

            printf("RPC Request Data: %s\n", tbuf);
            printf("RPC Return  Data: %s\n",outbuf);
            string HTML_Content = "<HTML><HEAD><meta http-equiv=\"content-type\" content=\"text/html;charset=utf-8\"><link rel=\"icon\" href=\"data:;base64,=\"></HEAD><BODY><h1>"+string(outbuf)+"</h1></BODY></HTML>\r\n";
            char *http_send ;
            strcpy(http_send,HTML_Content.c_str());
            clt_sock.send(http_send, strlen(http_send));//TCP Socket输出
        }	// CHROME浏览器访问回显
    }
}

```
### 串口终端SecureCRT设置
PC电脑通过数据线连接Modular-2，并运行SecureCRT设置为串口模式，选择Modular-2在系统中生成的串口，打开互动窗口后，进行SecureCRT会话设置。
### 串口监视内容
```
RPC Over Http example
0 
MAC Address is 00:80:e1:1f:00:2a 
The Server IP address is '192.168.11.122'
Listening...
----------------------------------------
RPC Request from 192.168.11.150:49288
RPC Request Data: /myled rpc
RPC Return  Data: write read delete
----------------------------------------
RPC Request from 192.168.11.150:49287
RPC Request Data: /myled 
RPC Return  Data: write read delete
----------------------------------------
RPC Request from 192.168.11.150:49307
RPC Request Data: /myled/write 1
RPC Return  Data: 
----------------------------------------
RPC Request from 192.168.11.150:49333
RPC Request Data: /myled/write 1
RPC Return  Data: 
----------------------------------------
RPC Request from 192.168.11.150:49334
RPC Request Data: /myled/read 
RPC Return  Data: 1
 ```
![浏览器访问回显](./screenshots/rpc_over_http_1_chrome_echo_1.png)
![浏览器访问回显](./screenshots/rpc_over_http_1_chrome_echo_2.png)
### 其他事项
更多源码范例可以查看[项目汇总表](https://github.com/modular2/modular-2/blob/master/software/readme.md)
