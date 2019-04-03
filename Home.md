### [《2019年认证认可检验检测工作要点》发布](https://mp.weixin.qq.com/s?__biz=MzAwNzUzNzU1MQ==&mid=2651208028&idx=1&sn=dc2b053d960cf164e0632ebd9b71a194&chksm=808e0fbcb7f986aaa8ccec93b13bb6ff25c4fd35008ee9a8ed0ca0ab74ddd10bd8b4917ff111&mpshare=1&scene=1&srcid=04021nwYGPk074NyAIKCxFna&key=f1b1b4aaf75e0cb5cb8ab15d5a3f546ed4f185505d2fe38d77edc2c3dcda48f0da5b57399af4672b5a0eb99169b0a39ff2b3ce7d5b08ab0f486bdbbcb347231fbae428f7185310a4619975b2039ace4b&ascene=1&uin=MTAyOTYwOTUyMw%3D%3D&devicetype=Windows+10&version=62060739&lang=zh_CN&pass_ticket=d2061J4FurqTKls7auyEcAYw58nxnbEUHnCWaR9R7Jd25EjXtoq49BmSEMa9His0)
### [额头出汗原则](https://wiki.mbalib.com/wiki/%E8%BE%9B%E5%8B%A4%E5%8E%9F%E5%88%99)
额头出汗原则（或称辛勤原则）是一条知识产权法律原则，尤其关系到著作权法。根据这条法律原则，作者通过创作（如数据库、通讯录）时所付出的劳动就可获得著作权。并不需要真正的创造或“原创性”。

### [JPM Coin 三部曲 (中) - 摩根大通为何青睐 Quorum 区块链?](https://www.8btc.com/article/375515)
#### POA共识的图解
PoA 基于一组有身份的节点，轮流进行记账。换句话说，每个节点在用自己的身份和权威作为担保。每个区块只需要一个签名确认，这大大提高了出块速度和每秒能够吞吐的交易。  
虽然有中心化的风险，不过 PoA 的设计中为了限制单个节点的权力，每个节点的签名间隔需要大于 N (总节点数)/2。  
#### RAFT
RAFT 其实是一种已经广为使用的传统分布式一致性协议，应用在包括 Kubernetes, Docker Swarm 等容器集群管理系统。RAFT 的节点分为 Leader 、Follower 以及暂时的 Candidate。

Leader 是负责生产区块的唯一节点，Follower 监听 Leader 的“心跳”，并收取 Leader 传递过来的区块。

