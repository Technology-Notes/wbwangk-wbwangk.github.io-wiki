插入TZ006到pc，自动安装驱动失败。
安装usb驱动:
CP210xVCPInstaller_x64.exe，安装窗口提示：
Welcome to the CP201X USB to UART Bridge Driver Installer

安装成功后在设备管理器窗口出现新的一级目录：
端口(COM和LTP)
  - Silicon Labs CP210x USB to UART Bridge(COM5)

打开putty，在Session选项中：
connection type选Serial（默认SSH），Serial line输入COM5，Speed输入115200，Saved Sessions输入wifi-sniffer，点save按钮.

(个人手机mate8的mac地址：58:2a:f7:2a:00:5f,hp笔记本mac：08:3e:8e:2f:d2:95)
点击open按钮，出现黑窗口显示：
```
28:01:00:00:40:4A|70:F9:6D:34:BD:51|01|09|11|-75
80:56:F2:7B:47:1D|FF:FF:FF:FF:FF:FF|00|04|12|-88
B4:29:3D:01:11:06|FF:FF:FF:FF:FF:FF|00|04|1|-93
70:F9:6D:EA:95:D6|74:25:8A:8B:1D:31|02|00|1|-81
...
```
以|分隔，第1列是wifi设备的mac地址；第2列是连接到的设备mac地址，一般是无线路由器，FF:FF:FF:FF:FF:FF表示尚未连上热点(探测？)；第3列是帧大类：管理帧00、控制帧01、数据帧02；第4列是帧小类；第5列是信道(1-14)；第6列是RSSI信号强度，最小是-100(表示最远).

## wifi数据帧类型

控制帧 Frame Control 字段 2 个字节：

4bit(Subtype，小类)+2bit(Type，大类)+2bit(Protocol Version，默认为 00)，针对 Frame Control 的各 bit 位的说明如下：

*** 管理帧：type  为 00  时，代表管理帧,负责监督，用来加入或退出无线网络以及处理接入点之间关联的转移事宜。为了限制广播或组播管理帧所造成的副作用，收到管理帧后，必须加以查验。只有广播或者组播帧来自工作站当前所关联的 BSSID 时，它们才会被送至 MAC 管理层。唯一例外是beacon 帧
此时各 Subtype 的值如下：  
    值        |      含义
--------------|-----------------------
    0000      |     Association request（连接要求）
    0001      |     Association response（连接应答）
    0010      |     Reassociation request（重新连接要求）
    0011      |     Reassociation response（重新连接应答）
    0100      |     Probe request（探查要求）
    0101      |     Probe response（探查应答）
    1000      |     Beacon（导引信号）
    1001      |     Announcement  traffic  indication  message (ATIM)    
    1010      |     Disassociation（解除连接）
    1011      |     Authentication（身份验证）
    1100      |     Deauthentication（解除认证）

### 控制帧：type  为 01  时，代表控制帧
   
   subType       |          含义
-----------------|--------------------------
   1010          |         Power Save‐Poll（省电模式－轮询）
   1011          |         RTS（请求发送）
   1100          |         CTS（允许发送）
   1101          |         ACK（应答）
   1110          |         CF‐End（免竞争期间结束）
   1111          |         CF‐End（免竞争期间结束）+CF‐Ack（免竞争期间回应）
   1001          |         块回应


### 数据帧：type  为 10  时，代表数据帧
subType       |          含义
--------------|---------------------------
0000          |          Data（数据）(0x08)
0001          |          Data+CF‐Ack
0010          |          Data+CF‐Poll (0x28)
0011          |          Data+CF‐Ack+CF‐Poll
0100          |          Null data (无数据：未发送数据)(0x48)
0101          |          CF‐Ack (未发送数据)
0110          |          CF‐Poll (未发送数据)
0111          |          Data＋CF‐Ack+CF‐Poll
1000          |          QoS Data【注 c】 (0x88)
1001          |          QoS Data + CF‐Ack
1010          |          QoS Data + CF‐Poll
1011          |          QoS Data + CF‐Ack + CF‐Pol
1100          |          QoS Null (未发送数据)
1101          |          QoS CF‐Ack (未发送数据)
1110          |          QoS CF‐Poll (未发送数据)
1111          |          QoS CF‐Ack+CF‐Poll （未发送数据)

