---
layout:		post
title:		Chapter 6 Attacking Authentication(4) - Securing Authentication(1)
subtitle:	
date:		2020-03-12
author:		D
header-img:
catalog:	true
mermaid:	true
tags: [web hacking]
---

参考: *The Web Application Hacker's Handbook* Chapter 6


实现一个安全的身份验证解决方案包括尝试同时满足几个关键的安全目标,在许多情况下,与其他目标进行交易,比如功能、可用性和总成本。在某些情况下,“更多”的安全实际上会适得其反。例如,强制用户设置很长的密码并经常更改它们经常会导致用户写下他们的密码。

由于存在各种可能的身份验证漏洞,以及可能需要部署的可能复杂的防御系统,以便对所有这些漏洞进行部署,许多应用程序设计人员和开发人员选择接受特定的威胁,并专注于防止最严重的攻击。这里有一些因素需要考虑,以确定适当的**平衡**:
- 考虑到应用程序提供的功能,安全性的重要性
- 用户将容忍和使用不同类型的身份验证控件的程度
- 支持一个不那么友好的系统的成本
- 竞争替代方案的财务成本,可能是由应用程序或其保护资产的价值产生的收入

# 1.使用强证书(Use Strong Credentials)

- 应严格执行适当的密码质量要求。这些可能包括关于最小长度的规则;字母、数字和排版字符的外观;两个大写字母和小写字符的外观;避免字典单词,名称和其他常见密码;防止密码被设置为用户名;防止与之前设置密码的相似或匹配。与大多数安全措施一样,不同的密码质量要求可能适合不同类别的用户。
- 用户名应该是唯一的。
- 任何系统生成的用户名和密码都应该用足够的熵来创建,它们不能被确定或预测——即使是一个攻击者可以获得一个大的连续生成实例的示例。
- 用户应该被允许设置足够强的密码。例如,应该允许长密码和广泛的字符。

# 2.秘密处理凭证(Handle Credential Secretively)

- 所有的证书都应该创建、存储和传输,而不导致未经授权的披露。
- 所有客户机-服务器通信都应该使用一种完善的加密技术,如SSL来保护。在运输中保护数据的自定义解决方案既不必要也不可取。
- 如果认为最好使用HTTP用于应用程序的未经验证的区域,确保登录表单本身使用HTTPS加载,而不是在登录提交的时候切换到HTTPS。
- 只有POST请求应该用于将凭证传输到服务器。证书不应该放置在URL参数或cookie中(甚至是临时cookie)。凭据不应该传递回客户机,即使是在重定向的参数中。
- 所有服务器端应用程序组件都应该以一种不允许其原始值轻松恢复的方式存储凭据,即使是在应用程序数据库中获得对所有相关数据的完整访问的攻击者。实现这一目标的通常方法是使用一个强的哈希函数(例如,在此编写时,例如SHA-256),适当地加盐,以减少预行脱机攻击的有效性。盐应该是专用于拥有密码的帐户,这样攻击者不能重放或替代散列值。
- 客户端“记住我”的功能应该只记得用户名之类的非机密项目。在更少的安全性关键应用程序中,可能被认为是合适的,允许用户选择一个设备来记住密码。在这种情况下,不应该存储在客户机上(密码应该使用只在服务器上已知的密钥来存储可逆加密)。此外,用户应该从攻击者的风险中发出警告,他们有物理访问计算机或远程访问计算机的人。应该特别注意在应用程序中消除跨站点的脚本漏洞
这可能被用来窃取存储凭证(见第12章)。
- 应该实现密码更改设施(参见“防止密码更改功能”部分的滥用),用户应该需要定期更改他们的密码。
- 在将新帐户的凭证分发给用户退出时,应该尽可能安全地发送,并且应该是时间限制的。用户应该被要求在第一次登录时更改它们,并应该被告知在使用后销毁通信。
- 在适用的地方,考虑捕获一些用户的登录信息(例如,从一个令人难忘的单词中发送的单字母),使用下拉菜单而不是文本字段。这将防止在用户的电脑上安装任何键盘记录器,以获取用户提交的所有数据。但是,请注意,一个简单的keylogger只是一个攻击者可以捕获用户输入的方法。如果他或她已经破坏了用户的计算机,在原则上,攻击者可以记录每一种事件,包括鼠标动作,在HTTPS上表单提交,以及屏幕截图。

