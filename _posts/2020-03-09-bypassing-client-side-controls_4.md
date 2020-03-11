---
layout:		post
title:		Chapter 5 Bypassing Client-Side Controls(4) - Capturing User Data:Browser Extensions
subtitle:
date:		2020-03-09
author:		D
header-img:
catalog:	true
mermaid:	true
tags: [web hacking]
---

参考: *The Web Application Hacker's Handbook*

# 1.安全地处理客户端数据(Handling Client-Side Data Securely)

如您所见，Web应用程序出现了核心安全问题，因为客户端组件和用户输入不在服务器的直接控制范围内。 客户端以及从客户端接收的所有数据本来就是不可信的。

## 1.1 通过客户端传输数据(Transmitting Data Via the Client)

由于许多应用程序以不安全的方式通过客户端传输诸如产品价格和折扣率之类的关键数据，因此许多应用程序自身处于暴露状态。

如果可能，应用程序应避免通过客户端传输此类数据。实际上，在任何可能的情况下，都可以将此类数据保存在服务器上，并在需要时直接从服务器端逻辑中引用它们。例如，接收用户对各种产品的订单的应用程序应允许用户提交产品代码和数量，并在服务器端数据库中查找每个请求的产品的价格。用户无需将商品价格提交回服务器。即使应用程序为不同的用户提供不同的价格或折扣，也无需脱离此模型。价格可以按用户存储在数据库中，折扣率可以存储在用户配置文件或会话对象中。该应用程序已经在服务器端拥有了为特定用户计算特定产品价格所需的所有信息。它必须。否则，在不安全的模型上，将无法以隐藏形式存储此价格。

如果开发人员认为他们别无选择，只能通过客户端传输关键数据，则应对数据进行签名和/或加密以防止用户篡改。 如果采取此措施，则应避免两个重要陷阱：
- 使用签名或加密数据的某些方式可能容易受到重放攻击。 例如，如果产品价格在存储在隐藏字段中之前已加密，则可以复制便宜产品的加密价格，然后代替原始价格提交。 为了防止这种攻击，应用程序需要在加密的数据中包含足够的上下文，以防止在不同的上下文中重播它。 例如，应用程序可以连接产品代码和价格，将结果作为单个项目加密，然后验证与订单一起提交的加密字符串实际上与所订购的产品匹配。
- 如果用户知道和/或控制发送给他们的加密字符串的明文值，则他们可能能够发起各种加密攻击，以发现服务器正在使用的加密密钥。 完成此操作后，他们可以加密任意值并完全避开该解决方案提供的保护。

在ASP.NET平台上运行的应用程序中，建议不要在ViewState中存储任何自定义数据，尤其是那些您不想在屏幕上显示给用户的敏感信息。 始终应激活启用ViewState MAC的选项。

## 1.2 验证客户端生成的数据(Validating Client-Generated Data)

在ASP.NET平台上运行的应用程序中，建议不要在ViewState中存储任何自定义数据，尤其是那些您不想在屏幕上显示给用户的敏感信息。 始终应激活启用ViewState MAC的选项。

- 1.轻量级的客户端控件（例如HTML表单字段和JavaScript）可以轻松规避，并且不能保证服务器收到的输入。
- 2.在浏览器扩展组件中实现的控件有时更难规避，但这可能只会在短时间内使攻击者放慢速度。
- 3.使用大量混淆或打包的客户端代码会带来更多障碍。 （在其他方面的比较点是，使用DRM技术来防止用户复制数字媒体文件。许多公司已在这些客户端控件和新解决方案上进行了大量投资 通常会在短时间内损坏。）

验证客户端生成的数据的唯一安全方法是在应用程序的服务器端。 从客户端收到的每一项数据都应视为已污染并且可能是恶意的。

## 1.3 记录和警报(Logging and Alerting)

当应用程序采用诸如长度限制和基于JavaScript的验证之类的机制来增强性能和可用性时，这些应与服务器端入侵检测防御集成在一起。 执行客户端提交的数据的验证的服务器端逻辑应注意已经在客户端进行的验证。 如果收到了本应被客户端验证阻止的数据，则应用程序可能会推断用户正在积极规避此验证，因此很可能是恶意的。 应记录异常情况，并在适当时向应用程序管理员发出实时警报，以便他们可以监视任何尝试的攻击并根据需要采取适当的措施。 该应用程序还可以通过终止用户的会话甚至暂停其帐户来积极地为自己辩护。

**NOTE**<br>
在某些使用JavaScript的情况下，仍然可以在浏览器中禁用JavaScript的用户使用该应用程序。 在这种情况下，浏览器仅跳过基于JavaScript的表单验证代码，然后提交用户输入的原始输入。 为避免误报，日志记录和警报机制应了解在何处以及如何发生。

# summary

几乎所有的客户端/服务器应用程序都必须接受以下事实：客户端组件及其上发生的所有处理都不能被信任为按预期方式运行。 如您所见，Web应用程序通常采用的透明通信方法意味着，具备简单工具和最低技能的攻击者可以轻松地绕过客户端上实施的大多数控件。 即使应用程序试图混淆客户端上的数据和处理，坚定的攻击者也可能会破坏这些防御措施。

在每个实例中，如果您确定要通过客户端传输的数据，或者在客户端上实现对用户提供的输入的验证，则应测试服务器如何响应绕过这些控件的意外数据。 通常，严重的漏洞会隐藏在应用程序对客户端实施的防御所提供的保护的假设的背后。


[Chapter 5 Bypassing Client-Side Controls(3) - Capturing User Data:Browser Extensions](https://dm116.github.io/2020/03/08/bypassing-client-side-controls_3/)