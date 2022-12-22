# Fuck_Telstra_IPv6

## ** Update 03/12/2022 使用本配置文件可以有效解决小红书图片无法加载，微信无法刷新消息等问题（Telstra已经于05/12/2022修复此问题）

## ** Update 22/12/2022 据说某些用户还是会出现微信加载不出图片等问题，依旧可以尝试使用本配置文件。

朋友最近去JB签了Telstra套餐之后，反馈用流量的时候，其他本地和海外应用都很快，但是下载和上传qq的图片基本上是龟速，而且很容易失败，相比之下之前使用的optus网络就很快。

研究了一下之后，发现qq负责上传和下载群聊图片的域名gchat.qpic.cn，从海外可以同时解析出ipv4和ipv6两种地址。

IPv4地址为香港腾讯云接入的“海外访问优化”，IPv6地址则为上海腾讯云的ip地址。

Optus Mobile这边默认APN下只有单栈IPv4，所以只会访问这个“海外优化”的香港腾讯云的地址，在访问速度上没有太大问题。

而Telstra这边默认的APN（Telstra.wap)则比较复杂，只有单栈IPv6，所有IPv4的流量会通过464XLAT的方式，通过IPv6内网转发到Telstra的公共IPv4出口来访问IPv4内容，并且貌似Telstra这边的iOS运营商配置文件或者默认的DNS的设置会优先使用IPv6地址，从而导致在使用移动网络情况下会优先连接到这个域名解析出来的上海腾讯云的IPv6地址。然后Telstra的IPv6访问过去绕了全球一圈（澳洲-香港-美西-美东-欧洲-上海），所以会出现下载和上传群图片很慢的情况。

然而朋友用的是iOS，无法直接在系统设置里修改APN来禁用IPv6的访问。在经过了一系列调查与研究之后，了解到了通过自己制作描述文件来修改APN，从而达到禁用移动数据下的IPv6访问的目的的方法，特此来分享给其他同样在Telstra移动数据下使用QQ与很慢的用户。

懒人方法：
1.在iOS设备上用Safari打开以下链接，选择安装描述文件到iPhone上

https://raw.githubusercontent.com/haha114514/No_Telstra_IPv6/main/Telstra_IPv6_sucks.mobileconfig

2.前往系统设置中启用下载的配置描述文件

3.关闭Wi-Fi然后打开移动数据，访问ip.sb，确认是否只有IPv4地址（没有IPv6地址）

<img width="300" alt="image" src="https://user-images.githubusercontent.com/47912037/205486292-8fafeed2-3886-4c87-83de-bf643e50ba5b.png">


4.测试移动数据下qq上传/下载图片速度。

### PS.目前已知有个小问题是，有些时候系统更新或者更换sim卡（包括切换实体sim或者esim），可能会导致配置文件失效。如果突然出现图片又加载不出来了或者微信又卡了，还是访问ip.sb看看ipv6是不是回来了。如果回来了的话，删掉描述文件再重新添加一遍即可

自助方法（如果不放心上面的profile的话）：
1.请自行参考https://www.youtube.com/watch?v=Nzx9T7GtmT4

## 对于某些好奇为什么小红书和微信刷不出来的原因
省流版：Telstra的DNS解析v6地址有问题，但是解析出来的v4地址没问题。所以关掉IPv6之后就优先走解析出来的v4地址了，访问就没问题了。

详细版：通过Telstra的DNS解析小红书图片域名sns-img-hw.xhscdn.com，AAAA结果（v6解析）解析出来是一个指向Telstra的地址（正确的解析应该也是一个属于Akamai悉尼的IPv6地址），然后A结果（v4解析）是正确的akamai CDN的地址。盲猜是Telstra所提供的DNS的问题，导致小红书图片域名的AAAA结果被解析到一个无法访问的地址，从而加载不出任何图片（因为有v6的时候，优先通过v6访问）。
### 通过Telstra移动网络解析小红书的图片域名

<img width="437" alt="image" src="https://user-images.githubusercontent.com/47912037/205472849-4d6bb3fe-0374-4d59-b53f-736db2fb0e4e.png">

通过查询，该IPv6是属于Telstra的

<img width="397" alt="image" src="https://user-images.githubusercontent.com/47912037/205472926-b65af806-dec1-4b70-a7fc-bbbe319a3039.png">

### **Update 05/12/2022 看来这个IP是Telstra用于464XLAT隧道或者反向代理（透明代理）的，所以不是Telstra的DNS问题，而是这个隧道突然炸掉了，所以导致维州这几天（01/12/2022-05/12/2022）炸掉了，目前应该是已经修好了的。

### 洗衣机论坛对这方面的解释：
Telstra is using DNS64 and NAT64 on its 'telstra.wap' APN.

Find more detail here: /thread/3vy5n749

Basically the following occurs.

Your device does a DNS query to Telstra's DNS server.
Telstra's DNS server queries another DNS server for the A (IPv4) and AAAA (IPv6) record.
The response to Telstra's DNS server is only a A record.
NSLOOKUP test1.sunshinecomputer.net.au 8.8.8.8 returns only an A record
Telstra DNS's in this case does a DNS64 translation.
Takes the returned IP Address, and converts it into hex.
144, 130, 97, 86 -> 0x90, 0x82, 0x61, 0x56
Appends the hex of the IP to the IPv6 prefix for NAT64 translation
2001:8004:11d0:4e2a::/96 is the prefix
2001:8004:11d0:4e2a::9082:6156
Telstra's DNS returns the AAAA record of '2001:8004:11d0:4e2a::9082:6156' to the initial device that made the request.
The device with the IPv6 address for the target server, opens the connection as it would normally with an IPv4 address, but with an IPv6 target.
NOTE: It is key at this point that the device and application must support IPv6.
The packet with the target IPv6 address is within the NAT64 prefix, and is routed to Telstra's NAT64 service.
The NAT64 service receives the packet, and does a translation from the IPv6 address back to the IPv4 address and sets itself as the source IPv4 address. It keeps a record of this state
The target server receives the packet with the IPv4 source as NAT64 IPv4 address
Server responds to the packet
NAT64 receives the IPv4 packet from the server
NAT64 references the state of the initial packet to the server, and uses that to update the destination of the packet to the IPv6 address of the device, and sends the packet.
Device then receives the packet via IPv6 from the NAT64 service.
This is good, only if:

Developers support IPv6 on their devices and applications without needing to deploy new IPv6 servers.
Developers hadn't hardcoded any IPv4 addresses (requires DNS to update the IPv4 address)
There is an extension of this called 464XLAT. The above still occurs, but the device still has its own IPv4 address, and allows applications to use the IPv4 address and the device translates the IPv4 address to IPv6 and forwards to the NAT64 service still. 

#### 默认telstra.internet+ipv4/v6(protocol)或者telstra.wap+ipv6(Protocol)下的curl信息，可以看到是优先走了解析出来了有问题的v6地址的

<img width="434" alt="image" src="https://user-images.githubusercontent.com/47912037/205472825-421844b3-bec1-4845-801e-22da758ac5b2.png">

#### 修改为telstra.internet+ipv4之后的，可以看到是走了解析出来正常的v4地址的

<img width="419" alt="image" src="https://user-images.githubusercontent.com/47912037/205472778-9526fc4c-707f-4e9d-8513-f2438258ecb4.png">