如果 Follower 在其周期内没有收到 Leader 发来的心跳，则会认为Leader 已经死了。此时，没有收到 Leader心跳的 Follower 重新发起选举，自己的身份从 Follower 改变为 Candidate。它会给自己投一票，然后发送投票申请到其他 Follower，自己成为 Leader。
#### IBFT
IBFT，全称 Istanbul Byzantine Fault Tolerance (伊斯坦布尔拜占庭容错) 可以在抗分叉的基础上，防止部分节点作恶。  
IBFT 是一种实用拜占庭容错算法，与 RAFT 完全相信 Leader 不同，IBFT  的前提是包容 1/3 不诚实节点，通过验证者多轮投票，达到彼此一致后出块。
### [一文读懂椭圆曲线加密学](https://www.8btc.com/article/376027)
![](https://appserversrc.8btc.com/newpost/201903191333291.)
RSA:  公钥：944,871,836,856,449,473  私钥：961,748,941 and 982,451,653  
私钥是两个质数，公钥是两个质数的和。在真实的加密中，私钥需要200+位数以上的长度以确保安全。

### [十分钟读懂央视CCTV力荐书籍：《加密资产》](https://www.8btc.com/article/376342)
2019年2月25日算力为18TH/s的蚂蚁矿机售价4850元，比特币的总算力是43 330 000TH/s，因此攻击者想要控制比特币网络需要51%的算力，就需要至少花费59.54亿人民币（51%*4850*43330000/18）。

比特币和以太坊分别用59.54亿人民币和11.32亿人民币的成本，来维护价值高达4470亿和966亿人民币的加密货币网络。每1元的比特币的安全成本为0.013元。

### [国务院办公厅关于深入开展消费扶贫助力打赢脱贫攻坚战的指导意见](http://www.cpad.gov.cn/art/2019/1/15/art_1461_93105.html?from=timeline)
### 安全测试笔记
7.1.2  注册页面验证码不失效  
注册页面发送短信处虽然增加了验证码，但是验证码不失效，可重放发送短信请求进行短信轰炸（同时，“发送时间间隔过断”有错别字）  
 
### 国家标准
现行和即将实施食品技术类强制国家标准59条[1]((http://www.gb688.cn/bzgk/gb/std_list_type?r=0.4910481089509746&page=5&pageSize=10&p.p1=1&p)  
现行和即将实施的食品技术类推荐国家标准1293条[2](http://www.gb688.cn/bzgk/gb/std_list_type?r=0.20297146687837775&p.p1=2&p.p5=PUBLISHED|TOBEIMP&p.p6=67&p.p90=circulation_date&p.p91=desc)  
现行和即将实施的农业类强制国家标准170条[3](http://www.gb688.cn/bzgk/gb/std_list_type?r=0.19708214239649124&p.p1=1&p.p5=PUBLISHED|TOBEIMP&p.p6=65&p.p90=circulation_date&p.p91=desc)  
现行和即将实施的农业类推荐国家标准2496条[3](http://www.gb688.cn/bzgk/gb/std_list_type?r=0.3864398457813478&p.p1=2&p.p5=PUBLISHED|TOBEIMP&p.p6=65&p.p90=circulation_date&p.p91=desc)  

### [质量链专利填报](https://docs.qq.com/sheet/DSXl4bVRweGpJZ25k?newPad=1&newPadType=clone&tab=BB08J2)
### [天眼查检索专利](https://www.tianyancha.com/s/zhuanli)
检索后出现专利列表页面，点击“查看详情”按钮后进入专利详情页。在专利详情页的右上方，有的专利会显示一个PDF字样的下载按钮，点击可以下载PDF格式的专利详细文档。

天眼查上一些专利显示的信息不全，可以到国家知识产权局下的“[中国及多国专利审查信息查询](http://cpquery.sipo.gov.cn/txnIndex.do?purl=http%3A%2F%2Fcpquery.sipo.gov.cn%2FtxnPantentInfoList.do%3Finner-flag%3Aopen-type%3Dwindow%26inner-flag%3Aflowno%3D1551953509852)”根据专利名称查询。


### [二维码的生成细节和原理](https://www.cnblogs.com/alantu2018/p/8504373.html)
### [专利申请基础知识](https://wenku.baidu.com/view/8d7c1c6d2bf90242a8956bec0975f46527d3a7e2.html)
### [TrustData：2018年短视频行业发展简析](http://www.199it.com/archives/730075.html)
### 信通院白皮书
信通院发布的《大数据白皮书（2018年）》：http://www.cac.gov.cn/2018-04/25/c_1122741894.htm
信通院2018人工智能发展白皮书技术篇重磅发布：http://www.caict.ac.cn/kxyj/qwfb/bps/201809/P020180906443463663989.pdf
工信部发布《2018年中国区块链产业白皮书》：
http://www.miit.gov.cn/n1146290/n1146402/n1146445/c6180238/part/6180297.pdf
信通院发布《区块链白皮书（2018年）》：https://pan.baidu.com/s/1ncJg3SwjrOgQ8tlXT0cZww

### [区块链信息服务管理规定](http://www.cac.gov.cn/2019-01/10/c_1123971164.htm)
### [国家互联网信息办公室有关负责人就《区块链信息服务管理规定》答记者问](http://www.cac.gov.cn/2019-01/10/c_1123971223.htm) 
### [互联网新闻信息服务管理规定](http://www.cac.gov.cn/2017-05/02/c_1120902760.htm)

### [市场监管总局印发《假冒伪劣重点领域治理工作方案（2019-2021）》](https://mp.weixin.qq.com/s/eO7sfFwSaY1FElMQ6YQLqA)
### [详解Google Authenticator工作原理](https://www.csdn.net/article/2014-09-23/2821808-Google-Authenticator)
### 有赞、酷客多、微盟
https://baijiahao.baidu.com/s?id=1622058678825014428&wfr=spider&for=pc

### [Hyperledger Caliper 安装使用分析](https://blog.csdn.net/u013938484/article/details/80979810)
一个账本性能基准测试框架，它允许用户用预定义的用例来测试不同的账本解决方案，并得到一组性能测试结果。
这个基准框架的核心是一个能够翻译信息的“适配层”，Caliper能够安装智能合约，调用合约，并且查询各种分布式账本的状态，继而更好地评估其效力。

在可控环境内所支持的区块链上进行压力测试，并且生成相关结果，其中包括交易成功率、每秒交易次数、交易结算耗时、以及所有操作的资源消耗（比如CPU和内存）等，而且是大华为的。

### [微软、IBM、BAT争相发布BaaS，到底什么是区块链即服务？](http://www.sohu.com/a/233753212_429401)
这篇文章的BaaS和区块链描述可当作区块链话术使用。


### [1.今日头条 精准竞价广告价格](http://toutiao.appho.cn/kanli.html)
### [落地页](https://baike.baidu.com/item/%E8%90%BD%E5%9C%B0%E9%A1%B5/673016?fr=aladdin)
三秒原则:秒懂、秒信、秒杀  

### [区域零售连锁的有赞样本：广缘e购“变形计”！](http://finance.ifeng.com/a/20181204/16602869_0.shtml)

### [有赞赵强：有赞如何帮商家投广告？](http://finance.ifeng.com/a/20181206/16607190_0.shtml)
以往商家过于注重用户一次性ROI(投资回报率)，但是在获客成本提升的背景下，商家更应该关注用户的LTV(客户终生价值)，广告效果评估从ROI到LTV，是社交电商广告投放必须要转变的思维。

### [中国链湾白皮书](file:///E:/Disk/%E5%8C%BA%E5%9D%97%E9%93%BE/%E5%8F%82%E8%80%83/%E4%B8%AD%E5%9B%BD%E9%93%BE%E6%B9%BE%E7%99%BD%E7%9A%AE%E4%B9%A62017.pdf)
### [“小程序跳转小程序”功能调整](https://mp.weixin.qq.com/cgi-bin/announce?action=getannouncement&announce_id=11541056526eufNY&version=&lang=zh_CN&token=1246405614)
### [微商进化史](https://www.huxiu.com/article/256315.html)
### [易观：中国网上零售B2C市场年度综合分析2018(39页)（附下载链接）](http://www.3mbang.com/i-1081.html)

### [微信小店与第三方微信商城有何区别呢？](https://baijiahao.baidu.com/s?id=1595439184519878052&wfr=spider&for=pc)
微信小店是由微信官方开发，功能相对比较基础，无法满足个性化或深入的需求，无法定制开发。微信商城服务商众多，功能更加丰富，商家可随意挑选符合自己需求的商城，能满足商家各种个性化需求。  

### [雷军追捧的Costco模式究竟是什么？](http://www.woshipm.com/pmd/136625.html)
Costco（好市多）  
Jet更极致，只收会员费。每个Jet用户花费每年49.99美元，雇佣Jet为他们挑选最划算的商品  
1. 过去工业化时代是一个“人找货”的搜索逻辑中  
2. 经营用户，往往是一个货找人的推荐模型  

### [李叫兽：品牌之后，下一代的用户经营工具是什么？](https://mp.weixin.qq.com/s/LpIanuC3jQxMlYyMx77yzA)
从经营产品到经营用户的产品体系  
过去商品流通的一个重要特点是媒体和卖场的分离  Costco

### [云集，三年时间杀入电商第一梯队，凭什么？](https://baijiahao.baidu.com/s?id=1598640200546846432&wfr=spider&for=pc)
过去三年，有两家奇葩电商公司，都是通过社交模式交易规模跨过百亿，迅速挤入电商第一阵营，一个是众所周知的拼多多；一个就是湖畔大学四期学员肖尚略创办的云集。

### PC调试手机APP
用psiphon3.exe翻墙；  
手机用USB连接到pc，并开启开发者模式；


