##wifi嗅探原理浅析
现在的手机都带有wifi功能，它平时总是在不停扫描周边的wifi热点。所谓扫描，就是发送下文中提到的“探查请求”信号。wifi嗅探器（或称wifi探针），不停地接收这类信号，并转换后发送到后台服务器。wifi嗅探是无线路由器的基本功能，只是一般的无线路由器不会把嗅探信号发送到后台服务器。
## 中科爱讯的wifi嗅探产品简介
 - TZ001只支持PC，它通过USB插在PC上。PC安装它自带的驱动程序后，在windows设备管理器上wifi探针显示为一个串口设备(如COM4或COM5)。需要自己写客户端程序接收来自串口的信号，并通过PC的网络发送到服务器端。88元。
 - TZ006支持PC或安卓手机，通过USB口插在PC或手机(需转换头)。TZ006不通过PC的网络传送上行数据，而是通过wifi热点。通过配置程序来设置通过哪个wifi热点来传输上行数据。上传的协议是UDP，提供了java/PHP的服务器端UDP接收数据的源码，几十行代码就可以。78元。
 - TZ007是TZ006的增强，带有双wifi模块。TZ006只带了一个wifi模块，不得不在嗅探模式和上传模式间切换。导致上行数据不是连续的，默认50秒上传一次数据。TZ007和TZ001的上传数据是连续的，只是一个通过PC的网络，一个通过wifi热点。198元。
 - TZ002是TZ001的增强，可以直接插上网线。相当于随身带了一个小PC，并装好了驱动和上传程序。288元。

我们选择了TZ001作为烟草零售POS的配套wifi探针。
## 对TZ001的测试
插入TZ001到pc，自动安装驱动失败。
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

### 管理帧：type为00
负责监督，用来加入或退出无线网络以及处理接入点之间关联的转移事宜。为了限制广播或组播管理帧所造成的副作用，收到管理帧后，必须加以查验。只有广播或者组播帧来自工作站当前所关联的 BSSID 时，它们才会被送至 MAC 管理层。唯一例外是beacon 帧
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

### 控制帧：type为01
   
   subType       |          含义
-----------------|--------------------------
   1010          |         Power Save‐Poll（省电模式－轮询）
   1011          |         RTS（请求发送）
   1100          |         CTS（允许发送）
   1101          |         ACK（应答）
   1110          |         CF‐End（免竞争期间结束）
   1111          |         CF‐End（免竞争期间结束）+CF‐Ack（免竞争期间回应）
   1001          |         块回应


### 数据帧：type为10
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

