# No_Telstra_IPv6
朋友最近去JB签了Telstra套餐之后，一直反映用流量的时候，其他应用都很快，但是下载和上传qq的图片基本上是龟速，而且很容易失败，相比之下之前使用的optus网络就很快。

研究了一下之后，发现qq负责上传和下载群聊图片的域名gchat.qpic.cn，从海外可以同时解析出ipv4和ipv6两种地址。

ipv4地址为香港腾讯云接入的“海外访问优化”，ipv6地址则为上海腾讯云的ip地址。

Optus Mobile这边默认APN下只有单栈IPv4，所以只会访问这个“海外优化”的香港腾讯云的地址，在访问速度上没有太大问题。

而Telstra这边则比较复杂，只有单栈IPv6，所有IPv4的流量会通过464XLAT的方式，通过IPv6内网转发到Telstra的公共IPv4出口来访问IPv4内容，并且貌似Telstra这边的运营商配置文件或者默认的DNS的设置会优先使用IPv6地址，从而导致在使用移动网络情况下会优先连接到这个域名解析出来的上海腾讯云的IPv6地址。然后Telstra的ipv6访问过去绕了全球一圈（澳洲-香港-美西-美东-欧洲-上海），所以会出现下载和上传群图片很慢的情况。

然而朋友用的是iOS，无法直接在系统设置里修改APN来禁用ipv6的访问，在经过了一系列调查与研究之后，了解到了通过自己制作描述文件来修改APN，从而达到禁用移动数据下的ipv6访问的目的的方法，特此来分享给其他同样在Telstra移动数据下用QQ很慢的用户。

懒人方法：
1.在iOS设备上用Safari打开以下链接，选择安装描述文件到iPhone上

https://raw.githubusercontent.com/haha114514/No_Telstra_IPv6/main/Telstra.No.IPv6.mobileconfig

2.前往系统设置中启用下载的配置描述文件

3.打开移动数据，访问ip.sb，确认是否只有ipv4地址（没有ipv6地址）

4.测试移动数据下qq群上传/下载图片速度。

自助方法（如果不放心上面的profile的话）：
1.请自行参考https://www.youtube.com/watch?v=Nzx9T7GtmT4
