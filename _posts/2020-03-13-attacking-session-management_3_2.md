---
layout:		post
title:		Chapter 7 Attacking Session Management(3)-Weakness in Session Token Handling(2)
subtitle:	
date:		2020-03-13
author:		D
header-img:
catalog:	true
mermaid:	true
tags: [web hacking]:
---

参考: *The Web Application Hacker's Handbook* Chapter 7

# 1.脆弱的会话终止(Vulnerable Session Termination)

适当的终止会话是重要的两个原因。首先,将会话的寿命缩短为必要时,减少攻击者可能捕获、猜测或误用一个有效的会话令牌的机会窗口。第二,它为用户提供了一种方法,当他们不再需要时,它可以使现有的会话无效。这使他们能够进一步减少这一窗口,并在共享计算中承担一些责任环境。会话终止函数的主要弱点包括失败,以满足这两个关键目标。

有些应用程序不执行有效的会话过期。一旦创建了,在服务器最终到期之前,在最后请求收到后的许多天内,会话可能仍然有效。如果令牌很容易受到某种类型的排序,这是一个特别扩散的命令(例如,每个有效的标记标识的10万次猜测),攻击者仍然可以捕获到每个用户在最近的过去访问应用程序的令牌。

一些应用程序没有提供有效的注销功能:
- 在某些情况下,注销函数根本没有实现。用户没有任何方法导致应用程序使其会话失效。
- 在某些情况下,注销函数实际上并没有导致服务器使会话失效。服务器从用户的浏览器中删除了令牌(例如,通过发出一个set cookie指令来清空令牌)。但是,如果用户继续提交令牌,服务器仍然接受它。
- 在最坏的情况下,当用户单击退出时,这个事实没有与服务器通信,因此服务器没有执行任何动作。相反,一个客户端脚本被执行,它会删除用户的cookie,这意味着随后的请求将用户返回到登录页面。获取此cookie的攻击者可以使用会话,就好像用户从未退出。

一些不使用身份验证的应用程序仍然包含了允许用户在其会话中构建敏感数据的功能(例如,购物应用程序)。然而,通常它们没有提供任何类似于用户终止会话的注销函数。

**HACK STEPS**<br>
- 1.不要陷入检查应用程序在客户端令牌上执行的操作的陷阱(如cookie指令、客户端脚本或过期时间属性)。在会话终止方面,没有什么取决于客户端浏览器中的令牌发生了什么。相反,调查服务器端是否实现会话过期:
	- a.登录到应用程序以获得一个有效的会话令牌。
	- b.等待一个不使用这个令牌的时期,然后提交一个保护页面的请求(比如“我的细节”)使用令牌。
	- c.如果页面显示为正常的,那么令牌仍然是活跃的。
	- d.使用试用和错误来确定任何会话过期超时时间是多长,或者在最后一次请求使用后是否还可以使用令牌。Burp入侵者可以被配置为增加连续请求之间的时间间隔,以自动化此任务。
- 2.确定退出函数是否存在,并在用户中获得显著的可用。如果不是,用户更容易受到伤害,因为他们没有办法导致应用程序使他们的会话失效。
- 3.提供了一个注销函数,测试其有效性。在注销后,尝试重用旧的令牌,并确定它是否仍然有效。如果是这样的话,即使在他们“注销”之后,用户仍然容易受到一些会议的劫持袭击。“您可以使用Burp套件来测试这一点,通过选择来自代理历史的最近的session依赖的请求,并将其发送到Burp中继器,在您从应用程序中退出后重新发布。

# 2.客户收到令牌劫持(Client Exposure to Token Hijacking)

攻击者可以针对应用程序的其他用户,试图以多种方式捕获或误用受害者的会话令牌:
- 对跨站点脚本攻击的明显有效负载是查询用户的cookie,以获得她的会话令牌,然后可以传输到由攻击者控制的任意服务器。第12章详细描述了这种攻击的各种排列。
- 针对用户的其他攻击可以用不同的方式来劫持用户会话。通过会话固定漏洞,攻击者将一个已知的会话令牌传递给用户,等待她登录,然后劫持她的会话。通过跨站点请求伪造攻击,攻击者通过一个web站点的应用程序提出一个精心设计的请求,他利用了这个事实,即用户的浏览器自动向她的当前cookie提交这个请求。这些攻击也在第12章中描述。