## TZ006测试
下图是配置TZ006的截图。从图上可以看出，UDP服务器的地址是192.168.3.203。截图上没有参数还有：无线路由器的IP是192.168.3.1，wifi探针的IP是192.168.3.4（无线路由器自动分配的）。
![](https://github.com/wbwangk/wbwangk.github.io/raw/master/images/tz006.png)
UDP服务器就是下文中java代码实现的。通过java Recv执行后输出到屏幕上可以看到TZ006的上传数据：
```
48cd02
F4:8C:50:0A:B9:87|FF:FF:FF:FF:FF:FF|00|04|5|-80
E0:CB:4E:59:B0:5A|E0:CB:4E:59:B0:5C|02|00|6|-84
34:68:95:AD:05:49|FF:FF:FF:FF:FF:FF|00|04|8|-89
68:5D:43:F7:66:88|FF:FF:FF:FF:FF:FF|00|04|8|-86
28:E3:47:D2:0E:38|FF:FF:FF:FF:FF:FF|00|04|10|-88
2C:D0:5A:FC:97:A8|70:F9:6D:34:BD:51|01|09|11|-71
BC:77:37:43:F9:0E|FF:FF:FF:FF:FF:FF|00|04|1|-84
38:29:5A:64:B6:59|FF:FF:FF:FF:FF:FF|00|04|3|-87
38:71:DE:2B:0B:D9|74:E5:0B:8D:FC:19|01|09|6|-79
F0:D5:BF:4A:60:82|FF:FF:FF:FF:FF:FF|00|04|7|-90
 Mar 1, 2017 8:34:59 AM
48cd02
14:2D:27:8E:84:ED|FF:FF:FF:FF:FF:FF|00|04|9|-85
 Mar 1, 2017 8:34:59 AM
48cd02
30:10:B3:94:B3:40|FF:FF:FF:FF:FF:FF|00|04|2|-83
68:3E:34:9E:20:A1|FF:FF:FF:FF:FF:FF|00|04|7|-78
08:3E:8E:2F:D2:95|B8:08:D7:71:CB:90|02|00|6|-49
B8:08:D7:71:CB:88|B8:08:D7:71:CB:90|02|08|5|-46
 Mar 1, 2017 8:35:56 AM
```
48cd02是wifi探针的设备id。“ Mar 1, 2017 8:34:59 AM”是上传时间。注意Mar前面有个空格，估计空格是个标志，用来表明这一行是时间。  
在上面的范例数据中，上传间隔是50秒。如果上传的数据超过10个mac地址，会拆分为10个mac一组。

#### 在linux下发送udp请求
如果往本地UDP端口發送數據，那麼可以使用以下命令：
```
echo “hello” > /dev/udp/10.10.11.86/9002
```
意思是往本地192.168.1.81的5060端口發送數據包hello。

如果往遠程UDP端口發送數據，那麼可以使用以下命令：
```
$ apt install socat
$ echo “hello” | socat - udp4-datagram:10.10.11.86:9002
```
## java实现的udp server
默认监听端口9002，如果要监听其他端口，如8888，可以这样执行：
```
$ java Recv 8888
```
java源码：
```
import java.io.IOException;
import java.net.DatagramPacket;
import java.net.DatagramSocket;
import java.net.InetAddress;
import java.net.SocketException;
import java.util.ArrayList;
import java.util.Date;
import java.util.List;
import java.util.concurrent.ExecutorService;
import java.util.concurrent.Executors;


public class Recv {

        public static final int DEFAULT_PORT = 9002;
        public static final int MAX_MSG_LEN = 1600;

        public static ExecutorService dataHandlePool = Executors
                        .newFixedThreadPool(64);


        public static void start(int port) {
                try {
                        DatagramSocket udp = new DatagramSocket(port);
                        DatagramPacket dPacket;
                        byte[] echo = new byte[1];
                        echo[0] = (byte)1;
                        while (true) {
                                dPacket = new DatagramPacket(new byte[MAX_MSG_LEN], MAX_MSG_LEN);
                                udp.receive(dPacket);
                                String result = new String(dPacket.getData(),0,dPacket.getLength());
                                System.out.println(result + " " + new Date(System.currentTimeMillis()).toLocaleString());
                                //返回一个字节给探针设备
                                InetAddress addr = dPacket.getAddress();
                                dPacket = new DatagramPacket(echo, echo.length);
                                dPacket.setAddress(addr);
                                udp.send(dPacket);
                                }

                } catch (SocketException e) {
                        e.printStackTrace();
                } catch (IOException e) {
                        e.printStackTrace();
                } catch (Exception e) {
                        e.printStackTrace();
                }
        }

        public static void main(String[] args) {
                if (args != null && args.length == 1) {
                        start(Integer.parseInt(args[0]));
                }else {
                        start(DEFAULT_PORT);
                }
        }
}                                        
```
Recv.class会在屏幕上打印出收到的UDP消息：
```
$ java Recv
“hello”
 Feb 28, 2017 5:50:03 AM
“hello”
 Feb 28, 2017 5:50:49 AM
```
上述消息是使用linux命令发出的。

## 客流统计需求分析
wifi探针TZ001是中科爱讯的产品，中科爱讯也提供客流统计的云服务，叫“**每时每客**”。每时每客九项核心数据：
####1.总客流量
到达店铺或区域的整体客流量。应是以日/周/月为单位统计，应为“人次”数。如果一个人一天进店两次，算两个人？

####2.驻店时长
进入店铺顾客在店内停留的时长。 也是按“人次”。如果一个人进来两次（时间间隔超过阈值），两次的时长不应叠加。

####3.进店跳出量
进入店铺后离开店铺的顾客流量。

####4.多店铺管理
多个店铺数据实时统计，数据统一管理。 多个店铺的数据合并，视为一个大店铺？

####5.新老顾客量
在某一时段内第一次/多次以上进入店铺的顾客流量

####6.进店客流量
进入店铺的客流量。包括在店的和离开的？

####7.在店客流量
当前在店铺的顾客流量

####8.老顾客增长量
在某一时段内二次进入店铺的顾客增长数量

####9.老顾客流失量
在某一时段内多次进入店铺的顾客流失数量

## 客流统计的实现
wifi探针给的样例代码使用UDP协议上传数据到云端。UDP协议比TCP协议轻量，但不保证消息能发送成功。

####负载均衡：按IP进行散列
nginx支持UDP的负载均衡，可选择默认的按IP的负载均衡策略（来自同一IP的数据会被反向代理到同一个应用）。这是最常见负载均衡策略，可支持会话粘连。nginx实现的UDP负载均衡配置：
```
stream {
    server {
        listen 9003 udp;
        proxy_pass wifi_sniffer_backend;
        proxy_responses 0;
    }
    upstream wifi_sniffer_backend {
        server 127.0.0.1:9002;
        ...  (其他的UDP应用)
    }
}
```
在本地启动UDP测试应用，并发送“hello”到9003端口，：
```
$ java Recv 9002 &
$ echo “hello” | socat - udp4-datagram:127.0.0.1:9003
“hello”
 Mar 1, 2017 8:44:32 AM
```
发送到9003端口的数据被NGINX反向代理到了9002端口，并被Recv类输出到了屏幕。（如果不在命令的最后加上```&```符号，需要另外再开一个终端。）
####java缓存：顾客离店前，他的数据可以一直放在内存里
判断他是否离店应使用另外的线程处理。