---
layout:		post
title:		Chapter 7 Attacking Session Management(3)-Weakness in Session Token Handling(1)
subtitle:	
date:		2020-03-13
author:		D
header-img:
catalog:	true
mermaid:	true
tags: [web hacking]:
---

参考: *The Web Application Hacker's Handbook* Chapter 7

无论应用程序如何有效地确保它生成的会话令牌不包含任何有意义的信息,而且不受分析或预测的影响,如果这些令牌在以后没有仔细处理,它的会话机制将会受到广泛的攻击。例如,如果以某些方式向攻击者披露令牌,攻击者可以劫持用户会话,即使预测令牌是不可能的。

应用程序不安全处理令牌可以使其容易受到多种方式攻击。

# 1.在网络上披露标记(Disclosure of Tokens on the Network)

当会话令牌通过未加密的表单传输到网络中时,就会出现这个漏洞区域,使一个合适的监听者可以获得令牌,因此伪装为合法用户。在用户的IT部门内,在用户的IT部门内,在用户的IT部门内,在用户的网络主干中,在应用程序的ISP内,以及在该应用程序的IT部门内,包括用户的IT部门。在每一种情况下,这包括有关组织的授权人员和任何损害基础设施的外部攻击者。

当会话令牌通过未加密的表单传输到网络中时,就会出现这个漏洞区域,使一个合适的监听者可以获得令牌,因此伪装为合法用户。在用户的IT部门内,在用户的IT部门内,在用户的IT部门内,在用户的网络主干中,在应用程序的ISP内,以及在该应用程序的IT部门内,包括用户的IT部门。在每一种情况下,这包括有关组织的授权人员和任何损害基础设施的外部攻击者。

在其他情况下,应用程序可以使用HTTPS来保护关键的客户机-服务器通信,但可能仍然很容易在网络上拦截会话标记。这种弱点可能以各种方式出现,其中许多方法可以在HTTP cookie被用作会话标记的传输机制时出现。
- 1.在登录过程中,一些应用程序选择使用HTTPS来保护用户的凭据,然后在用户会话的其余部分恢复到HTTP。许多web邮件应用程序以这种方式运行。在这种情况下,窃听器不能拦截用户的凭据,但可能仍然捕获会话标记。作为Firefox的插件,Firesheep工具使它成为一个简单的过程。
- 2.一些应用程序使用HTTP用于站点的预先验证区域,比如站点的首页,但从登录页面切换到HTTPS。然而,在许多情况下,用户在访问的第一个页面上发布了一个会话令牌,当用户登录时,这一标记并没有被修改。用户的会话,最初是未经身份验证的,在登录后升级为经过身份验证的会话。在这种情况下,窃听者可以在登录前拦截用户的令牌,等待用户的通信切换到HTTPS,指示用户正在登录,然后尝试使用该令牌访问受保护的页面(如我的帐户)。
- 3.即使应用程序在成功登录后发布了一个新的令牌,并且从登录页面上使用HTTPS,用户身份验证会话的令牌可能还会被披露。如果用户重新访问一个预先认证的页面(例如帮助或有关),要么通过在经过身份验证的区域内的链接,通过使用后按钮,或者直接输入URL,就会发生这种情况。
- 4.在前面的案例中,应用程序可能会尝试在用户单击登录链接时切换到HTTPS。然而,如果用户修改了URL,它仍然可以接受HTTP登录。在这种情况下,适当定位的攻击者可以修改站点前验证区域返回的页面,以便登录链接指向HTTP页面。即使应用程序在成功登录后发布了一个新的会话令牌,攻击者仍然可以拦截这个令牌,如果他成功地将用户的连接降级到HTTP。
- 5.有些应用程序使用HTTP中的所有静态内容,比如图像、脚本、样式表和页面模板。这种行为通常在用户浏览器中发出警告,如图7-9所示。当浏览器显示这个警告时,它已经在HTTP上检索了相关的项,因此会话令牌已经传输。浏览器警告的目的是让用户拒绝处理已经收到的HTTP的响应数据,因此可能会受到污染。如前所述,攻击者可以在用户的浏览器访问HTTP上的资源时拦截用户的会话标记,并使用这个令牌来访问受保护的、非静态区域的HTTPS。

![figure7-9](/img/web_hacking/twahh/figure7-9.jpg)

