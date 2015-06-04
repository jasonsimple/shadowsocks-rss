# ShadowsocksR CSharp 版本3.4.1 #

最新版下载链接：[https://github.com/breakwa11/shadowsocks-csharp/releases](https://github.com/breakwa11/shadowsocks-csharp/releases)  
发布链接：[https://github.com/breakwa11/shadowsocks-rss](https://github.com/breakwa11/shadowsocks-rss)  
服务端配置教程：[Wiki](https://github.com/breakwa11/shadowsocks-rss/wiki/Server-Setup) （含单用户和多用户）

本版本发布一个新的实验性TCP连接下的握手协议，在原来的握手协议外套一层验证和混淆的头部（以下为加密前的报文）：  
标志版本号(1byte)|首包总长度(2byte)|随机填充长度(1byte)|随机填充数据|原ss首数据包|CRC32(4byte)  
其中，标志版本号=0x88，首包总长度是包含版本号和CRC32的长度，随机填充长度值范围为1~255，随机填充数据的长度为随机填充长度值减1，验证CRC32时，计算整个头部的CRC32看是否等于0xFFFFFFFF即可。注意服务端还应处理可能的粘包。客户端可以实现为把原来的首握手包连同第一个数据包一起作为首数据包（不一定必须有第一个数据包，如ssh），或把首包的中间任意地方拆开，剩余部分放在CRC32之后亦可。  
目前，SSR服务端的tcprelay.py源代码35行的地方，有一个`FORCE_NEW_PROTOCOL`的开关，如果是个人用户，建议打开，然后客户端相应节点也打开相应的开关，以避免GFW的主动探测封锁和包长度分析封锁。而多用户端站长，如果希望使用此协议，可以对用户引导，待大家都使用此功能再启用开关。目前此协议和少数几个开发者讨论过，确认可行，如果你发现此协议有什么问题，请尽快提出。如测试下来发现此协议确实在对抗封锁有效，那么将聚集大家升级各平台的实现版本。

另外此程序为**开源软件**，GPLv3协议，有部分人在前两天还以为没有开源，所以特此说明。

你要是有兴趣和我联系的话，特别是编程技术上的支持，那就发邮件到mmgac001[at]gmail[dot]com，或者到  
Google Group: https://groups.google.com/forum/#!forum/shadowsocksr  
社区: https://plus.google.com/communities/117390969460066916686


#### 关于代理UDP（以TCP转发或UDP转发均可） ####
此版本支持配合使用`SocksCap64/SocksCap/ProxyCap`等工具把需要TCP和UDP代理的程序通过本版本程序转发。  
即不需要折腾`OpenWrt`或路由器即可在PC上获得UDP转发的能力，配置容易了很多，是游戏玩家的福音  
如果你是个人用户，可自行更新服务端或向你的ss站长提议更新后端。  
如果你是站长，可从[https://github.com/breakwa11/shadowsocks/tree/manyuser](https://github.com/breakwa11/shadowsocks/tree/manyuser)获取最新的多用户分支代码，与原版本兼容。  
具体配置方法可参阅[服务端配置教程](https://github.com/breakwa11/shadowsocks-rss/wiki/Server-Setup)  
如有任何问题可发issue到github。

#### 配置术语说明： ####
1. TCP over UDP  
打钩则把TCP包以UDP隧道发送，协议还在完善中，需要和相应的服务端配合使用。目前协议的设计是优化下载和看视频（打开网页不见得比原来的TCP好），但如果在封锁UDP严重的地区，效果会比直接使用TCP更差，若丢包率在10%~50%的环境下，均会有一定的提升。建议不要过度使用以免IP被盯上。如果服务器不支持，打钩此选项会导致网页无法打开。
2. UDP over TCP   
不打钩即以UDP方式发送UDP包，打钩则把UDP包在TCP里发送。以TCP方式发送UDP包在封锁UDP的网络环境下特别有用，如无此需求就不必打钩。  
如果在发送UDP数据失败，可能是服务器不支持。此选项不会影响浏览网页（TCP）
3. 新TCP连接协议  
TCP连接采用新的握手协议，同时具有抗暴力探测（关闭原协议的情况下），强制随机化包长度（抗包长度分析），目前此协议在测试阶段，具体协议内容可参阅源代码，或前文说明。同样需要服务端支持，否则会连接失败，影响浏览网页，和UDP over TCP。
4. 混淆UDP协议  
用于随机化包长度，但目前UDP未发现明显特征，如无必要可不打钩。
5. 重连次数  
目标服务器连接失败时选择其它服务器再次尝试连接，仅当没有发送接收任何数据时进行（即出现密码错误或加密方式错误时不能重连，因为已有数据发送）
6. 超时秒数（TTL）  
TTL为Time to Live的缩写，表示连接超过多久没有再发送或接收数据时断开（服务端默认值为300），时间单位是秒，较小的值可减少浏览器出现卡顿时的时间，但同时在网络连接不佳时可能会导致部分页面元素或整个页面随机性下载失败。一般网页浏览建议设置为30~120，下载或看视频时建议直接设置为0。设置为0表示不使用TTL设置（实际TTL值由服务端及客户端较小的一方控制）。

#### 快捷操作提示： ####
1. 双击任务栏图标弹出服务器配置窗口，而右键图标弹出菜单
2. 中键点击任务栏图标弹出服务器连接统计窗口
3. 连接统计窗口点击服务器会切换当前服务器
4. 连接统计窗口双击服务器会打开服务器配置窗口
5. 点击连接统计窗口列标题可排序（部分列不允许排序）
6. 连接统计窗口，在连接状态的错误记录列，双击则重置错误数，双击百分比列则重置本服务器的所有信息
7. 连接统计窗口，在连接状态的“开”一列，鼠标点击切换开关状态（随机时有效），红色为关闭
8. 连接统计窗口，右键弹出清空所有的菜单
9. 连接统计窗口，双击连接数，断开该节点现有的连接（不一定立即清零，双击过就行了）

#### 其它提示： ####
- 随机选择服务器功能，适用于网页浏览，不适用于看视频或下载等需要大流量的环境。如需下载请在连接统计窗口通过下载测速测试速度最快的服务器然后单独连接之。  
- 连接统计窗口“连错”是指最近连接连续出错次数，包含发送了数据但无返回数据，或连接超时，或连接失败，或连接被重置等，可能为ss密码错误，也可能是实际访问的结果，有返回数据包时即会归0。启用了“自动禁用出错服务器”时，连错50次会自动临时禁用本节点，若服务器连接失败则3次便禁用，可在统计窗口里重新开启。但作为二级代理时不管连错多少均不会自动禁用。  
- ChnIPList 使用这个PAC，让国内的站直连，国外的走代理，不过不一定准确，如果DNS被污染为国内地址段时则会有问题，但大部分都能正常工作。此PAC对firefox可能不太友好。

#### 更新记录： ####
版本3.4.1  
1.增加新TCP连接协议

版本3.4.0  
1.使用新的GFWList地址  
2.增加一个实验性功能TCP over UDP（目前有BUG，轻度使用还可，需要使用相应的服务端）

版本3.3.6  
1.修正“自动禁用出错服务器”选项保存的问题  
2.所有临时文件全部放到./temp/下  
3.连接方式不同也视为不同服务器分别统计  
4.删除升级提示检查  
5.删除实验性功能代码并开源https://github.com/breakwa11/shadowsocks-csharp

版本3.3.5  
1.合并部分主干代码，改用privoxy替代polipo（主程序增大140Kb）  
2.增加“低错误优先”的随机方式  
3.细调“低延迟优先”的算法  
4.增加“自动禁用出错服务器”选项  
5.连接超过超时秒数后断开算作“超时”

版本3.3.4  
1.修正打开统计列表后，程序退出不正常的错误  
2.修正UDP over UDP的连接错误  
3.修正二重代理时UDP连接错误

版本3.3.2  
1.合并部分主干代码  
2.断开当前所有连接功能  
3.优化统计窗口资源占用

版本3.3.1  
1.修正连接数统计和管理错误  
2.修正多重代理时UDP代理错误  
3.重新实现UDP over TCP（与之前版本不兼容，以前的实现不正确，服务端需更新）

版本3.2.2  
1.修正UDP下chacha20加密错误以及内存泄露  
2.提升UDP下加解密速度  
3.优化统计计算速度  
4.随机选择服务器改名负载均衡  
5.统计列表简化显示效果，减少卡顿  
6.自动更新方式调整

版本3.2.0  
1.修正部分TCP连接失败的问题  
2.增加Socks5代理设置

版本3.1.4  
1.调整TCP发包方式，把首协议包与第一个数据包连接发送  
2.移除混淆TCP选项  
3.服务端统计三列错误统计列改用“连错”列代替  
4.不要把版本号看成圆周率（明明差一个小数点）

版本3.1.3  
1.本地连接的TTL支持，默认TTL=0（0表示不启用）  
2.重连次数的保存修正

版本3.1.2  
1.连接统计窗口排序修正  
2.重连次数配置（默认值3）  
3.UDP over UDP选项更换为UDP over TCP，即高级选项全不打钩为通用设置，通用于所有服务端  
4.log文件和polipo放于temp子目录  
5.统计窗口数值格式化调整  
6.解决config问题，移除config文件  
7.允许密码显示

版本3.1.1  
1.合并主干部分代码  
2.增加混淆UDP选项  
3.支持接收混淆UDP包  
4.连接数统计修正  
5.鼠标中键点击任务栏图标弹出服务器连接统计窗口  
6.修正配置界面部分环境下按钮不显示  
7.附加config文件避免部分系统运行异常  
8.改用7zip压缩，减少一半体积

版本3.0.1  
1.配置保存bug修正  
2.右键菜单整理  
3.配置界面调整  
4.修正UDP握手协议实现与ProxyCap兼容  
5.加入源md格式说明文档

版本3.0  
1.软件更名  
2.修正 UDP over UDP 的加解密错误

版本2.3.2  
1.修正ipv4连接的问题  
其它：目前已知使用pc版微信发图发文件会导致polipo崩溃，建议改用Provixy，具体使用方法可以参考网上教程

版本2.3.1.9  
1.修正http连接会出现连接失败的问题（同时修正了自动更新和pac更新）  
2.增加连接管理，双击连接数即可断开该节点现有连接  
3.增强UDP连接兼容性  
4.更新地址可以配置窗口点击  

版本2.3.1.8  
1.支持TCP协议头混淆（需服务端更新支持）  
2.UDP包混淆（需服务端更新支持）  
3.支持UDP包通过 UDP Relay 转发（需服务端更新支持）  
4.支持Socks4/Socks4a协议  
5.局域网共享时支持同时监听IPv4/IPv6  
6.配置添加UDP包通过方式  
7.配置添加TCP协议头混淆

版本2.3.1.7  
1.IPv6连接修正  
2.随机时“按次序”方式的bug修正  
3.IPv6地址显示方式调整  
4.空连统计包含连接重置等错误  
5.服务器连接统计列排序功能  
6.取消自动关闭无效服务器，防误判  
7.手工检查更新

版本2.3.1.6  
1.增加随机选择服务器的方式  
2.连接数统计修正  
3.支持代理UDP（以TCP转发），需要服务端更新

版本2.3.1.5  
1.调整配置窗口的设置体验  
2.软件自动更新支持  

版本2.3.1.4R2  
1.修正polipo某些情况下运行异常  
2.修正连接统计窗口一些错误  
3.修正部分dns错误未处理的问题  
4.三次错误后自动关闭无效服务器  
5.增加ChnIPList的pac，国内不代理，国外全走代理  
6.调整连接统计窗口列顺序  
7.节点自动重连尝试（开启随机选择服务器时，第一次重连相同节点，第二次、第三次选择其它节点）  
8.从剪贴板复制多行文本地址功能修正  
9.服务器配置显示二维码，增加列表高度

版本2.3.1.4  
1.polipo配置修正  
2.使用当前目录运行polipo，完美支持多开（端口同时支持socks5和http协议）  
3.增加当前下载速度统计，以及最大下载速度记录  
4.连接统计窗口优化显示效率  
5.服务器配置显示ss链接，以便快速复制  
6.连接统计窗口支持手工调整列宽  

版本2.3.1.3  
1.服务器的DNS缓冲（TTL=600），让直接填写域名与写IP一样稳定  
2.服务器连接状态记录，含上传下载流量，连接延迟统计，连接错误统计  
3.服务器独立开关，在连接状态的“开”一列，鼠标点击之，红色为关闭  
4.连接记录清除，在连接状态的错误记录三列，双击则重置错误数，双击百分比列则重置本服务器的所有信息  
5.统计列表标题显示监听方式及端口（方便多开的时候对应上）  
6.从剪贴板复制地址，方便从ss://xxxx这种格式快速输入  

版本2.3.1.2  
1.允许程序多开  
2.延迟统计，自动在低延迟服务器之中选择（高延迟的也会有低概率选取到）  
3.二维码菜单位置变动  
4.偶然会程序崩溃的问题不确定解决了没有，如还有发生请联系作者  

版本2.3.1.1  
1.随机选择服务器