**HACK  STEPS**<br>
- 1.在应用程序中识别任何跨站点的脚本漏洞,并确定这些漏洞是否可以被用来捕获其他用户的会话令牌(见第12章)。
- 2.如果应用程序向未经身份验证的用户发出会话令牌,则获取一个令牌并执行登录。如果应用程序在成功登录后没有发布新的令牌,则容易受到会话固定。
- 3.即使应用程序不向未经身份验证的用户发布会话标记,也可以通过登录来获取一个令牌,然后返回到登录页面。如果应用程序愿意返回这个页面,即使您已经经过身份验证,则使用同样的令牌提交另一个登录作为不同的用户。如果应用程序在第二次登录后没有发出新的令牌,则容易受到会话固定。
- 4.确定应用程序使用的会话令牌的格式。将您的令牌修改为一个已被验证形成的值,并尝试登录。如果应用程序允许您使用一个发明的令牌来创建一个经过身份验证的会话,那么它就容易受到会话固定的影响。
- 5.如果应用程序不支持登录,但处理敏感的用户信息(如个人和支付细节),并允许在提交后显示这一点(比如在“验证我的订单”页面),在之前的三个测试中执行显示敏感数据的页面。如果在应用程序的匿名使用期间的标记集被用来检索敏感的用户信息,那么应用程序就容易受到会话固定的影响。
- 6.如果应用程序使用HTTP cookie来传输会话标记,那么它很可能容易受到跨站点请求伪造(XSRF)的脆弱。首先,登录到应用程序。然后确认对应用程序的请求,但源自不同应用程序的页面,从而提交用户的令牌。(此提交需要从用于登录到目标应用程序的相同浏览器进程的窗口中进行。)试图识别任何敏感的应用程序函数,攻击者可以预先确定其参数,并利用这一点来在目标用户的安全上下文中进行未经授权的操作。请参阅第13章,了解如何执行XSRF攻击的更多细节。

# 3.自由Cookie的范围(Liberal Cookie Scope)

cookie如何工作的通常简单的总结是,服务器使用HTTP响应头集cookie处理cookie,然后浏览器将该cookie重新提交到使用cookie头的相同服务器的后续请求中。事实上,事情比这更微妙。

cookie机制允许服务器指定每个cookie将重新提交的域和URL路径。为了做到这一点,它使用了可能包含在设置cookie指令中的域和路径属性。

## 3.1 Cookie域限制(Cookie Domain Restrictions)

当应用程序驻留在`foo.wahh-app.com`设置了一个cookie,默认情况下,浏览器将cookie重新提交到`foo.wahh-app.com`的所有后续请求中,以及任何子域名,比如`admin.foo.wahh-app`.计算机它不将cookie提交到任何其他域,包括父域`wahh-app.com`和任何其他子域的父,如`bar.wahh-app.com`。

服务器可以通过在Set-cookie指令中包含域属性来覆盖这种默认行为。例如，假设`foo.wahh-app.com`上的应用程序返回以下HTTP头信息:
```
Set-cookie: sessionId=19284710; domain=wahh-app.com;
```
然后,浏览器将该cookie重新提交到`wahh-app.com`的所有子域,包括`bar.wahh-app.com`。

**NOTE**<br>
服务器不能使用此属性指定任何域。首先,域专门化必须是应用程序正在运行的同一个域,也必须是它的父域(要么立即删除)。其次,域专门化不能成为顶级领域`.com`或`.co.uk`,因为这会使恶意服务器在任何其他领域设置任意的cookie。如果服务器违反了其中一个规则,浏览器就忽略了`Set-cookie`的指令。

如果应用程序将cookie的域范围设置为不适当的自由,这可能会使应用程序暴露在各种安全漏洞上。

例如,考虑一个博客应用程序,允许用户注册、登录、写博客文章,并阅读其他人的博客。主要应用程序位于`wahh-blogs.com`。计算机当用户登录到应用程序时,他们会收到一个会话令牌,它在这个域的范围内。每个用户都可以创建通过一个新的子域访问的博客,这些子域是由他的用户名预先设置的:
```
herman.wahh-blogs.com
solero.wahh-blogs.com
```
因为cookie会自动重新提交到它们的范围内的每一个子域,当一个用户在浏览其他用户的博客时,他的会话令牌就会被他的请求提交。如果博客作者允许在他们自己的博客中放置任意JavaScript(通常在真实的博客应用程序中),恶意的博客用户可以像在存储的跨站点脚本攻击中完成的那样,以同样的方式窃取其他用户的会话令牌(见第12章)。