# 3.正确验证凭证(Validate Credentials Properly)

- 密码应该完全被验证——也就是说,以一种区分大小写的方式,不干扰或修改任何字符,而不会截断密码。
- 应用程序应该积极地保护自己,防止在登录处理过程中发生的意外事件。例如,根据使用的开发语言,应用程序应该在所有API调用周围使用catch-all异常处理程序。这些应该显式地删除用于控制登录处理状态的所有会话和方法-本地数据,并应该显式地使当前会话失效,从而导致服务器被迫注销,即使身份验证被某种方式绕过。
- 所有的身份验证逻辑都应该与伪代码和实际的应用程序源代码一起被仔细地描述,以识别逻辑错误,如故障打开的条件。
- 如果实现支持用户模拟的功能,则应该严格控制,以确保它不能被误用获得未经授权的访问。由于功能的临界性,通常可以从公共应用程序中删除该功能,并只对内部管理用户执行它,它们的模拟使用应该受到严格控制和审计。
- 多阶段登录应该严格控制,以防止攻击者干扰阶段之间的转换和关系:
	- 所有关于进度的数据,以及之前的验证任务的结果都应该在服务器端会话对象中进行,不应该从客户端传输到或读取。
	- 不应该向用户提交不止一次的信息,也不应该让用户修改已经收集和/或验证的数据。在多个阶段使用诸如用户名之类的数据的项目时,这应该存储在一个会话变量中,当第一次收集并引用之后。
	- 在每个阶段进行的第一个任务应该是验证所有之前的阶段已经正确完成。如果不是这样,身份验证的尝试应该立即被标记为坏的。
	- 为了防止对登录失败的哪个阶段的信息泄漏(这将使攻击者能够实现每个阶段的目标),应用程序应该始终进行登录的所有阶段,即使用户未能正确完成早期阶段,即使原始用户名无效。在完成所有阶段之后,应用程序应该在非主阶段的结论中出现一个通用的“登录失败”消息,而不提供任何故障发生在哪里的信息。
- 在一个登录过程中,包括一个随机的问题,确保攻击者不能有效地选择他自己的问题:
	- 总是使用一个多阶段的过程,用户在初始阶段识别自己,然后在后面的阶段向他们提出随机问题。
	- 当给定的用户被提出一个给定的问题时,在她持续的用户配置文件中存储这个问题,并确保在每个尝试登录之前,都要对相同的用户提出同样的问题,直到她成功地回答了这个问题。
	- 当向用户提出随机不同的挑战时,存储在服务器端会话变量中被询问的问题,而不是在HTML表单中隐藏的字段,并验证后续的回答与保存的问题。

**NOTE**<br>
在这里设计一个安全的认证机制的微妙之处。如果在询问一个随机的问题时,这可能会导致用户名枚举的新机会。例如,为了防止攻击者选择自己的问题,应用程序可以在每个用户的配置文件中存储用户被要求的最后一个问题,并继续提出这个问题,直到用户正确地回答它。一个攻击者使用任何给定用户的用户名启动几个登录将会得到同样的问题。但是,如果攻击者使用无效的用户名进行相同的过程,应用程序可能会采取不同的行为:因为没有用户配置文件与无效的用户名相关联,因此将没有存储问题,因此将呈现一个不同的问题。攻击者可以使用这个行为的差异,表现为多次登录尝试,以推断出给定用户名的有效性。在有剧本的攻击中,他将能够收获大量的用户名很快。

如果一个应用程序想要保护自己不受这种可能性的影响,它必须达到一定的长度。当登录尝试使用无效的用户名启动时,应用程序必须在某个随机问题上记录它提交给那个无效的用户名,并确保使用相同的用户名的后续登录尝试也遇到同样的问题。更进一步,应用程序可以周期性地切换到不同的问题,以模拟不存在的用户的正常登录,从而导致下一个问题的更改!然而,在某种程度上,应用程序设计人员必须划定一条线,并承认对这样一个确定的攻击者的总胜利可能是不可能的。

[Chapter 6 Attacking Authentication(3) - Implementation Flaws in Authentication](https://dm116.github.io/2020/03/11/attacking-authentication_3/)<br>
[Chapter 6 Attacking Authentication(4) - Securing Authentication(2)](https://dm116.github.io/2020/03/12/attacking-authentication_4_2/)<br>
