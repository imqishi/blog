---
title: SDN基础知识学习-环境配置
date: 2017-07-19 23:18:48
tags: SDN
---


### 安装配置ODL

1. 安装Java
2. 官网下载对应的包，并解压
3. 进入解压后的文件夹
4. `./bin/karaf`
5. 安装必要的feature：`feature:install odl-l2switch-switch odl-restconf odl-mdsal-all odl-openflowplugin-all odl-dlux-all odl-adsal-northbound` 新版本里有可能没有`odl-adsal-northbound`这个feature，装了`odl-mdsal-all`就可以了
6. 打开浏览器，输入`127.0.0.1:8181/index.html`登录Web控制台以便后续查看拓扑等。如果页面404，可以尝试`127.0.0.1:8181/index.html#/topology`或者`127.0.0.1:8181/index.html#/login`

### 安装配置MN

1. 安装mininet，可直接使用`apt-get install mininet`
2. 在mininet目录下的example文件夹下使用miniedit编辑拓扑：
   1. 控制器方式设置为Remote Controller
   2. 使用switch（not Legacy switch）
   3. 设置主机ip
   4. 连线后Run，看是否有报错。Run前要确定odl控制器是否已开启。若无报错，使用`pingall`测试连通性。若连通，可以继续进行。在相应设备上右键可以打开相应Terminal。

##### 如果miniedit成功Run但是不出来`mininet>`命令提示符，可以先将拓扑导出为python文件：File-->Export Level2 Script，然后手动执行`python test.py` 进入`mininet>`界面，使用`xterm h1`打开相应设备的Terminal

##### 如果mininet总是报奇怪的错误，可以尝试在Terminal中使用`mn -c`清空原有配置，再次运行。

### 查看配置、下流表

1. 查看拓扑：可以从Web端查看，或者使用`ovs-vsctl show`命令
2. 查看流表：`ovs-ofctl dump-flows s1`
3. 下流表

利用Postman、*PUT方式*、*Authorization使用Basic，用户密码均为admin*。

Headers设置如下：

```http
Accept:application/xml
Content-Type:applcation/xml
Postman还会自动生成一个Authorization的域，我们可以不用管它
```

Body设置类型为*raw* 类型 *XML(application/xml)*，下述为**阻断 ** *源和目的均为10.0.0.0/24(ipv4-source, ipv4-destination)* *IPv4(ethernet-type)* *TCP(ip-protocol)* *13911目的端口(tcp-destination-port)*的流

url为：http://localhost:8181/restconf/config/opendaylight-inventory:nodes/node/openflow:1/table/0/flow/1

注意openflow:后面跟的数字应该是在Web拓扑中看到的openflow:后的数字，初步配置table就是0对应xml文件中的**table_id**，flow后的1对应xml文件中的**id**，xml文件中**flow-name**随便起。像**strict**，**cookie-mask**，**installHw**，**barrier**没有特殊要求都可以先不配。还需要注意的一点是，**priority数值越大优先级越大**。其实匹配规则就是按照ovs-ofctl dump-flows s1从上往下的顺序匹配的。

```xml
<?xml version="1.0" encoding="UTF-8" standalone="no"?>
<flow xmlns="urn:opendaylight:flow:inventory">
    <strict>false</strict>
    <match>
        <ethernet-match>
            <ethernet-type>
                <type>2048</type>
            </ethernet-type>
        </ethernet-match>
        <ipv4-source>10.0.0.1/24</ipv4-source>
        <ipv4-destination>10.0.0.3/24</ipv4-destination>
         <ip-match>
            <ip-protocol>6</ip-protocol>
        </ip-match>
        <tcp-destination-port>13911</tcp-destination-port>
    </match>
    <table_id>0</table_id>
    <id>1</id>
    <flow-name>flow1</flow-name>
    <priority>1</priority>
    <instructions>
        <instruction>
            <order>0</order>
            <apply-actions>
                <action>
                    <order>0</order>
                    <drop-action/>
                </action>
            </apply-actions>
        </instruction>
    </instructions>
    <cookie>3</cookie>
    <cookie_mask>255</cookie_mask>
    <installHw>false</installHw>
    <hard-timeout>1200</hard-timeout>
    <barrier>false</barrier>
</flow>
```

同理，我们可以将**ip-protoco**l置为**17**，**tcp-destination-port**改为**udp-destination-port**去阻断udp端口

### 测试工具 iperf简介

Iperf 是一个网络性能测试工具。Iperf可以测试最大TCP和UDP带宽性能。Iperf具有多种参数和UDP特性，可以根据需要调整。Iperf可以报告带宽，延迟抖动和数据包丢失。

一般测试两台设备之间的连通性。在一台设备上使用*server模式*，另一台使用*client模式*。

常用参数：

-s 以server方式运行

-c 10.0.0.1 以client方式，作为10.0.0.1server的client运行

-p 指定端口

-u udp方式（默认是tcp方式）

### 附录

#### 以太网类型码

