# Fuck_Telstra_IPv6

## ** Update 03/12/2022 使用本配置文件可以有效解决小红书图片无法加载，微信无法刷新消息等问题


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

4.测试移动数据下qq上传/下载图片速度。

### PS.目前已知有个小问题是，有些时候系统更新或者更换sim卡（包括切换实体sim或者esim），可能会导致配置文件失效。如果突然出现图片又加载不出来了或者微信又卡了，还是访问ip.sb看看ipv6是不是回来了。如果回来了的话，删掉描述文件再重新添加一遍即可

自助方法（如果不放心上面的profile的话）：
1.请自行参考https://www.youtube.com/watch?v=Nzx9T7GtmT4

## 对于某些好奇为什么小红书和微信刷不出来的原因
省流版：Telstra的DNS解析v6地址有问题，但是解析出来的v4地址没问题。所以关掉IPv6之后就优先走解析出来的v4地址了，访问就没问题了。

详细版：通过Telstra的DNS解析小红书图片域名sns-img-hw.xhscdn.com，AAAA结果（v6解析）解析出来是一个指向Telstra的地址（正确的解析应该也是一个属于Akamai悉尼的IPv6地址），然后A结果（v4解析）是正确的akamai CDN的地址。盲猜是Telstra所提供的DNS的问题，导致小红书图片域名的AAAA结果被解析到一个无法访问的地址，从而加载不出任何图片（因为有v6的时候，优先通过v6访问）。
### 通过Telstra移动网络解析小红书的图片域名

![40147af63ecfe7cde2acd89bbed94363](https://user-images.githubusercontent.com/47912037/205447304-c956123b-f87d-4c36-b70a-ebff61b80fd4.png)
