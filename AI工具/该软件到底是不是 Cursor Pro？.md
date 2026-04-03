> *本文仅作为技术探讨，逆向工作大多数由 Codex 完成，不具备专业性；本文不针对任何实体商家、没有透露任何商家姓名，文中插件完全由自己开发，仅供娱乐。*

> 本文在 [逆向淘宝cursor销量第一商户：AI商家赚钱到底有多轻松？ – sora's blog](https://sorablog.me/%e9%80%86%e5%90%91%e6%b7%98%e5%ae%9dcursor%e9%94%80%e9%87%8f%e7%ac%ac%e4%b8%80%e5%95%86%e6%88%b7%ef%bc%9aai%e5%95%86%e5%ae%b6%e8%b5%9a%e9%92%b1%e5%88%b0%e5%ba%95%e6%9c%89%e5%a4%9a%e8%bd%bb%e6%9d%be/) 上阅读最佳。


今年1月，我发现我的同学全部都在使用 Cursor，并且随意开启了 Claude 4.5 opus 以及 Max 模式进行使用。我询问了他们，他们说淘宝与闲鱼存在大量的 30 元包月 Cursor Pro 套餐。

虽然我知道这类套餐大概率是掺水或者不真实的，站内也有提醒，但我还是想知道这类套餐的原理和购买数量，因此我打开了淘宝，并下单了一个1天的pro套餐进行使用，此商家就是在淘宝信息流中推荐第一位的，本期的主角，xx旗舰店。
![image|252x500, 100%](https://cdn3.linux.do/original/4X/6/6/d/66d6c879408019d820a11f3e94be57e60d9e8330.jpeg)

其插件如下所示，大家感兴趣的话可以下载试试看：
[cursorpool-1.0.48.vsix.zip|attachment](upload://z5XZdjaBACNd6MNsywIkx7saXf2.zip) (855.5 KB)
逆向的结果如下：
[逆向visx.zip|attachment](upload://pZ5Evcp6tuzODicVADYQSVGD8vE.zip) (3.8 MB)
简单说下该插件的实现原理。
## 该软件到底是不是 Cursor Pro？
肯定不是 Pro，也不可能这么便宜，所谓的 Claude Opus 模型，我觉得甚至根本都不用测，连 Cursor 的 Composer 模型我觉得都是假的。

不过为了严谨性，我们还是用日文小说法和引号法测试一下好了：
![image|555x500, 75%](https://cdn3.linux.do/original/4X/3/3/c/33c712aad3f2a78fd8a55f70596e95ccdcb72886.jpeg)
不论是用乱码还是引号，很显然都不符合 Claude 与 Opus 模式的最基本特征；

那么它是怎么做到伪装的呢？它是通过本地篡改 Cursor 客户端，配合第三方网关来伪装成 Pro 的能力。具体操作是：将插件装入 Cursor 并激活后，它会重启 Cursor 并篡改本地客户端的表现方式。

其篡改的主要有两点：
1. 将各类模型的选择劫持到自己的代理上，注入 window.CODEX_URL / window.CODEX_TOKEN 、并 hook 上报逻辑，从而影响 Cursor 内部后续向模型端发起请求的目标地址，具体实现在该文件内：
![image|690x319](https://cdn3.linux.do/original/4X/4/4/c/44c113e0ec369c02be06cc149549f1dbb8c3b33c.jpeg)
![image|483x500, 50%](https://cdn3.linux.do/original/4X/5/1/2/5122ab107f3ceb05b01af73f5826b56a6ad5fd04.jpeg)

2. 在设置页面堂而皇之地印上“Pro”字样，面板触发的登录/激活/换号等动作，会调用自行的服务端 API，以检查在某个网站上的 付费 状态。
![image|690x264](https://cdn3.linux.do/original/4X/f/8/b/f8b6cd826441c0831aa3b2f903b970c3d96f91a6.jpeg)
![image|690x361, 75%](https://cdn3.linux.do/original/4X/8/9/1/891bcad7efe595d75566739990c55f0d55f60c51.jpeg)

也就是说，该 Pro 不是 Cursor 官方的 Pro，而是他们商家的 Pro。你开通的并非是 Cursor 的 Pro 账号，而是商家的 Pro 账号，和 Cursor 官方没有半毛钱关系。而 Cursor 本身还明晃晃地写着 Upgrade to Pro 字样...

### 模型怎么绕过 Cursor 进行转发的？
具体实现如下图代码所示。
![image|690x420](https://cdn3.linux.do/original/4X/d/7/a/d7a9bf1d539adb5cbb24be9836edee1209ab96a6.jpeg)
插件先把 Cursor 自身的代码打补丁，让 Cursor 运行时使用被注入的 CODEX_URL 与 CODEX_TOKEN。

默认会指向一个奇怪的 URL，大家在图中可以看到。

远程 worker 拉取也是一个奇怪的域名，在下面的代码中有所展示：
![image|690x182](https://cdn3.linux.do/original/4X/0/a/3/0a39f66ba47c987c7689bf558e11bb2ea0413096.jpeg)

总而言之，这肯定不是一个老老实实提供官方 Cursor Pro 账号的插件，而是一个劫持 Cursor 请求，使用不知道是什么（也许是大量免费廉价 API）的模型。

您以为这就结束了？

没有，精彩的地方才在后面。**让我们访问一下这些网址都是什么，您就会发现，淘宝与闲鱼的商家赚钱到底到底到底有多容易，容易到只要一个人拥有一部手机、一个淘宝商户账号，就能够开启，并无限制地贩卖这些内容，甚至不需要插件开发的成本，不需要任何服务器，也不需要任何 API 的费用。**

首先打开 某官网 ，也就是在逆向代码中提到过的网址。

该网址是一个售卖 Cursor 激活插件的网址：
![image|690x367](https://cdn3.linux.do/original/4X/b/9/0/b90f927a013b5ecf3df6b41a6666ef8ca99060f2.jpeg)
但是大家不要去怀疑这个网址的商家，**因为该网址的商家和淘宝商家不同，它可能才是老老实实破解 Cursor 并且提供 API 的那一个。**

我们接着往下看，点开这个 Cursor 激活器的售卖页面，正好是上面提到的 microsoft.icu 的某个子网站。
#### 我们有一个（仅仅是）推测：该淘宝上的商家甚至没有研究过插件，也没有使用服务器购买 API 或者 Cursor Pro 账号，仅仅是通过上游渠道进行批量进货，并卖给下游；中间只需要一个淘宝账号，就能源源不断地生钱。到底是不是这样呢？我们看一下它在淘宝上的购买价格：
![7be9f8727feeca57055f6529ff1b563d|370x500, 75%](https://cdn3.linux.do/original/4X/3/0/b/30b248f5e7d8bb46b50a569694ce199879cdab10.jpeg)
啊哈！上游卖 2 元一天的一天独享 Cursor Pro，到了下游就变成了 8 块；上游一周 9.9 元的价格，到了下游就是 26 元，翻了不止一倍。

那么有没有进一步的证据证明这两者之间存在关联呢？
有的，而且这两者之间关系很紧密，请看下图：
左图是该网址商家提供的源文档，右图是淘宝商家提供的文档，两者几乎可以说是一模一样：
![image|690x246](https://cdn3.linux.do/original/4X/b/f/2/bf24bce364700808294aa649e68721a2e6377cbf.jpeg)
也就是说，某商家可能连文档都懒得写，就是原封不动地搬运过来；
那么有没有可能这个网站的作者就是这个淘宝商家呢？

依我看来，这并不可能，一是他没有必要提供两份一模一样的文档，二是也没有必要在官网上卖一个价格，在淘宝上又卖另一个价格。

除此之外，该网站也就是插件作者本人，已经在文档下面进行了澄清。
![image|690x113](https://cdn3.linux.do/original/4X/8/c/8/8c81547bffcc6866409c0bd693ec2b94009003b1.jpeg)

### 总结
也就是说，存在这样一种情况：某个人仅仅只有一台手机，并注册了淘宝商家账号。通过“左手倒右手”的形式，在没有购买任何服务器、甚至连 API 、文档都是直接照搬过来的情况下，就能轻轻松松通过倒卖，冲上淘宝搜索第一的位置，月销成千甚至上万；而这样的商家在淘宝中并不占少数。
![image|252x500](https://cdn3.linux.do/original/4X/5/8/4/584c32948118af9241f72e8893dcf89ccfe695a3.jpeg)

我曾经听过一句话，就是道德感太高的人，赚不到什么钱。现在我相信这句话是对的。
L 站有数不尽的资源。如果佬友们愿意的话，那么想要赚到与月薪持平的钱，是一件非常容易的事情，是道德感在告诉我们不能这么去做，至少这件事情给我提供了两个思路：

第一个是绕过 Cursor 使用自己的模型，即使是 Free 账号也是有可能的；
第二个是赚钱的路子很多，远不止工资这一种——至少今天这种就让我大开眼界。