- 6.即使应用程序为每个页面使用HTTPS,包括站点和静态内容的未经身份验证的区域,也可能存在用户的令牌通过HTTP传输的环境。如果攻击者可以以某种方式诱导用户在HTTP上发出请求(如果一个服务器运行或访问到http://server:443/otherwise),则可以提交它。攻击者可以尝试的方法包括在电子邮件或即时信息中发送用户一个URL,在攻击者控制的网站上安装自动链接,或者使用可点击的横幅广告。(参见第12章和第13章,了解更多关于如何攻击其他用户的技术。)

**HACK STEPS**<br>
- 1.从第一次访问(“启动”URL)、通过登录过程,然后通过所有应用程序的功能访问应用程序。保存每个URL访问的记录,并注意一个新的会话令牌被接收的每个实例。特别注意HTTP和HTTPS通信之间的登录函数和转换。这可以通过使用网络嗅探器(如Wireshark)或部分自动使用您的拦截代理的日志功能进行手动实现,如图7-10所示。

![figure7-10](/img/web_hacking/twahh/figure7-10.jpg)

- 2.如果HTTP cookie被用作会话令牌的传输机制,则验证安全标志是否设置,防止它们通过未加密的连接传输。
- 3.确定在应用程序的正常使用中,会话令牌是否会通过未加密的连接传输。如果是这样的话,他们应该被认为很容易被拦截。
- 4.开始页面使用HTTP,应用程序切换到站点的登录和身份验证区域的HTTPS,验证新令牌是否在登录后发布,或者在HTTP阶段传递的标记是否仍然被用来跟踪用户的身份验证会话。另外,如果登录URL被相应地修改,应用程序是否会接受HTTP登录。
- 5.即使应用程序为每一页使用HTTPS,验证服务器是否也在监听端口80,运行任何服务或内容。如果是这样的话,请直接从身份验证的会话中访问任何HTTP URL,并验证会话令牌是否被传输。
- 6.在通过HTTP传输到服务器的身份验证会话的情况下,验证该标记是否继续有效,或者由服务器立即终止。

# 2.在日志中披露令牌(Disclosure of Tokens in Logs)

除了在网络通信中,会话令牌的明文传输之外,最常见的标记是在各种类型的系统日志中公开的。虽然这是一种罕见的发生,但这种披露的后果通常更严重。这些日志可能会被更广泛的潜在攻击者观看,而不仅仅是一个合适的位置来窃听网络的人。

许多应用程序为管理员和其他支持人员提供了功能,以监视应用程序运行时状态的各个方面,包括用户会话。例如,帮助工作人员帮助有问题的用户可能会询问她的用户名,通过列表或搜索函数定位她的当前会话,并查看有关会话的相关细节。或者管理员可以在调查安全漏洞的过程中查阅最近的一段记录。通常,这种监视和控制功能公开了与每个会话关联的实际会话令牌。通常,功能不受保护,允许未经授权的用户访问当前会话令牌的列表,从而劫持所有应用程序用户的会话。

在系统日志中出现的会话令牌的另一个主要原因是应用程序使用URL查询字符串作为传输令牌的机制,而不是使用HTTP cookie或POST请求的主体。例如,Googling `inurl:jsessionid` 标识成千上万的应用程序将Java平台会话令牌(称为`jsessionid`)传输到URL中:
```
http://www.webjunction.org/do/Navigation;jsessionid=
F27ED2A6AAE4C6DA409A3044E79B8B48?category=327
```
当应用程序以这种方式传输它们的会话标记时,可能会出现在不同的系统日志中,而未经授权的一方可能会有访问:
- 用户浏览器日志
- Web服务器日志
- 公司或ISP代理服务器的日志
- 在应用程序的托管环境中使用的任何反向代理的日志
- 应用程序用户通过离线链接访问的任何服务器的引用日志,如图7-11所示

即使在应用程序中使用HTTPS,有些漏洞也会出现。

![figure7-11](/img/web_hacking/twahh/figure7-11.jpg)

在一些应用程序中,仅仅描述了一个攻击者在捕获会话标记的有效方法时出现了一个攻击者。例如,如果一个web邮件应用程序在URL中传输会话标记,那么攻击者就可以向应用程序的用户发送电子邮件,其中包含一个与他控制的web服务器链接的链接。如果任何用户访问链接(因为她点击它,或者因为她的浏览器加载了在HTML格式的电子邮件中包含的图像,攻击者实时接收用户的会话标记。攻击者可以在他的服务器上运行一个简单的脚本,以劫持接收和执行一些恶意行动的会话,比如发送垃圾邮件、获取个人信息或更改密码。

**NOTE**<br>
当前版本的Internet Explorer不包括一个引用头,因为在一个页面中包含的离线链接被访问HTTPS。在这种情况下,Firefox包括了引用头,前提是离线站点链接也被访问HTTPS,即使它属于一个不同的域。因此,放置在url中的敏感数据很容易在引用日志中泄漏,甚至在使用SSL的地方。

**HACK STEPS**<br>
- 1.当前版本的Internet Explorer不包括一个引用头,因为在一个页面中包含的离线链接被访问HTTPS。在这种情况下,Firefox包括了引用头,前提是离线站点链接也被访问HTTPS,即使它属于一个不同的域。因此,放置在url中的敏感数据很容易在引用日志中泄漏,甚至在使用SSL的地方。
- 2.在应用程序中识别会话令牌在URL中传输的任何实例。可能是令牌通常以更安全的方式传输,但开发人员在特定的案例中使用了URL,以解决特定的困难。例如,这种行为通常被观察到一个外部系统的web应用程序接口。
- 3.如果会话令牌在url中传输,尝试找到任何应用程序功能,使您能够将任意的离线链接注入到其他用户查看的页面中。示例包括实现消息板的功能、站点反馈、问答和等等。如果是这样,提交链接到一个web服务器,您可以控制并等待任何用户的会话令牌在您的参考日志中收到。
- 4.如果有任何会话令牌被捕获,则尝试通过使用应用程序来劫持用户会话,但将捕获的令牌替换为您自己的。您可以通过拦截服务器的下一个响应来做到这一点,并在捕获的cookie值中添加您自己的Set-Cookie头。在Burp中,您可以应用一个单一的适合的配置,在所有向目标应用程序的请求中设置一个特定的cookie,以允许在测试期间轻松切换不同的会话上下文。
- 5.如果捕获了大量的令牌,会话劫持允许您访问诸如个人细节、支付信息或用户密码等敏感数据,您可以使用第14章中描述的自动化技术来获取属于其他应用程序用户的所有所需数据。

# 3.将令牌映射到会话的脆弱映射(Vulnerable Mapping of Tokens to Sessions)

会话管理机制的各种常见漏洞会出现,因为应用程序如何将会话令牌的创建和处理映射到单个用户的会话本身。

最简单的缺点是允许将多个有效的令牌同时分配给相同的用户帐户。在几乎每个应用程序中,没有合法的理由,为什么用户应该一次有多个会话活动。当然,用户放弃一个主动会话并启动一个新的会话是相当普遍的,因为他关闭了一个浏览器窗口或移动到不同的计算机。但是,如果用户似乎同时使用两个不同的会话,这通常意味着一个安全的妥协发生了:要么用户已经将他的凭证披露给另一个政党,要么攻击者通过其他方式获得了他的证书。在这两种情况下,允许并发会话是不受欢迎的,因为它允许用户在不方便的情况下坚持不受欢迎的做法,因为它允许攻击者使用捕获的证书,而无需检测风险。

一个相关但明显的弱点是应用程序使用“静态”令牌。这些看起来像会话令牌,可能最初看起来像它们一样,但实际上它们并不是这样的。在这些应用程序中,每个用户都被分配一个令牌,当他登录时,同样的令牌就会被重新发布给用户。应用程序总是以有效的方式接受令牌,而不管用户是否最近登录并使用它。像这样的应用程序确实涉及到对一个会话的整个概念的误解,以及它提供了对应用程序管理和控制访问的好处。有时,应用程序像这样操作,作为一种实现设计不佳的“记住我”功能的方法,并且静态令牌就会被存储在一个持久的cookie中(见第6章)容易受到预测攻击,使脆弱性更加严重。而不是损害当前登录用户的会话,这是一个成功的攻击妥协,一直以来,所有注册用户的帐户。

其他类型的奇怪的应用程序行为也偶尔被观察到,这证明了令牌和会话之间关系的根本缺陷。一个例子是一个有意义的令牌是基于用户名和一个随机组件构建的。例如,考虑令牌:
```
dXNlcj1kYWY7cjE9MTMwOTQxODEyMTM0NTkwMTI=
```
Base64解码为:
```
user=daf;r1=13094181213459012
```
在对`r1`组件进行了广泛的分析之后,我们可以得出这样的结论,这不能根据值的样本来预测。然而,如果应用程序的会话处理逻辑是错误的,可能是攻击者只需提交任何有效值为`r1`,任何有效值作为`user`在特定用户的安全上下文中访问会话。这本质上是一个访问控制漏洞,因为关于访问的决定是在会话之外用户提供的数据的基础上进行的(见第8章),因为应用程序有效地使用会话令牌来表示请求者已经建立了某种有效的会话。然而,会话处理的用户上下文并不是会话本身的整体属性,而是通过其他方法确定每个请求。在这种情况下,这意味着可以直接由请求者控制。

**HACK STEPS**
- 1.使用相同的用户帐户登录到应用程序,无论是从不同的浏览器过程还是不同的计算机。确定两个会话是否同时保持活动。如果是这样的话,应用程序支持并发会话,使攻击者允许使用这些没有风险检测的用户的证书。
- 2.使用相同的用户帐户登录和注销好几次,无论是来自不同的浏览器过程还是来自不同的计算机。确定每次发布的新会话令牌,或者每次登录时是否发布相同的令牌。如果后者发生,应用程序并不真正使用适当的会话。
- 3.如果令牌似乎包含任何结构和含义,则尝试将可能识别出似乎不可理解的组件的组件分开。试着修改令牌中的任何用户相关组件,以便它们指的是应用程序的其他已知用户,并验证该应用程序是否接受了生成令牌,并使您能够伪装为该用户。

[Chapter 7 Attacking Session Management(2)-Weaknesses in Token Generation](https://dm116.github.io/2020/03/13/attacking-session_management_2/)<br>
[Chapter 7 Attacking Session Management(3)-Weakness in Session Token Handling(2)](https://dm116.github.io/2020/03/13/attacking-session_management_3_2/)<br>