这个问题出现是因为用户编写的博客被创建为处理身份验证和会话管理的主要应用程序的子域。在HTTP cookie中没有任何设施可以防止主要域发布的cookie被重新提交其子域。

解决方案是使用一个不同的域名作为主要应用程序(例如,www.wahh-blogs.com),并将其会话令牌cookie的域范围范围范围范围范围内。当登录用户浏览其他用户的博客时,就不会提交会话cookie。

当应用程序显式地将其cookie的域范围设置为父域时,会出现不同版本的这个漏洞。例如,假设一个安全密钥应用程序位于域 `sensitiveapp.wahh-organization.com`。当它设置cookie时,它明确地开放了它们的域范围,如下:
```
Set-cookie: sessionId=12df098ad809a5219; domain=wahh-organization.com
```
因此,当一个用户访问`wahh-organization.com`的每一个子域时,就会提交敏感应用程序的会话令牌cookie包括:
```
www.wahh-organization.com
testapp.wahh-organization.com
```
尽管这些其他应用程序可能都属于与敏感应用程序相同的组织,但由于几个原因,敏感应用程序的cookie将被提交到其他应用程序中,这是不可取的:
- 1.对其他应用程序负责的人员可能有不同程度的信任,而不是对敏感应用程序负责。
- 2.其他应用程序可能包含允许第三方获得提交给应用程序的cookie值的功能,如前面的博客示例。
- 3.其他应用程序可能没有受到相同的安全标准或测试作为敏感应用程序(因为它们不那么重要,不处理敏感数据,或者只是为了测试目的而创建)。在这些应用程序(例如,跨站点脚本漏洞)中可能存在的许多类型的漏洞可能与这些应用程序的安全姿态无关。但它们可以使外部攻击者利用不安全的应用程序来捕获敏感应用程序创建的会话标记。

**NOTE**<br>
在一般情况下,cookie的基于domaina的cookie不那么严格(见第3章),除了在处理主机名时所描述的问题之外,浏览器在确定cookie范围时忽略协议和端口号。如果应用程序与不受信任的应用共享一个主机名,并且依赖于协议或端口号的差异来隔离本身,那么对cookie的更轻松的处理可能会破坏这种隔离。应用程序发出的任何cookie都可以通过共享其主机名的不受信任的应用程序访问。

**HACK STEPS**<br>
检查应用程序发出的所有cookie,并检查用于控制cookie范围的任何域属性。
- 1.如果应用程序显式地将其cookie的范围扩展到父域,那么它可能会因为其他web应用程序而受到攻击。
- 2.如果一个应用程序将其cookie的域范围设置为它自己的域名(或者不指定域属性),那么它可能仍然会接触到通过子域访问的应用程序或功能。

识别所有可能接收应用程序发出的cookie的域名。建立任何其他web应用程序或功能可以通过这些域名访问,您可以利用这些域名获取向目标应用程序的用户发布的cookie。

## 3.2 Cookie路径限制(Cookie Path Restrictions)

当应用程序驻留在`/apps/secure/foo-app/index.jsp`设置了一个cookie,默认情况下,浏览器将cookie重新提交到路径`/apps/secure/foo-app/`以及任何子目录中的后续请求。它不将cookie提交父目录或服务器上存在的任何其他目录路径。

与基于domainbased的关于cookie范围的限制一样,服务器可以通过设置cookie指令中包括路径属性来覆盖此默认行为。例如,如果应用程序返回以下HTTP头:
```
Set-cookie: sessionId=187ab023e09c00a881a; path=/apps/;
```
浏览器将该cookie重新提交到`/apps/`路径的所有子目录。

与基于domainbased的cookie范围相比,这种基于路径的限制比同源策略更严格。因此,如果使用安全机制来保护在同一个域上托管的不受信任的应用程序,则几乎完全无效。在一条路径运行的客户端代码可以打开一个窗口或iframe,它瞄准了同一个域的不同路径,并可以从没有任何限制的情况下读取和写入这个窗口。因此,获取一个在同一个域上的不同路径的cookie相对简单。有关更多细节,请参阅以下文件:
```
http://lists.webappsec.org/pipermail/websecurity_lists.webappsec.org/2006-March/000843.html
```

[Chapter 7 Attacking Session Management(3)-Weakness in Session Token Handling(1)](https://dm116.github.io/2020/03/13/attacking-session_management_3_1/)<br>
[Chapter 7 Attacking Session Management(4)-Securing Session Management](https://dm116.github.io/2020/03/13/attacking-session_management_4/)<br>