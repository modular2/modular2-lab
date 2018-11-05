# 实验-4 Modular-2 ESP-12F模块加入Wifi网络
## 实验目的
了解Modular-2的无线接入方式，实验使用Modular-2 ESP-12F模块接入Wifi网络。
## 实验设备
### 硬件：
+ modular-2一台（V1.3）
+ ESP-12F(esp8266) WIFI模块
+ PC电脑一台
### 软件：
+ WINDOWS 7操作系统
+ [SecureCRT终端仿真程序](https://www.vandyke.com/download/securecrt/download.html)
## 实验原理
Modular-2使用Mbed操作系统内置ESP8266无线驱动，通过文件配置接入Wifi网络。
## 实验内容
配置mbed_app.json文件，编译导入程序至modular-2，通过串口监视，查看modular-2访问网站的情况，代码列出，其他操作下文省略。
### 修改mbed_app.json，设置SSID网络名称和密码
```
{
    "config": {
        "wifi-ssid": {
            "help": "WiFi SSID",
            "value": "\"MAXIM\""
        },
        "wifi-password": {
            "help": "WiFi Password",
            "value": "\"PASSWORD\""
        }
    },
    "target_overrides": {
        "*": {
            "platform.stdio-convert-newlines": true,
            "esp8266.provide-default" : true,
            "drivers.uart-serial-rxbuf-size"    : 1024,
            "drivers.uart-serial-txbuf-size"    : 1024,
            "esp8266.rx"                        : "PB_7",
            "esp8266.tx"                        : "PB_6",
            "nsapi.default-wifi-security"       : "WPA_WPA2"
        }
    }
}
```
### 代码
```
#include "mbed.h"
#include "TCPSocket.h"
WiFiInterface *wifi;
DigitalOut WifiEnable(PD_13);
DigitalOut WifiRST(PD_12);
const char *sec2str(nsapi_security_t sec)
{
    switch (sec) {
        case NSAPI_SECURITY_NONE:
            return "None";
        case NSAPI_SECURITY_WEP:
            return "WEP";
        case NSAPI_SECURITY_WPA:
            return "WPA";
        case NSAPI_SECURITY_WPA2:
            return "WPA2";
        case NSAPI_SECURITY_WPA_WPA2:
            return "WPA/WPA2";
        case NSAPI_SECURITY_UNKNOWN:
        default:
            return "Unknown";
    }
}
int scan_demo(WiFiInterface *wifi)
{
    WiFiAccessPoint *ap;

    printf("Scan:\n");

    int count = wifi->scan(NULL,0);

    if (count <= 0) {
        printf("scan() failed with return value: %d\n", count);
        return 0;
    }

    /* Limit number of network arbitrary to 15 */
    count = count < 15 ? count : 15;

    ap = new WiFiAccessPoint[count];
    count = wifi->scan(ap, count);

    if (count <= 0) {
        printf("scan() failed with return value: %d\n", count);
        return 0;
    }

    for (int i = 0; i < count; i++) {
        printf("Network: %s secured: %s BSSID: %hhX:%hhX:%hhX:%hhx:%hhx:%hhx RSSI: %hhd Ch: %hhd\n", ap[i].get_ssid(),
               sec2str(ap[i].get_security()), ap[i].get_bssid()[0], ap[i].get_bssid()[1], ap[i].get_bssid()[2],
               ap[i].get_bssid()[3], ap[i].get_bssid()[4], ap[i].get_bssid()[5], ap[i].get_rssi(), ap[i].get_channel());
    }
    printf("%d networks available.\n", count);

    delete[] ap;
    return count;
}
void http_demo(NetworkInterface *net)
{
    TCPSocket socket;
    nsapi_error_t response;

    printf("Sending HTTP request to www.arm.com...\n");

    // Open a socket on the network interface, and create a TCP connection to www.arm.com
    response = socket.open(net);
    if(0 != response) {
        printf("socket.open() failed: %d\n", response);
        return;
    }

    response = socket.connect("api.ipify.org", 80);
    if(0 != response) {
        printf("Error connecting: %d\n", response);
        socket.close();
        return;
    }

    // Send a simple http request
    char sbuffer[] = "GET / HTTP/1.1\r\nHost: api.ipify.org\r\nConnection: close\r\n\r\n";
    nsapi_size_t size = strlen(sbuffer);

    // Loop until whole request send
    while(size) {
        response = socket.send(sbuffer+response, size);
        if (response < 0) {
            printf("Error sending data: %d\n", response);
            socket.close();
            return;
        }
        size -= response;
        printf("sent %d [%.*s]\n", response, strstr(sbuffer, "\r\n")-sbuffer, sbuffer);
    }

    // Receieve a simple http response and print out the response line
    char rbuffer[64];
    response = socket.recv(rbuffer, sizeof rbuffer);
    if (response < 0) {
        printf("Error receiving data: %d\n", response);
    } else {
        printf("recv %d [%.*s]\n", response, strstr(rbuffer, "\r\n")-rbuffer, rbuffer);
    }

    // Close the socket to return its memory and bring down the network interface
    socket.close();
}
int main()
{
    int count = 0;
    WifiEnable=1;
    WifiRST=0;
    wait(0.1);
    WifiRST=1; 
    printf("WiFi example\n");
#ifdef MBED_MAJOR_VERSION
    printf("Mbed OS version %d.%d.%d\n\n", MBED_MAJOR_VERSION, MBED_MINOR_VERSION, MBED_PATCH_VERSION);
#endif
    wifi = WiFiInterface::get_default_instance();
    if (!wifi) {
        printf("ERROR: No WiFiInterface found.\n");
        return -1;
    }
    count = scan_demo(wifi);
    if (count == 0) {
        printf("No WIFI APNs found - can't continue further.\n");
        return -1;
    }
    printf("\nConnecting to %s...\n", MBED_CONF_APP_WIFI_SSID);
    int ret = wifi->connect(MBED_CONF_APP_WIFI_SSID, MBED_CONF_APP_WIFI_PASSWORD, NSAPI_SECURITY_WPA_WPA2);
    if (ret != 0) {
        printf("\nConnection error: %d\n", ret);
        return -1;
    }
    printf("Success\n\n");
    printf("MAC: %s\n", wifi->get_mac_address());
    printf("IP: %s\n", wifi->get_ip_address());
    printf("Netmask: %s\n", wifi->get_netmask());
    printf("Gateway: %s\n", wifi->get_gateway());
    printf("RSSI: %d\n\n", wifi->get_rssi());
    http_demo(wifi);
    wifi->disconnect();
    printf("\nDone\n");
}
```
### 在线导入,Mbed-CLI编译
```
mbed import https://github.com/modular2/mbed-os-example-wifi
cd mbed-os-example-wifi
mbed compile -t GCC_ARM -m NUCLEO_F429ZI
```
mbed-os-example-wifi\BUILD\NUCLEO_F429ZI\GCC_ARM目录下的mbed-os-example-wifi.bin烧录至modular-2。
#### 串口监控
PC电脑通过数据线连接Modular-2，并运行SecureCRT设置为串口模式，选择Modular-2在系统中生成的串口，打开互动窗口后，进行SecureCRT会话设置。
#### 打印输出信息
```
WiFi example
Mbed OS version 5.10.2

Scan:
Network: MAXIM secured: WPA/WPA2 BSSID: C:4B:54:b2:cb:42 RSSI: -33 Ch: 7

1 networks available.

Connecting to MAXIM...
Success

MAC: dc:4f:22:54:8d:b3
IP: 192.168.11.112
Netmask: 255.255.255.0
Gateway: 192.168.11.1
RSSI: -31

Sending HTTP request to www.arm.com...
sent 58 [GET / HTTP/1.1]
recv 64 [HTTP/1.1 200 OK]

Done
```
### 扩展内容
利用以太网接口及Socket访问更多的网站，处理网站响应，显示更多回应内容。

### 其他事项
更多源码范例可以查看[项目汇总表](https://github.com/modular2/modular-2/blob/master/software/readme.md)