| **Ethernet** | **Exp. Ethernet** | **Description** |       |                           |
| ------------ | ----------------- | --------------- | ----- | ------------------------- |
| decimal      | Hex               | decimal         | octal |                           |
| 0000         | 0000-05DC         |                 |       | IEEE802.3LengthField      |
| 0257         | 0101-01FF         | -               | -     | Experimental              |
| 0512         | 0200              | 512             | 1000  | XEROX PUP (see 0A00)      |
| 0513         | 0201              | -               | -     | PUP Addr Trans (see 0A01) |
|              | 0400              | -               | -     | Nixdorf                   |
| 1536         | 0600              | 1536            | 3000  | XEROX NS IDP              |
|              | 0660              | -               | -     | DLOG                      |
|              | 0661              | -               | -     | DLOG                      |
| 2048         | 0800              | 513             | 1001  | Internet IP (IPv4)        |
| 2049         | 0801              | -               | -     | X.75 Internet             |
| 2050         | 0802              | -               | -     | NBS Internet              |
| 2051         | 0803              | -               | -     | ECMA Internet             |
| 2052         | 0804              | -               | -     | Chaosnet                  |
| 2053         | 0805              | -               | -     | X.25 Level 3              |
| 2054         | 0806              | -               | -     | ARP                       |
| 2055         | 0807              | -               | -     | XNS Compatability         |
| 2056         | 0808              | -               | -     | Frame Relay ARP           |
| 2076         | 081C              | -               | -     | Symbolics Private         |
| 2184         | 0888-088A         | -               | -     | Xyplex                    |
| 2304         | 0900              | -               | -     | Ungermann-Bass net debugr |
| 2560         | 0A00              | -               | -     | Xerox IEEE802.3 PUP       |
| 2561         | 0A01              | -               | -     | PUP Addr Trans            |
| 2989         | 0BAD              | -               | -     | Banyan VINES              |
| 2990         | 0BAE              | -               | -     | VINES Loopback            |
| 2991         | 0BAF              | -               | -     | VINES Echo                |
| 4096         | 1000              | -               | -     | Berkeley Trailer nego     |
| 4097         | 1001-100F         | -               | -     | Berkeley Trailer encap/IP |
| 5632         | 1600              | -               | -     | Valid Systems             |
| 16962        | 4242              | -               | -     | PCS Basic Block Protocol  |
| 21000        | 5208              | -               | -     | BBN Simnet                |
| 24576        | 6000              | -               | -     | DEC Unassigned (Exp.)     |
| 24577        | 6001              | -               | -     | DEC MOP Dump/Load         |
| 24578        | 6002              | -               | -     | DEC MOP Remote Console    |
| 24579        | 6003              | -               | -     | DEC DECNET Phase IV Route |
| 24580        | 6004              | -               | -     | DEC LAT                   |
| 24581        | 6005              | -               | -     | DEC Diagnostic Protocol   |
| 24582        | 6006              | -               | -     | DEC Customer Protocol     |
| 24583        | 6007              | -               | -     | DEC LAVC, SCA             |
| 24584        | 6008-6009         | -               | -     | DEC Unassigned            |
| 24586        | 6010-6014         | -               | -     | 3Com Corporation          |
| 25944        | 6558              | -               | -     | Trans Ether Bridging      |
| 25945        | 6559              | -               | -     | Raw Frame Relay           |
| 28672        | 7000              | -               | -     | Ungermann-Bass download   |
| 28674        | 7002              | -               | -     | Ungermann-Bass dia/loop   |
| 28704        | 7020-7029         | -               | -     | LRT                       |
| 28720        | 7030              | -               | -     | Proteon                   |
| 28724        | 7034              | -               | -     | Cabletron                 |
| 32771        | 8003              | -               | -     | Cronus VLN                |
| 32772        | 8004              | -               | -     | Cronus Direct             |
| 32773        | 8005              | -               | -     | HP Probe                  |
| 32774        | 8006              | -               | -     | Nestar                    |
| 32776        | 8008              | -               | -     | AT&T                      |
| 32784        | 8010              | -               | -     | Excelan                   |
| 32787        | 8013              | -               | -     | SGI diagnostics           |
| 32788        | 8014              | -               | -     | SGI network games         |
| 32789        | 8015              | -               | -     | SGI reserved              |
| 32790        | 8016              | -               | -     | SGI bounce server         |
| 32793        | 8019              | -               | -     | Apollo Domain             |
| 32815        | 802E              | -               | -     | Tymshare                  |
| 32816        | 802F              | -               | -     | Tigan, Inc.               |
| 32821        | 8035              | -               | -     | Reverse ARP               |
| 32822        | 8036              | -               | -     | Aeonic Systems            |
| 32824        | 8038              | -               | -     | DEC LANBridge             |
| 32825        | 8039-803C         | -               | -     | DEC Unassigned            |
| 32829        | 803D              | -               | -     | DEC Ethernet Encryption   |
| 32830        | 803E              | -               | -     | DEC Unassigned            |
| 32831        | 803F              | -               | -     | DEC LAN Traffic Monitor   |
| 32832        | 8040-8042         | -               | -     | DEC Unassigned            |
| 32836        | 8044              | -               | -     | Planning Research Corp.   |
| 32838        | 8046              | -               | -     | AT&T                      |
| 32839        | 8047              | -               | -     | AT&T                      |
| 32841        | 8049              | -               | -     | ExperData                 |
| 32859        | 805B              | -               | -     | Stanford V Kernel exp.    |
| 32860        | 805C              | -               | -     | Stanford V Kernel prod.   |
| 32861        | 805D              | -               | -     | Evans & Sutherland        |
| 32864        | 8060              | -               | -     | Little Machines           |
| 32866        | 8062              | -               | -     | Counterpoint Computers    |
| 32869        | 8065              | -               | -     | Univ. of Mass. @A mherst  |
| 32870        | 8066              | -               | -     | Univ. of Mass. @ Amherst  |
| 32871        | 8067              | -               | -     | Veeco Integrated Auto.    |
| 32872        | 8068              | -               | -     | General Dynamics          |
| 32873        | 8069              | -               | -     | AT&T                      |
| 32874        | 806A              | -               | -     | Autophon                  |
| 32876        | 806C              | -               | -     | ComDesign                 |
| 32877        | 806D              | -               | -     | Computgraphic Corp.       |
| 32878        | 806E-8077         | -               | -     | Landmark Graphics Corp.   |
| 32890        | 807A              | -               | -     | Matra                     |
| 32891        | 807B              | -               | -     | Dansk Data Elektronik     |
| 32892        | 807C              | -               | -     | Merit Internodal          |
| 32893        | 807D-807F         | -               | -     | Vitalink Communications   |
| 32896        | 8080              | -               | -     | Vitalink TransLAN III     |
| 32897        | 8081-8083         | -               | -     | Counterpoint Computers    |
| 32923        | 809B              | -               | -     | Appletalk                 |
| 32924        | 809C-809E         | -               | -     | Datability                |
| 32927        | 809F              | -               | -     | Spider Systems Ltd.       |
| 32931        | 80A3              | -               | -     | Nixdorf Computers         |
| 32932        | 80A4-80B3         | -               | -     | Siemens Gammasonics Inc.  |
| 32960        | 80C0-80C3         | -               | -     | DCA Data Exchange Cluster |
| 32964        | 80C4              | -               | -     | Banyan Systems            |
| 32965        | 80C5              | -               | -     | Banyan Systems            |
| 32966        | 80C6              | -               | -     | Pacer Software            |
| 32967        | 80C7              | -               | -     | Applitek Corporation      |
| 32968        | 80C8-80CC         | -               | -     | Intergraph Corporation    |
| 32973        | 80CD-80CE         | -               | -     | Harris Corporation        |
| 32975        | 80CF-80D2         | -               | -     | Taylor Instrument         |
| 32979        | 80D3-80D4         | -               | -     | Rosemount Corporation     |
| 32981        | 80D5              | -               | -     | IBM SNA Service on Ether  |
| 32989        | 80DD              | -               | -     | Varian Associates         |
| 32990        | 80DE-80DF         | -               | -     | Integrated Solutions TRFS |
| 32992        | 80E0-80E3         | -               | -     | Allen-Bradley             |
| 32996        | 80E4-80F0         | -               | -     | Datability                |
| 33010        | 80F2              | -               | -     | Retix                     |
| 33011        | 80F3              | -               | -     | AppleTalk AARP (Kinetics) |
| 33012        | 80F4-80F5         | -               | -     | Kinetics                  |
| 33015        | 80F7              | -               | -     | Apollo Computer           |
| 33023        | 80FF-8103         | -               | -     | Wellfleet Communications  |
| 33031        | 8107-8109         | -               | -     | Symbolics Private         |
| 33072        | 8130              | -               | -     | Hayes Microcomputers      |
| 33073        | 8131              | -               | -     | VG Laboratory Systems     |
| 33074        | 8132-8136         | -               | -     | Bridge Communications     |
| 33079        | 8137-8138         | -               | -     | Novell, Inc.              |
| 33081        | 8139-813D         | -               | -     | KTI                       |
|              | 8148              | -               | -     | Logicraft                 |
|              | 8149              | -               | -     | Network Computing Devices |
|              | 814A              | -               | -     | Alpha Micro               |
| 33100        | 814C              | -               | -     | - SNMP                    |
|              | 814D              | -               | -     | BIIN                      |
|              | 814E              | -               | -     | BIIN                      |
|              | 814F              | -               | -     | Technically Elite Concept |
|              | 8150              | -               | -     | Rational Corp             |
|              | 8151-8153         | -               | -     | Qualcomm                  |
|              | 815C-815E         | -               | -     | Computer Protocol Pty Ltd |
|              | 8164-8166         | -               | -     | Charles River Data System |
|              | 817D              | -               | -     | XTP                       |
|              | 817E              | -               | -     | SGI/Time Warner prop.     |
|              | 8180              | -               | -     | HIPPI-FP encapsulation    |
|              | 8181              | -               | -     | STP, HIPPI-ST             |
|              | 8182              | -               | -     | Reserved for HIPPI-6400   |
|              | 8183              | -               | -     | Reserved for HIPPI-6400   |
|              | 8184-818C         | -               | -     | Silicon Graphics prop.    |
|              | 818D              | -               | -     | Motorola Computer         |
|              | 819A-81A3         | -               | -     | Qualcomm                  |
|              | 81A4              | -               | -     | ARAI Bunkichi             |
|              | 81A5-81AE         | -               | -     | RAD Network Devices       |
|              | 81B7-81B9         | -               | -     | Xyplex                    |
|              | 81CC-81D5         | -               | -     | Apricot Computers         |
|              | 81D6-81DD         | -               | -     | Artisoft                  |
|              | 81E6-81EF         | -               | -     | Polygon                   |
|              | 81F0-81F2         | -               | -     | Comsat Labs               |
|              | 81F3-81F5         | -               | -     | SAIC                      |
|              | 81F6-81F8         | -               | -     | VG Analytical             |
|              | 8203-8205         | -               | -     | Quantum Software          |
|              | 8221-8222         | -               | -     | Ascom Banking Systems     |
|              | 823E-8240         | -               | -     | Advanced Encryption Syste |
|              | 827F-8282         | -               | -     | Athena Programming        |
|              | 8263-826A         | -               | -     | Charles River Data System |
|              | 829A-829B         | -               | -     | Inst Ind Info Tech        |
|              | 829C-82AB         | -               | -     | Taurus Controls           |
|              | 82AC-8693         | -               | -     | Walker Richer & Quinn     |
|              | 8694-869D         | -               | -     | Idea Courier              |
|              | 869E-86A1         | -               | -     | Computer Network Tech     |
|              | 86A3-86AC         | -               | -     | Gateway Communications    |
|              | 86DB              | -               | -     | SECTRA                    |
|              | 86DE              | -               | -     | Delta Controls            |
|              | 86DD              | -               | -     | IPv6                      |
| 34543        | 86DF              | -               | -     | ATOMIC                    |
|              | 86E0-86EF         | -               | -     | Landis & Gyr Powers       |
|              | 8700-8710         | -               | -     | Motorola                  |
| 34667        | 876B              | -               | -     | TCP/IP Compression        |
| 34668        | 876C              | -               | -     | IP Autonomous Systems     |
| 34669        | 876D              | -               | -     | Secure Data               |
|              | 880B              | -               | -     | PPP                       |
|              | 8847              | -               | -     | MPLS Unicast              |
|              | 8848              | -               | -     | MPLS Multicast            |
|              | 8A96-8A97         | -               | -     | Invisible Software        |
| 36864        | 9000              | -               | -     | Loopback                  |
| 36865        | 9001              | -               | -     | 3Com(Bridge) XNS Sys Mgmt |
| 36866        | 9002              | -               | -     | 3Com(Bridge) TCP-IP Sys   |
| 36867        | 9003              | -               | -     | 3Com(Bridge) loop detect  |
| 65280        | FF00              | -               | -     | BBN VITAL-LanBridge cache |
|              | FF00-FF0F         | -               | -     | ISC Bunker Ramo           |
| 65535        | FFFF              | -               | -     | Reserved                  |



