title: 逆向直播盒子Green-iOS客户端
id: 1527694526218
author: 王磊磊
date: 2018-05-30 23:35:36
tags:
---
### ```写在前面的话```本次要使用的几个工具

- IDA
- AloneMonkey的MonkeyDev
- Charles
- Kali

## 什么是直播盒子？
- 单个直播的叫平台，比如斗鱼，熊猫，快手等等
- 所有的平台放在一个App里面就被称为盒子
- 现在上面上有很多直播平台（带颜色的）

## 什么是iOS逆向？
- 对我来说就是研究别人App里面的东西

### 为什么逆向这个盒子？
- 事情的起因是：帮兄弟的帮，兄弟让破解，所以研究下

### 首先上个图（市面上的直播盒子现在也有很多，图只是其中一种，兄弟发过来的链接）



- 首页

![](https://user-gold-cdn.xitu.io/2018/5/13/1635881c273f0d8e?w=1080&h=1921&f=jpeg&s=151446)

- 随便点击一个


![](https://user-gold-cdn.xitu.io/2018/5/13/163589cddf8f6005?w=1080&h=1921&f=jpeg&s=205853)

### ``` 您的会员账号已到期，请续费```付费是不可能付费的，这辈子都不会付费的。只能靠逆向才能维持生活这样子。。。

### 然后使用Chareles抓包 得到以下结果

![](https://user-gold-cdn.xitu.io/2018/5/13/163588e2d546a1c7?w=1060&h=472&f=jpeg&s=95367)

- 显然作者对数据进行了加密
- 看到了Host	api.appplat6688.com
- 看到域名那就扫描一下端口吧 !祭出Kali
```dos
 nmap api.appplat6688.com 
```
### 过了半根烟的时间
- 出现如下结果：

```
Starting Nmap 7.60 ( https://nmap.org ) at 2018-05-13 15:52 CST
Nmap scan report for api.appplat6688.com (101.55.26.69)
Host is up (0.64s latency).
Other addresses for api.appplat6688.com (not scanned): 220.95.210.101 101.55.26.70 182.16.53.100 216.118.239.124 52.128.230.228 180.178.48.220 103.90.137.107 216.118.239.132 220.95.210.78 182.16.55.76 180.178.51.212 119.42.148.148
Not shown: 983 closed ports
PORT     STATE    SERVICE
22/tcp   open     ssh
80/tcp   open     http
135/tcp  filtered msrpc
139/tcp  filtered netbios-ssn
443/tcp  open     https
445/tcp  filtered microsoft-ds
593/tcp  filtered http-rpc-epmap
901/tcp  filtered samba-swat
1068/tcp filtered instl_bootc
3128/tcp filtered squid-http
3333/tcp filtered dec-notes
4444/tcp filtered krb524
5800/tcp filtered vnc-http
5900/tcp filtered vnc
6129/tcp filtered unknown
6667/tcp filtered irc
6789/tcp open     ibm-db2-admin

Nmap done: 1 IP address (1 host up) scanned in 59.59 seconds
```
- 上面这些端口呢。基本上都是常用的。坦白的说我也搞不定它。所以先不管它(估计有人会问：既然搞不定，为什么要扫呢？因为人外有人，天外有天，我搞不了的不代表正在看文章的你搞不定。帮你扫的！)

   


### 因为我们的主题是逆向iOS客户端
- 如上图所示。此客户端进行了数据加密。一般数据加密的App说明开发者对自己的App做了保护。那么我们就要去看看他是怎么加密的

- 把件[IPA文件](http://)（经过3秒的思想斗争。最终决定还是不放链接了，想研究的加我V信：yuzhouheikewll）扔到IDA里面

### 半根烟时间后

* 全局搜索 ```您的会员账号已到期 ```


![](https://user-gold-cdn.xitu.io/2018/5/13/16358a59192725f8?w=1127&h=590&f=jpeg&s=228197)

* 最终结果


![](https://user-gold-cdn.xitu.io/2018/5/13/16358b1d1639ebef?w=766&h=480&f=jpeg&s=200207)

* 点击X按钮（一路X只到出现汇编代码）


![](https://user-gold-cdn.xitu.io/2018/5/13/16358b35dc318eba?w=785&h=509&f=jpeg&s=144456)

* 经过一些列分析，定位到如下代码


![](https://user-gold-cdn.xitu.io/2018/5/13/16358bbec90224b4?w=1408&h=634&f=jpeg&s=261904)


* hook ```KYLMQxqXCDsiemxz:params:success:failure:```
```
%hook GBoxNetManager

-(void)KYLMQxqXCDsiemxz:(id)arg1 params:(id) arg2 success:(id)arg3 failure:(id)arg4 {

%log;

NSLog(@"arg1%@", arg1);

NSLog(@"arg2%@", arg2);

NSLog(@"arg3%@", arg3);

NSLog(@"arg4%@", arg4);

%orig;

}

%end
```
* 得到以下结果

![](https://user-gold-cdn.xitu.io/2018/5/13/16358bf81220fee6?w=1118&h=192&f=jpeg&s=93896)

 - 那就看看这个函数的返汇编，和F5（IDA常用功能）出来的伪代码
 
![](https://user-gold-cdn.xitu.io/2018/5/13/16358c3251003981?w=1439&h=567&f=jpeg&s=160787)
* 根据我浅显的英文水准，判断出```GBoxNetCrypto```这个类就是加密类
* 那么我们就去看看这个类，然后Hook它
![](https://user-gold-cdn.xitu.io/2018/5/13/16358c5c39d67360?w=614&h=422&f=jpeg&s=94529)
* Hook代码
```
%hook GBoxNetCrypto

- (id) desEncrypt:(id)arg1 key:(id)arg2 {
	// %log;
	NSLog(@"desEncrypt arg1 = %@ arg2 = %@", arg1, arg2 );
	NSLog(@"desEncrypt===orig %@", %orig);

	return %orig;
}


- (id) desDecrypt:(id)arg1 key:(id)arg2 {
	// %log;
	NSLog(@"desDecrypt arg1 = %@ arg2 = %@", arg1, arg2 );
	NSLog(@"desDecrypt===orig %@", %orig);
	return %orig;
}
- (id) QVGRSpobWNqWYHVm:(id)arg1 key:(id)arg2 {
	// %log;
	NSLog(@"QVGRSpobWNqWYHVm：key arg1 = %@ arg2 = %@", arg1, arg2 );
	return %orig;
}

- (id) QVGRSpobWNqWYHVm:(id)arg1 {
	// %log;
	NSLog(@"QVGRSpobWNqWYHVm arg1 = %@", arg1 );
	return %orig;
}

- (id) dCkFSxbcvATgvDOF:(id)arg1 {
	// %log;
	NSLog(@"dCkFSxbcvATgvDOF arg1 = %@", arg1 );
	return %orig;
}


- (id) PxXAtABexHNGjGWc:(id)arg1 {
	// %log;
	NSLog(@"PxXAtABexHNGjGWc arg1 = %@", arg1 );
	return %orig;
}
%end

```
* 这里我没有去关注它内部的加密逻辑，只是拿到了加密的输出和输入（我们要的就是这个）


![](https://user-gold-cdn.xitu.io/2018/5/13/16358ca3f565aea1?w=1129&h=561&f=jpeg&s=304084)
 
 
![](https://user-gold-cdn.xitu.io/2018/5/13/16358d25ea9fe404?w=1137&h=392&f=jpeg&s=214454)



* 那么我们就去我们想要的界面去找想要的内容


![](https://user-gold-cdn.xitu.io/2018/5/13/16358d0c7aa22325?w=1080&h=1921&f=jpeg&s=171488)
 
### 看打印出的服务端返回内容
```json
{
	"code": 200,
	"list": [{
		"avatar": "http://lopk.oss-cn-shanghai.aliyuncs.com/public/attachment/201805/310598/1525874444233.png?x-oss-process=image/resize,m_mfit,h_200,w_200",
		"nickName": "MZZ颜宝",
		"roomId": "213909",
		"roomPay": 0,
		"url": "",
		"userId": 310598,
		"watchNum": 2774
	}, {
		"avatar": "http://lopk.oss-cn-shanghai.aliyuncs.com/public/attachment/201805/300719/1526197877705.png?x-oss-process=image/resize,m_mfit,h_200,w_200",
		"nickName": "红酒女神",
		"roomId": "213901",
		"roomPay": 0,
		"url": "",
		"userId": 300719,
		"watchNum": 2640
	}, {
		"avatar": "http://lopk.oss-cn-shanghai.aliyuncs.com/public/attachment/201805/343317/201805030457277987.png?x-oss-process=image/resize,m_mfit,h_200,w_200",
		"nickName": "U兔宝宝",
		"roomId": "213934",
		"roomPay": 0,
		"url": "",
		"userId": 343317,
		"watchNum": 2433
	}, {
		"avatar": "http://lopk.oss-cn-shanghai.aliyuncs.com/public/attachment/201805/258508/201805091218287030.png?x-oss-process=image/resize,m_mfit,h_200,w_200",
		"nickName": "Mzz人丑对不起祖国",
		"roomId": "213709",
		"roomPay": 0,
		"url": "",
		"userId": 258508,
		"watchNum": 8792
	}, {
		"avatar": "http://lopk.oss-cn-shanghai.aliyuncs.com/public/attachment/201805/222192/1525758635927.png?x-oss-process=image/resize,m_mfit,h_200,w_200",
		"nickName": "er秀秀女",
		"roomId": "213861",
		"roomPay": 0,
		"url": "",
		"userId": 222192,
		"watchNum": 3623
	}, {
		"avatar": "http://lopk.oss-cn-shanghai.aliyuncs.com/public/attachment/201805/273879/201805130501314734.png?x-oss-process=image/resize,m_mfit,h_200,w_200",
		"nickName": "妞妞Da姐姐",
		"roomId": "213955",
		"roomPay": 0,
		"url": "",
		"userId": 273879,
		"watchNum": 387
	}, {
		"avatar": "http://lopk.oss-cn-shanghai.aliyuncs.com/public/attachment/201805/288102/1526199691114.png?x-oss-process=image/resize,m_mfit,h_200,w_200",
		"nickName": "cK小淫妹",
		"roomId": "213923",
		"roomPay": 0,
		"url": "",
		"userId": 288102,
		"watchNum": 1407
	}, {
		"avatar": "http://lopk.oss-cn-shanghai.aliyuncs.com/public/attachment/201804/217817/1523589369535.png?x-oss-process=image/resize,m_mfit,h_200,w_200",
		"nickName": "MZZ小辣椒",
		"roomId": "213964",
		"roomPay": 0,
		"url": "",
		"userId": 217817,
		"watchNum": 359
	}, {
		"avatar": "http://lopk.oss-cn-shanghai.aliyuncs.com/public/attachment/201805/330326/1525259084018.png?x-oss-process=image/resize,m_mfit,h_200,w_200",
		"nickName": "FV俄国留学生",
		"roomId": "213875",
		"roomPay": 0,
		"url": "",
		"userId": 330326,
		"watchNum": 4004
	}, {
		"avatar": "http://lopk.oss-cn-shanghai.aliyuncs.com/public/attachment/201805/283380/1526202846359.png?x-oss-process=image/resize,m_mfit,h_200,w_200",
		"nickName": "GG新葡京官方认证推筒子",
		"roomId": "213969",
		"roomPay": 0,
		"url": "",
		"userId": 283380,
		"watchNum": 315
	}, {
		"avatar": "http://lopk.oss-cn-shanghai.aliyuncs.com/public/attachment/201805/351343/1526190381915.png?x-oss-process=image/resize,m_mfit,h_200,w_200",
		"nickName": "GG魔图精灵",
		"roomId": "213787",
		"roomPay": 0,
		"url": "",
		"userId": 351343,
		"watchNum": 437
	}, {
		"avatar": "http://lopk.oss-cn-shanghai.aliyuncs.com/public/attachment/201804/225589/201804151030587590.png?x-oss-process=image/resize,m_mfit,h_200,w_200",
		"nickName": "燕子",
		"roomId": "213929",
		"roomPay": 0,
		"url": "",
		"userId": 225589,
		"watchNum": 912
	}, {
		"avatar": "http://lopk.oss-cn-shanghai.aliyuncs.com/public/attachment/201805/383599/1526185673046.png?x-oss-process=image/resize,m_mfit,h_200,w_200",
		"nickName": "Q户外运动2",
		"roomId": "213897",
		"roomPay": 0,
		"url": "",
		"userId": 383599,
		"watchNum": 2
	}, {
		"avatar": "http://lopk.oss-cn-shanghai.aliyuncs.com/public/attachment/201805/341027/1526190449533.png?x-oss-process=image/resize,m_mfit,h_200,w_200",
		"nickName": "MZZ我叫然儿",
		"roomId": "213790",
		"roomPay": 0,
		"url": "",
		"userId": 341027,
		"watchNum": 3139
	}, {
		"avatar": "http://lopk.oss-cn-shanghai.aliyuncs.com/public/attachment/201804/218746/1523616631878.png?x-oss-process=image/resize,m_mfit,h_200,w_200",
		"nickName": "苏苏",
		"roomId": "213624",
		"roomPay": 0,
		"url": "",
		"userId": 218746,
		"watchNum": 4265
	}, {
		"avatar": "http://lopk.oss-cn-shanghai.aliyuncs.com/public/attachment/201805/381140/1526200185989.png?x-oss-process=image/resize,m_mfit,h_200,w_200",
		"nickName": "GG赌皇国际娱乐会所",
		"roomId": "213936",
		"roomPay": 0,
		"url": "",
		"userId": 381140,
		"watchNum": 53
	}, {
		"avatar": "http://lopk.oss-cn-shanghai.aliyuncs.com/public/attachment/201805/235006/1526194826473.png?x-oss-process=image/resize,m_mfit,h_200,w_200",
		"nickName": "GG娱乐在线",
		"roomId": "213865",
		"roomPay": 0,
		"url": "",
		"userId": 235006,
		"watchNum": 70
	}, {
		"avatar": "http://lopk.oss-cn-shanghai.aliyuncs.com/public/attachment/201805/335223/1525414300646.png?x-oss-process=image/resize,m_mfit,h_200,w_200",
		"nickName": "SR-甜甜蜜蜜",
		"roomId": "213949",
		"roomPay": 0,
		"url": "",
		"userId": 335223,
		"watchNum": 143
	}, {
		"avatar": "http://lopk.oss-cn-shanghai.aliyuncs.com/public/attachment/201805/365481/1526195868109.png?x-oss-process=image/resize,m_mfit,h_200,w_200",
		"nickName": "GG娱乐一筒天下",
		"roomId": "213881",
		"roomPay": 0,
		"url": "",
		"userId": 365481,
		"watchNum": 408
	}, {
		"avatar": "http://lopk.oss-cn-shanghai.aliyuncs.com/public/attachment/201805/283355/1526007090245.png?x-oss-process=image/resize,m_mfit,h_200,w_200",
		"nickName": "Q闺蜜老公",
		"roomId": "213721",
		"roomPay": 0,
		"url": "",
		"userId": 283355,
		"watchNum": 949
	}, {
		"avatar": "http://lopk.oss-cn-shanghai.aliyuncs.com/public/attachment/201805/238973/1526150049307.png?x-oss-process=image/resize,m_mfit,h_200,w_200",
		"nickName": "湿妹",
		"roomId": "213956",
		"roomPay": 0,
		"url": "",
		"userId": 238973,
		"watchNum": 487
	}, {
		"avatar": "http://lopk.oss-cn-shanghai.aliyuncs.com/public/attachment/201805/217200/1526201580870.png?x-oss-process=image/resize,m_mfit,h_200,w_200",
		"nickName": "霸道小浪妹",
		"roomId": "213948",
		"roomPay": 0,
		"url": "",
		"userId": 217200,
		"watchNum": 714
	}, {
		"avatar": "http://lopk.oss-cn-shanghai.aliyuncs.com/public/attachment/201804/313511/1524838889797.png?x-oss-process=image/resize,m_mfit,h_200,w_200",
		"nickName": "sr魅魔",
		"roomId": "213595",
		"roomPay": 0,
		"url": "",
		"userId": 313511,
		"watchNum": 2845
	}, {
		"avatar": "http://lopk.oss-cn-shanghai.aliyuncs.com/public/attachment/201805/384935/1526185367686.png?x-oss-process=image/resize,m_mfit,h_200,w_200",
		"nickName": "GG名流娱乐会所",
		"roomId": "213726",
		"roomPay": 0,
		"url": "",
		"userId": 384935,
		"watchNum": 5325
	}, {
		"avatar": "http://lopk.oss-cn-shanghai.aliyuncs.com/public/attachment/201805/329991/201805011847014858.png?x-oss-process=image/resize,m_mfit,h_200,w_200",
		"nickName": "MI芊",
		"roomId": "213889",
		"roomPay": 0,
		"url": "",
		"userId": 329991,
		"watchNum": 1315
	}, {
		"avatar": "http://lopk.oss-cn-shanghai.aliyuncs.com/public/attachment/201805/312549/201805010204564037.png?x-oss-process=image/resize,m_mfit,h_200,w_200",
		"nickName": "JL雨",
		"roomId": "213907",
		"roomPay": 0,
		"url": "",
		"userId": 312549,
		"watchNum": 1899
	}, {
		"avatar": "http://lopk.oss-cn-shanghai.aliyuncs.com/public/attachment/201805/262886/1525618035798.png?x-oss-process=image/resize,m_mfit,h_200,w_200",
		"nickName": "Dz妖孽人生",
		"roomId": "213821",
		"roomPay": 0,
		"url": "",
		"userId": 262886,
		"watchNum": 2608
	}, {
		"avatar": "http://lopk.oss-cn-shanghai.aliyuncs.com/public/attachment/201805/223279/1526201868037.png?x-oss-process=image/resize,m_mfit,h_200,w_200",
		"nickName": "至尊大湿妹",
		"roomId": "213951",
		"roomPay": 0,
		"url": "",
		"userId": 223279,
		"watchNum": 1055
	}, {
		"avatar": "http://lopk.oss-cn-shanghai.aliyuncs.com/public/attachment/201805/264781/1526201888712.png?x-oss-process=image/resize,m_mfit,h_200,w_200",
		"nickName": "小小姐",
		"roomId": "213953",
		"roomPay": 0,
		"url": "",
		"userId": 264781,
		"watchNum": 954
	}, {
		"avatar": "http://lopk.oss-cn-shanghai.aliyuncs.com/public/attachment/201805/360406/201805071210131372.png?x-oss-process=image/resize,m_mfit,h_200,w_200",
		"nickName": "霸道宝贝",
		"roomId": "213940",
		"roomPay": 0,
		"url": "",
		"userId": 360406,
		"watchNum": 681
	}, {
		"avatar": "http://lopk.oss-cn-shanghai.aliyuncs.com/public/attachment/201805/354808/1525544975204.png?x-oss-process=image/resize,m_mfit,h_200,w_200",
		"nickName": "Q妖姬",
		"roomId": "213789",
		"roomPay": 0,
		"url": "",
		"userId": 354808,
		"watchNum": 1143
	}, {
		"avatar": "http://lopk.oss-cn-shanghai.aliyuncs.com/public/attachment/201805/377175/1526170882681.png?x-oss-process=image/resize,m_mfit,h_200,w_200",
		"nickName": "SG闭月羞花2",
		"roomId": "213567",
		"roomPay": 0,
		"url": "",
		"userId": 377175,
		"watchNum": 2508
	}, {
		"avatar": "http://lopk.oss-cn-shanghai.aliyuncs.com/public/attachment/201805/272572/1525795865574.png?x-oss-process=image/resize,m_mfit,h_200,w_200",
		"nickName": "霸道全国跪求约泡可怜求爱爱",
		"roomId": "213950",
		"roomPay": 0,
		"url": "",
		"userId": 272572,
		"watchNum": 45
	}, {
		"avatar": "http://lopk.oss-cn-shanghai.aliyuncs.com/public/attachment/201805/224784/1525323978804.png?x-oss-process=image/resize,m_mfit,h_200,w_200",
		"nickName": "MZZ甜心可可",
		"roomId": "213900",
		"roomPay": 0,
		"url": "",
		"userId": 224784,
		"watchNum": 1824
	}, {
		"avatar": "http://lopk.oss-cn-shanghai.aliyuncs.com/public/attachment/201805/328016/201805130829280235.png?x-oss-process=image/resize,m_mfit,h_200,w_200",
		"nickName": "霸道椰子壳",
		"roomId": "213570",
		"roomPay": 0,
		"url": "",
		"userId": 328016,
		"watchNum": 1067
	}, {
		"avatar": "http://lopk.oss-cn-shanghai.aliyuncs.com/public/attachment/201804/283909/1524472684340.png?x-oss-process=image/resize,m_mfit,h_200,w_200",
		"nickName": "283909丢丢",
		"roomId": "213702",
		"roomPay": 0,
		"url": "",
		"userId": 283909,
		"watchNum": 5689
	}, {
		"avatar": "http://lopk.oss-cn-shanghai.aliyuncs.com/public/attachment/201805/383651/1526101482856.png?x-oss-process=image/resize,m_mfit,h_200,w_200",
		"nickName": "GG诚信走天下B",
		"roomId": "213944",
		"roomPay": 0,
		"url": "",
		"userId": 383651,
		"watchNum": 24
	}, {
		"avatar": "http://lopk.oss-cn-shanghai.aliyuncs.com/public/attachment/201805/374105/1525901432016.png?x-oss-process=image/resize,m_mfit,h_200,w_200",
		"nickName": "新人求礼物",
		"roomId": "213961",
		"roomPay": 0,
		"url": "",
		"userId": 374105,
		"watchNum": 456
	}, {
		"avatar": "http://lopk.oss-cn-shanghai.aliyuncs.com/public/attachment/201805/378153/1526181757820.png?x-oss-process=image/resize,m_mfit,h_200,w_200",
		"nickName": "霸道淫乱",
		"roomId": "213674",
		"roomPay": 0,
		"url": "",
		"userId": 378153,
		"watchNum": 742
	}, {
		"avatar": "http://lopk.oss-cn-shanghai.aliyuncs.com/public/attachment/201805/387256/201805130509055450.png?x-oss-process=image/resize,m_mfit,h_200,w_200",
		"nickName": "xv小妖精",
		"roomId": "213965",
		"roomPay": 0,
		"url": "",
		"userId": 387256,
		"watchNum": 34
	}, {
		"avatar": "http://lopk.oss-cn-shanghai.aliyuncs.com/public/attachment/201805/224084/1526197795009.png?x-oss-process=image/resize,m_mfit,h_200,w_200",
		"nickName": "nL小姐姐",
		"roomId": "213899",
		"roomPay": 0,
		"url": "",
		"userId": 224084,
		"watchNum": 3
	}, {
		"avatar": "http://lopk.oss-cn-shanghai.aliyuncs.com/public/attachment/201805/338856/1525672991470.png?x-oss-process=image/resize,m_mfit,h_200,w_200",
		"nickName": "霸道左小雨",
		"roomId": "213921",
		"roomPay": 0,
		"url": "",
		"userId": 338856,
		"watchNum": 3
	}, {
		"avatar": "http://lopk.oss-cn-shanghai.aliyuncs.com/public/attachment/201804/263866/1525086433855.png?x-oss-process=image/resize,m_mfit,h_200,w_200",
		"nickName": "醉红颜凡凡",
		"roomId": "213960",
		"roomPay": 0,
		"url": "",
		"userId": 263866,
		"watchNum": 303
	}, {
		"avatar": "http://lopk.oss-cn-shanghai.aliyuncs.com/public/attachment/201805/298724/201805130427082900.png?x-oss-process=image/resize,m_mfit,h_200,w_200",
		"nickName": "GG愤怒的蚊子",
		"roomId": "213933",
		"roomPay": 0,
		"url": "",
		"userId": 298724,
		"watchNum": 914
	}, {
		"avatar": "http://lopk.oss-cn-shanghai.aliyuncs.com/public/attachment/201805/316941/201805131623228255.png?x-oss-process=image/resize,m_mfit,h_200,w_200",
		"nickName": "妞妞可爱丽嘚嘚",
		"roomId": "213928",
		"roomPay": 0,
		"url": "",
		"userId": 316941,
		"watchNum": 1566
	}, {
		"avatar": "http://lopk.oss-cn-shanghai.aliyuncs.com/public/attachment/201805/379899/1526202488299.png?x-oss-process=image/resize,m_mfit,h_200,w_200",
		"nickName": "ZJ夢無痕",
		"roomId": "213963",
		"roomPay": 0,
		"url": "",
		"userId": 379899,
		"watchNum": 251
	}, {
		"avatar": "http://lopk.oss-cn-shanghai.aliyuncs.com/public/attachment/201805/277556/1526197660901.png?x-oss-process=image/resize,m_mfit,h_200,w_200",
		"nickName": "Oh舒服",
		"roomId": "213959",
		"roomPay": 0,
		"url": "",
		"userId": 277556,
		"watchNum": 2
	}, {
		"avatar": "http://lopk.oss-cn-shanghai.aliyuncs.com/public/attachment/201805/365205/1525763881629.png?x-oss-process=image/resize,m_mfit,h_200,w_200",
		"nickName": "GG东东老虎鸡哦",
		"roomId": "213958",
		"roomPay": 0,
		"url": "",
		"userId": 365205,
		"watchNum": 394
	}, {
		"avatar": "http://lopk.oss-cn-shanghai.aliyuncs.com/public/attachment/201805/379459/1526135486226.png?x-oss-process=image/resize,m_mfit,h_200,w_200",
		"nickName": "LY多多呀",
		"roomId": "213970",
		"roomPay": 0,
		"url": "",
		"userId": 379459,
		"watchNum": 63
	}, {
		"avatar": "http://lopk.oss-cn-shanghai.aliyuncs.com/public/attachment/201805/356966/1525596024623.png?x-oss-process=image/resize,m_mfit,h_200,w_200",
		"nickName": "GG公平推筒子",
		"roomId": "213914",
		"roomPay": 0,
		"url": "",
		"userId": 356966,
		"watchNum": 17
	}, {
		"avatar": "http://lopk.oss-cn-shanghai.aliyuncs.com/public/attachment/201805/370521/201805130344003097.png?x-oss-process=image/resize,m_mfit,h_200,w_200",
		"nickName": "GG你好明天吧",
		"roomId": "213893",
		"roomPay": 0,
		"url": "",
		"userId": 370521,
		"watchNum": 65
	}, {
		"avatar": "http://lopk.oss-cn-shanghai.aliyuncs.com/public/attachment/201805/327725/201805130242323786.png?x-oss-process=image/resize,m_mfit,h_200,w_200",
		"nickName": "GG童颜大波",
		"roomId": "213851",
		"roomPay": 0,
		"url": "",
		"userId": 327725,
		"watchNum": 945
	}, {
		"avatar": "http://lopk.oss-cn-shanghai.aliyuncs.com/public/attachment/201805/245238/1526201838262.png?x-oss-process=image/resize,m_mfit,h_200,w_200",
		"nickName": "GG依",
		"roomId": "213952",
		"roomPay": 0,
		"url": "",
		"userId": 245238,
		"watchNum": 5
	}, {
		"avatar": "http://lopk.oss-cn-shanghai.aliyuncs.com/public/attachment/201805/384925/1526189955136.png?x-oss-process=image/resize,m_mfit,h_200,w_200",
		"nickName": "GG辉煌娱乐",
		"roomId": "213782",
		"roomPay": 0,
		"url": "",
		"userId": 384925,
		"watchNum": 297
	}, {
		"avatar": "http://lopk.oss-cn-shanghai.aliyuncs.com/public/attachment/201805/365824/1526173626609.png?x-oss-process=image/resize,m_mfit,h_200,w_200",
		"nickName": "GGYYA诚信天下",
		"roomId": "213592",
		"roomPay": 0,
		"url": "",
		"userId": 365824,
		"watchNum": 8458
	}],
	"accountConfig": "{\"sdkAppId\":\"1400081396\",\"accountType\":\"24916\",\"IMType\":\"1\",\"webSdkAppId\":\"1106161652\"}"
}
```
### 其实正常其他盒子到这一步，就已经可以拿到直播url了，但是这家的盒子做的比较严谨！url居然返回空！散了吧！到此为止！这个盒子我搞不了

### 最后补上一个成功的文章[http://bbs.iosre.com/t/mt-box-ios/11907](http://bbs.iosre.com/t/mt-box-ios/11907)