#### IP Protocol number与各协议对应表

| 十进制     | 十六进制      | 关键字                                      | 协议                                       | 引用                                       |
| ------- | --------- | ---------------------------------------- | ---------------------------------------- | ---------------------------------------- |
| 0       | 0x00      | HOPOPT                                   | IPv6逐跳选项                                 | [RFC 2460](https://tools.ietf.org/html/rfc2460) |
| 1       | 0x01      | ICMP                                     | [互联网控制消息协议](https://zh.wikipedia.org/wiki/%E4%BA%92%E8%81%94%E7%BD%91%E6%8E%A7%E5%88%B6%E6%B6%88%E6%81%AF%E5%8D%8F%E8%AE%AE) | [RFC 792](https://tools.ietf.org/html/rfc792) |
| 2       | 0x02      | IGMP                                     | [因特网组管理协议](https://zh.wikipedia.org/wiki/%E5%9B%A0%E7%89%B9%E7%BD%91%E7%BB%84%E7%AE%A1%E7%90%86%E5%8D%8F%E8%AE%AE) | [RFC 1112](https://tools.ietf.org/html/rfc1112) |
| 3       | 0x03      | GGP                                      | [网关对网关协议](https://zh.wikipedia.org/w/index.php?title=%E7%BD%91%E5%85%B3%E5%AF%B9%E7%BD%91%E5%85%B3%E5%8D%8F%E8%AE%AE&action=edit&redlink=1) | [RFC 823](https://tools.ietf.org/html/rfc823) |
| 4       | 0x04      | IPv4                                     | [IPv4](https://zh.wikipedia.org/wiki/IPv4) (封装) | [RFC 791](https://tools.ietf.org/html/rfc791) |
| 5       | 0x05      | ST                                       | [因特网流协议](https://zh.wikipedia.org/wiki/%E5%9B%A0%E7%89%B9%E7%BD%91%E6%B5%81%E5%8D%8F%E8%AE%AE) | [RFC 1190](https://tools.ietf.org/html/rfc1190), [RFC 1819](https://tools.ietf.org/html/rfc1819) |
| 6       | 0x06      | TCP                                      | [传输控制协议](https://zh.wikipedia.org/wiki/%E4%BC%A0%E8%BE%93%E6%8E%A7%E5%88%B6%E5%8D%8F%E8%AE%AE) | [RFC 793](https://tools.ietf.org/html/rfc793) |
| 7       | 0x07      | CBT                                      | [有核树组播路由协议](https://zh.wikipedia.org/w/index.php?title=%E6%9C%89%E6%A0%B8%E6%A0%91%E7%BB%84%E6%92%AD%E8%B7%AF%E7%94%B1%E5%8D%8F%E8%AE%AE&action=edit&redlink=1) | [RFC 2189](https://tools.ietf.org/html/rfc2189) |
| 8       | 0x08      | EGP                                      | [外部网关协议](https://zh.wikipedia.org/wiki/%E5%A4%96%E9%83%A8%E7%BD%91%E5%85%B3%E5%8D%8F%E8%AE%AE) | [RFC 888](https://tools.ietf.org/html/rfc888) |
| 9       | 0x09      | IGP                                      | [内部网关协议](https://zh.wikipedia.org/wiki/%E5%86%85%E9%83%A8%E7%BD%91%E5%85%B3%E5%8D%8F%E8%AE%AE)（任意私有内部网关（用于思科的IGRP）） |                                          |
| 10      | 0x0A      | BBN-RCC-MON                              | BBN RCC 监视                               |                                          |
| 11      | 0x0B      | NVP-II                                   | [网络语音协议](https://zh.wikipedia.org/wiki/%E7%BD%91%E7%BB%9C%E8%AF%AD%E9%9F%B3%E5%8D%8F%E8%AE%AE) | [RFC 741](https://tools.ietf.org/html/rfc741) |
| 12      | 0x0C      | PUP                                      | Xerox [PUP](https://zh.wikipedia.org/w/index.php?title=PUP&action=edit&redlink=1) |                                          |
| 13      | 0x0D      | ARGUS                                    | ARGUS                                    |                                          |
| 14      | 0x0E      | EMCON                                    | EMCON                                    |                                          |
| 15      | 0x0F      | XNET                                     | Cross Net Debugger                       | IEN 158                                  |
| 16      | 0x10      | CHAOS                                    | Chaos                                    |                                          |
| 17      | 0x11      | UDP                                      | [用户数据报协议](https://zh.wikipedia.org/wiki/%E7%94%A8%E6%88%B7%E6%95%B0%E6%8D%AE%E6%8A%A5%E5%8D%8F%E8%AE%AE) | [RFC 768](https://tools.ietf.org/html/rfc768) |
| 18      | 0x12      | MUX                                      | [Multiplexing](https://zh.wikipedia.org/w/index.php?title=Multiplexing&action=edit&redlink=1) | IEN 90                                   |
| 19      | 0x13      | DCN-MEAS                                 | DCN Measurement Subsystems               |                                          |
| 20      | 0x14      | HMP                                      | [Host Monitoring Protocol](https://zh.wikipedia.org/w/index.php?title=Host_Monitoring_Protocol&action=edit&redlink=1) | [RFC 869](https://tools.ietf.org/html/rfc869) |
| 21      | 0x15      | PRM                                      | Packet Radio Measurement                 |                                          |
| 22      | 0x16      | XNS-IDP                                  | XEROX NS IDP                             |                                          |
| 23      | 0x17      | TRUNK-1                                  | Trunk-1                                  |                                          |
| 24      | 0x18      | TRUNK-2                                  | Trunk-2                                  |                                          |
| 25      | 0x19      | LEAF-1                                   | Leaf-1                                   |                                          |
| 26      | 0x1A      | LEAF-2                                   | Leaf-2                                   |                                          |
| 27      | 0x1B      | RDP                                      | [Reliable Datagram Protocol](https://zh.wikipedia.org/w/index.php?title=Reliable_Datagram_Protocol&action=edit&redlink=1) | [RFC 908](https://tools.ietf.org/html/rfc908) |
| 28      | 0x1C      | IRTP                                     | [Internet Reliable Transaction Protocol](https://zh.wikipedia.org/w/index.php?title=Internet_Reliable_Transaction_Protocol&action=edit&redlink=1) | [RFC 938](https://tools.ietf.org/html/rfc938) |
| 29      | 0x1D      | ISO-TP4                                  | ISO Transport Protocol Class 4           | [RFC 905](https://tools.ietf.org/html/rfc905) |
| 30      | 0x1E      | NETBLT                                   | [Bulk Data Transfer Protocol](https://zh.wikipedia.org/w/index.php?title=Bulk_Data_Transfer_Protocol&action=edit&redlink=1) | [RFC 998](https://tools.ietf.org/html/rfc998) |
| 31      | 0x1F      | MFE-NSP                                  | [MFE Network Services Protocol](https://zh.wikipedia.org/w/index.php?title=MFE_Network_Services_Protocol&action=edit&redlink=1) |                                          |
| 32      | 0x20      | MERIT-INP                                | [MERIT Internodal Protocol](https://zh.wikipedia.org/w/index.php?title=MERIT_Internodal_Protocol&action=edit&redlink=1) |                                          |
| 33      | 0x21      | DCCP                                     | [Datagram Congestion Control Protocol](https://zh.wikipedia.org/wiki/Datagram_Congestion_Control_Protocol) | [RFC 4340](https://tools.ietf.org/html/rfc4340) |
| 34      | 0x22      | 3PC                                      | [Third Party Connect Protocol](https://zh.wikipedia.org/w/index.php?title=Third_Party_Connect_Protocol&action=edit&redlink=1) |                                          |
| 35      | 0x23      | IDPR                                     | [Inter-Domain Policy Routing Protocol](https://zh.wikipedia.org/w/index.php?title=Inter-Domain_Policy_Routing_Protocol&action=edit&redlink=1) | [RFC 1479](https://tools.ietf.org/html/rfc1479) |
| 36      | 0x24      | XTP                                      | [Xpress Transport Protocol](https://zh.wikipedia.org/w/index.php?title=Xpress_Transport_Protocol&action=edit&redlink=1) |                                          |
| 37      | 0x25      | DDP                                      | [Datagram Delivery Protocol](https://zh.wikipedia.org/w/index.php?title=Datagram_Delivery_Protocol&action=edit&redlink=1) |                                          |
| 38      | 0x26      | IDPR-CMTP                                | [IDPR Control Message Transport Protocol](https://zh.wikipedia.org/w/index.php?title=IDPR_Control_Message_Transport_Protocol&action=edit&redlink=1) |                                          |
| 39      | 0x27      | TP++                                     | [TP++ Transport Protocol](https://zh.wikipedia.org/w/index.php?title=TP%2B%2B_Transport_Protocol&action=edit&redlink=1) |                                          |
| 40      | 0x28      | IL                                       | [IL Transport Protocol](https://zh.wikipedia.org/w/index.php?title=IL_(network_protocol)&action=edit&redlink=1) |                                          |
| 41      | 0x29      | IPv6                                     | IPv6 Encapsulation                       | [RFC 2473](https://tools.ietf.org/html/rfc2473) |
| 42      | 0x2A      | SDRP                                     | [Source Demand Routing Protocol](https://zh.wikipedia.org/w/index.php?title=Source_Demand_Routing_Protocol&action=edit&redlink=1) | [RFC 1940](https://tools.ietf.org/html/rfc1940) |
| 43      | 0x2B      | IPv6-Route                               | Routing Header for [IPv6](https://zh.wikipedia.org/wiki/IPv6) | [RFC 2460](https://tools.ietf.org/html/rfc2460) |
| 44      | 0x2C      | IPv6-Frag                                | Fragment Header for [IPv6](https://zh.wikipedia.org/wiki/IPv6) | [RFC 2460](https://tools.ietf.org/html/rfc2460) |
| 45      | 0x2D      | IDRP                                     | [Inter-Domain Routing Protocol](https://zh.wikipedia.org/w/index.php?title=Inter-Domain_Routing_Protocol&action=edit&redlink=1) |                                          |
| 46      | 0x2E      | RSVP                                     | [Resource Reservation Protocol](https://zh.wikipedia.org/wiki/Resource_Reservation_Protocol) | [RFC 2205](https://tools.ietf.org/html/rfc2205) |
| 47      | 0x2F      | GRE                                      | [Generic Routing Encapsulation](https://zh.wikipedia.org/w/index.php?title=Generic_Routing_Encapsulation&action=edit&redlink=1) | [RFC 2784](https://tools.ietf.org/html/rfc2784), [RFC 2890](https://tools.ietf.org/html/rfc2890) |
| 48      | 0x30      | MHRP                                     | [Mobile Host Routing Protocol](https://zh.wikipedia.org/w/index.php?title=Mobile_Host_Routing_Protocol&action=edit&redlink=1) |                                          |
| 49      | 0x31      | BNA                                      | BNA                                      |                                          |
| 50      | 0x32      | ESP                                      | [Encapsulating Security Payload](https://zh.wikipedia.org/w/index.php?title=Encapsulating_Security_Payload&action=edit&redlink=1) | [RFC 4303](https://tools.ietf.org/html/rfc4303) |
| 51      | 0x33      | AH                                       | [Authentication Header](https://zh.wikipedia.org/w/index.php?title=Authentication_Header&action=edit&redlink=1) | [RFC 4302](https://tools.ietf.org/html/rfc4302) |
| 52      | 0x34      | I-NLSP                                   | [Integrated Net Layer Security Protocol](https://zh.wikipedia.org/w/index.php?title=Integrated_Net_Layer_Security_Protocol&action=edit&redlink=1) | TUBA                                     |
| 53      | 0x35      | SWIPE                                    | [SwIPe](https://zh.wikipedia.org/w/index.php?title=SwIPe_(protocol)&action=edit&redlink=1) | IP with Encryption                       |
| 54      | 0x36      | NARP                                     | [NBMA Address Resolution Protocol](https://zh.wikipedia.org/w/index.php?title=NBMA_Address_Resolution_Protocol&action=edit&redlink=1) | [RFC 1735](https://tools.ietf.org/html/rfc1735) |
| 55      | 0x37      | MOBILE                                   | [IP Mobility](https://zh.wikipedia.org/wiki/Mobile_IP) (Min Encap) | [RFC 2004](https://tools.ietf.org/html/rfc2004) |
| 56      | 0x38      | TLSP                                     | [Transport Layer Security Protocol](https://zh.wikipedia.org/w/index.php?title=Transport_Layer_Security_Protocol&action=edit&redlink=1) (using Kryptonet key management) |                                          |
| 57      | 0x39      | SKIP                                     | [Simple Key-Management for Internet Protocol](https://zh.wikipedia.org/w/index.php?title=Simple_Key-Management_for_Internet_Protocol&action=edit&redlink=1) | [RFC 2356](https://tools.ietf.org/html/rfc2356) |
| 58      | 0x3A      | IPv6-ICMP                                | [ICMP for IPv6](https://zh.wikipedia.org/wiki/ICMPv6) | [RFC 4443](https://tools.ietf.org/html/rfc4443), [RFC 4884](https://tools.ietf.org/html/rfc4884) |
| 59      | 0x3B      | IPv6-NoNxt                               | No Next Header for [IPv6](https://zh.wikipedia.org/wiki/IPv6) | [RFC 2460](https://tools.ietf.org/html/rfc2460) |
| 60      | 0x3C      | IPv6-Opts                                | Destination Options for [IPv6](https://zh.wikipedia.org/wiki/IPv6) | [RFC 2460](https://tools.ietf.org/html/rfc2460) |
| 61      | 0x3D      |                                          | Any host internal protocol               |                                          |
| 62      | 0x3E      | CFTP                                     | CFTP                                     |                                          |
| 63      | 0x3F      |                                          | Any local network                        |                                          |
| 64      | 0x40      | SAT-EXPAK                                | SATNET and Backroom EXPAK                |                                          |
| 65      | 0x41      | KRYPTOLAN                                | Kryptolan                                |                                          |
| 66      | 0x42      | RVD                                      | MIT [Remote Virtual Disk Protocol](https://zh.wikipedia.org/w/index.php?title=Remote_Virtual_Disk_Protocol&action=edit&redlink=1) |                                          |
| 67      | 0x43      | IPPC                                     | [Internet Pluribus Packet Core](https://zh.wikipedia.org/w/index.php?title=Internet_Pluribus_Packet_Core&action=edit&redlink=1) |                                          |
| 68      | 0x44      |                                          | Any distributed file system              |                                          |
| 69      | 0x45      | SAT-MON                                  | SATNET Monitoring                        |                                          |
| 70      | 0x46      | VISA                                     | VISA Protocol                            |                                          |
| 71      | 0x47      | IPCV                                     | Internet Packet Core Utility             |                                          |
| 72      | 0x48      | CPNX                                     | Computer Protocol Network Executive      |                                          |
| 73      | 0x49      | CPHB                                     | [Computer Protocol Heart Beat](https://zh.wikipedia.org/w/index.php?title=Computer_Protocol_Heart_Beat&action=edit&redlink=1) |                                          |
| 74      | 0x4A      | WSN                                      | [Wang Span Network](https://zh.wikipedia.org/w/index.php?title=Wang_Span_Network&action=edit&redlink=1) |                                          |
| 75      | 0x4B      | PVP                                      | [Packet Video Protocol](https://zh.wikipedia.org/w/index.php?title=Packet_Video_Protocol&action=edit&redlink=1) |                                          |
| 76      | 0x4C      | BR-SAT-MON                               | Backroom SATNET Monitoring               |                                          |
| 77      | 0x4D      | SUN-ND                                   | SUN ND PROTOCOL-Temporary                |                                          |
| 78      | 0x4E      | WB-MON                                   | WIDEBAND Monitoring                      |                                          |
| 79      | 0x4F      | WB-EXPAK                                 | WIDEBAND EXPAK                           |                                          |
| 80      | 0x50      | ISO-IP                                   | International Organization for Standardization Internet Protocol |                                          |
| 81      | 0x51      | VMTP                                     | [Versatile Message Transaction Protocol](https://zh.wikipedia.org/w/index.php?title=V_(operating_system)&action=edit&redlink=1) | [RFC 1045](https://tools.ietf.org/html/rfc1045) |
| 82      | 0x52      | SECURE-VMTP                              | Secure Versatile Message Transaction Protocol | [RFC 1045](https://tools.ietf.org/html/rfc1045) |
| 83      | 0x53      | VINES                                    | VINES                                    |                                          |
| 84      | 0x54      | TTP                                      | TTP                                      |                                          |
| 84      | 0x54      | IPTM                                     | [Internet Protocol Traffic Manager](https://zh.wikipedia.org/w/index.php?title=Internet_Protocol_Traffic_Manager&action=edit&redlink=1) |                                          |
| 85      | 0x55      | NSFNET-IGP                               | NSFNET-IGP                               |                                          |
| 86      | 0x56      | DGP                                      | [Dissimilar Gateway Protocol](https://zh.wikipedia.org/w/index.php?title=Dissimilar_Gateway_Protocol&action=edit&redlink=1) |                                          |
| 87      | 0x57      | TCF                                      | TCF                                      |                                          |
| 88      | 0x58      | EIGRP                                    | [EIGRP](https://zh.wikipedia.org/wiki/EIGRP) |                                          |
| 89      | 0x59      | OSPF                                     | [Open Shortest Path First](https://zh.wikipedia.org/wiki/Open_Shortest_Path_First) | [RFC 1583](https://tools.ietf.org/html/rfc1583) |
| 90      | 0x5A      | Sprite-RPC                               | Sprite RPC Protocol                      |                                          |
| 91      | 0x5B      | LARP                                     | [Locus Address Resolution Protocol](https://zh.wikipedia.org/w/index.php?title=Locus_Address_Resolution_Protocol&action=edit&redlink=1) |                                          |
| 92      | 0x5C      | MTP                                      | [Multicast Transport Protocol](https://zh.wikipedia.org/w/index.php?title=Multicast_Transport_Protocol&action=edit&redlink=1) |                                          |
| 93      | 0x5D      | AX.25                                    | [AX.25](https://zh.wikipedia.org/w/index.php?title=AX.25&action=edit&redlink=1) |                                          |
| 94      | 0x5E      | IPIP                                     | [IP-within-IP Encapsulation Protocol](https://zh.wikipedia.org/wiki/IP_in_IP) | [RFC 2003](https://tools.ietf.org/html/rfc2003) |
| 95      | 0x5F      | MICP                                     | [Mobile Internetworking Control Protocol](https://zh.wikipedia.org/w/index.php?title=Mobile_Internetworking_Control_Protocol&action=edit&redlink=1) |                                          |
| 96      | 0x60      | SCC-SP                                   | Semaphore Communications Sec. Pro        |                                          |
| 97      | 0x61      | ETHERIP                                  | Ethernet-within-IP Encapsulation         | [RFC 3378](https://tools.ietf.org/html/rfc3378) |
| 98      | 0x62      | ENCAP                                    | Encapsulation Header                     | [RFC 1241](https://tools.ietf.org/html/rfc1241) |
| 99      | 0x63      |                                          | Any private encryption scheme            |                                          |
| 100     | 0x64      | GMTP                                     | GMTP                                     |                                          |
| 101     | 0x65      | IFMP                                     | [Ipsilon Flow Management Protocol](https://zh.wikipedia.org/w/index.php?title=Ipsilon_Flow_Management_Protocol&action=edit&redlink=1) |                                          |
| 102     | 0x66      | PNNI                                     | PNNI over IP                             |                                          |
| 103     | 0x67      | PIM                                      | [Protocol Independent Multicast](https://zh.wikipedia.org/w/index.php?title=Protocol_Independent_Multicast&action=edit&redlink=1) |                                          |
| 104     | 0x68      | ARIS                                     | IBM's ARIS (Aggregate Route IP Switching) Protocol |                                          |
| 105     | 0x69      | SCPS                                     | [SCPS (Space Communications Protocol Standards)](http://scps.org/) | SCPS-TP[[1\]](https://zh.wikipedia.org/wiki/IP%E5%8D%8F%E8%AE%AE%E5%8F%B7%E5%88%97%E8%A1%A8#cite_note-1) |
| 106     | 0x6A      | [QNX](https://zh.wikipedia.org/wiki/QNX) | QNX                                      |                                          |
| 107     | 0x6B      | A/N                                      | Active Networks                          |                                          |
| 108     | 0x6C      | IPComp                                   | [IP Payload Compression Protocol](https://zh.wikipedia.org/w/index.php?title=IP_Payload_Compression_Protocol&action=edit&redlink=1) | [RFC 3173](https://tools.ietf.org/html/rfc3173) |
| 109     | 0x6D      | SNP                                      | [Sitara Networks Protocol](https://zh.wikipedia.org/w/index.php?title=Sitara_Networks_Protocol&action=edit&redlink=1) |                                          |
| 110     | 0x6E      | Compaq-Peer                              | [Compaq Peer Protocol](https://zh.wikipedia.org/w/index.php?title=Compaq_Peer_Protocol&action=edit&redlink=1) |                                          |
| 111     | 0x6F      | IPX-in-IP                                | [IPX in IP](https://zh.wikipedia.org/w/index.php?title=IPX_in_IP&action=edit&redlink=1) |                                          |
| 112     | 0x70      | VRRP                                     | [Virtual Router Redundancy Protocol](https://zh.wikipedia.org/w/index.php?title=Virtual_Router_Redundancy_Protocol&action=edit&redlink=1), [Common Address Redundancy Protocol](https://zh.wikipedia.org/w/index.php?title=Common_Address_Redundancy_Protocol&action=edit&redlink=1) (not [IANA](https://zh.wikipedia.org/wiki/Internet_Assigned_Numbers_Authority) assigned) | VRRP:[RFC 3768](https://tools.ietf.org/html/rfc3768) |
| 113     | 0x71      | PGM                                      | [PGM Reliable Transport Protocol](https://zh.wikipedia.org/wiki/Pragmatic_General_Multicast) | [RFC 3208](https://tools.ietf.org/html/rfc3208) |
| 114     | 0x72      |                                          | Any 0-hop protocol                       |                                          |
| 115     | 0x73      | L2TP                                     | [Layer Two Tunneling Protocol Version 3](https://zh.wikipedia.org/w/index.php?title=L2TPv3&action=edit&redlink=1) | [RFC 3931](https://tools.ietf.org/html/rfc3931) |
| 116     | 0x74      | DDX                                      | D-II Data Exchange (DDX)                 |                                          |
| 117     | 0x75      | IATP                                     | [Interactive Agent Transfer Protocol](https://zh.wikipedia.org/w/index.php?title=Interactive_Agent_Transfer_Protocol&action=edit&redlink=1) |                                          |
| 118     | 0x76      | STP                                      | [Schedule Transfer Protocol](https://zh.wikipedia.org/w/index.php?title=Schedule_Transfer_Protocol&action=edit&redlink=1) |                                          |
| 119     | 0x77      | SRP                                      | [SpectraLink Radio Protocol](https://zh.wikipedia.org/w/index.php?title=SpectraLink_Radio_Protocol&action=edit&redlink=1) |                                          |
| 120     | 0x78      | UTI                                      | Universal Transport Interface Protocol   |                                          |
| 121     | 0x79      | SMP                                      | [Simple Message Protocol](https://zh.wikipedia.org/w/index.php?title=Simple_Message_Protocol&action=edit&redlink=1) |                                          |
| 122     | 0x7A      | SM                                       | Simple Multicast Protocol                | [draft-perlman-simple-multicast-03](http://tools.ietf.org/html/draft-perlman-simple-multicast-03) |
| 123     | 0x7B      | PTP                                      | [Performance Transparency Protocol](https://zh.wikipedia.org/w/index.php?title=Performance_Transparency_Protocol&action=edit&redlink=1) |                                          |
| 124     | 0x7C      | IS-IS over IPv4                          | [Intermediate System to Intermediate System (IS-IS) Protocol](https://zh.wikipedia.org/wiki/IS-IS) over [IPv4](https://zh.wikipedia.org/wiki/IPv4) | [RFC 1142](https://tools.ietf.org/html/rfc1142) and [RFC 1195](https://tools.ietf.org/html/rfc1195) |
| 125     | 0x7D      | FIRE                                     | Flexible Intra-AS Routing Environment    |                                          |
| 126     | 0x7E      | CRTP                                     | [Combat Radio Transport Protocol](https://zh.wikipedia.org/w/index.php?title=Combat_Radio_Transport_Protocol&action=edit&redlink=1) |                                          |
| 127     | 0x7F      | CRUDP                                    | [Combat Radio User Datagram](https://zh.wikipedia.org/w/index.php?title=Combat_Radio_User_Datagram&action=edit&redlink=1) |                                          |
| 128     | 0x80      | SSCOPMCE                                 | Service-Specific Connection-Oriented Protocol in a Multilink and Connectionless Environment | [ITU-T Q.2111 (1999)](http://www.itu.int/rec/T-REC-Q.2111-199912-I) |
| 129     | 0x81      | IPLT                                     |                                          |                                          |
| 130     | 0x82      | SPS                                      | [Secure Packet Shield](https://zh.wikipedia.org/w/index.php?title=Secure_Packet_Shield&action=edit&redlink=1) |                                          |
| 131     | 0x83      | PIPE                                     | Private IP Encapsulation within IP       | [Expired I-D draft-petri-mobileip-pipe-00.txt](http://www.watersprings.org/pub/id/draft-petri-mobileip-pipe-00.txt) |
| 132     | 0x84      | SCTP                                     | [Stream Control Transmission Protocol](https://zh.wikipedia.org/wiki/Stream_Control_Transmission_Protocol) |                                          |
| 133     | 0x85      | FC                                       | [Fibre Channel](https://zh.wikipedia.org/w/index.php?title=Fibre_Channel&action=edit&redlink=1) |                                          |
| 134     | 0x86      | RSVP-E2E-IGNORE                          | Reservation Protocol (RSVP) End-to-End Ignore | [RFC 3175](https://tools.ietf.org/html/rfc3175) |
| 135     | 0x87      | Mobility Header                          | Mobility Extension Header for IPv6       | [RFC 6275](https://tools.ietf.org/html/rfc6275) |
| 136     | 0x88      | UDPLite                                  | [Lightweight User Datagram Protocol](https://zh.wikipedia.org/w/index.php?title=UDP_Lite&action=edit&redlink=1) | [RFC 3828](https://tools.ietf.org/html/rfc3828) |
| 137     | 0x89      | MPLS-in-IP                               | [Multiprotocol Label Switching](https://zh.wikipedia.org/w/index.php?title=Multiprotocol_Label_Switching&action=edit&redlink=1) Encapsulated in IP | [RFC 4023](https://tools.ietf.org/html/rfc4023) |
| 138     | 0x8A      | manet                                    | [MANET](https://zh.wikipedia.org/w/index.php?title=Mobile_ad_hoc_network&action=edit&redlink=1) Protocols | [RFC 5498](https://tools.ietf.org/html/rfc5498) |
| 139     | 0x8B      | HIP                                      | [Host Identity Protocol](https://zh.wikipedia.org/w/index.php?title=Host_Identity_Protocol&action=edit&redlink=1) | [RFC 5201](https://tools.ietf.org/html/rfc5201) |
| 140     | 0x8C      | Shim6                                    | [Site Multihoming by IPv6 Intermediation](https://zh.wikipedia.org/w/index.php?title=Site_Multihoming_by_IPv6_Intermediation&action=edit&redlink=1) | [RFC 5533](https://tools.ietf.org/html/rfc5533) |
| 141     | 0x8D      | WESP                                     | [Wrapped Encapsulating Security Payload](https://zh.wikipedia.org/w/index.php?title=Wrapped_Encapsulating_Security_Payload&action=edit&redlink=1) | [RFC 5840](https://tools.ietf.org/html/rfc5840) |
| 142     | 0x8E      | ROHC                                     | [Robust Header Compression](https://zh.wikipedia.org/w/index.php?title=Robust_Header_Compression&action=edit&redlink=1) | [RFC 5856](https://tools.ietf.org/html/rfc5856) |
| 143-252 | 0x8F-0xFC | UNASSIGNED                               |                                          |                                          |
| 253-254 | 0xFD-0xFE | Use for experimentation and testing      | [RFC 3692](https://tools.ietf.org/html/rfc3692) |                                          |
| 255     | 0xFF      | Reserved.                                |                                          |                                          |



[ovs-基本操作](http://blog.sina.com.cn/s/blog_701557360101n6ye.html)

