---
layout:		post
title:		Chapter 6 Attacking Authentication(1) - Authentication Technologies
subtitle:
date:		2020-03-09
author:		D
header-img:
catalog:	true
mermaid:	true
tags: [web hacking]
---

参考: *The Web Application Hacker's Handbook*

从表面上看，身份验证是Web应用程序中采用的所有安全机制中最简单的一种。 在典型情况下，用户提供其用户名和密码，应用程序必须验证这些项目正确无误。 如果是这样，则允许用户进入。如果不是，则不允许。

身份验证也是应用程序防范恶意攻击的核心。 它是防止未经授权访问的第一道防线。 如果攻击者可以打败这些防御措施，那么他通常将获得对该应用程序功能的完全控制，并且可以不受限制地访问其中的数据。 如果没有可靠的身份验证依赖，其他核心安全机制都不会
（例如会话管理和访问控制）可以有效。

实际上，尽管表面上看起来很简单，但是设计安全的身份验证功能却是一项微妙的工作。 在实际的Web应用程序中，身份验证通常是最薄弱的环节，这使攻击者可以获得未经授权的访问。 由于身份验证逻辑中的各种缺陷，作者已经失去了我们从根本上折衷的应用程序数量。

本章详细讨论通常与Web应用程序有关的各种设计和实现缺陷。 这些通常是由于应用程序设计人员和开发人员无法提出一个简单的问题而引起的：如果攻击者针对我们的身份验证机制，将会达到什么目的？ 在大多数情况下，一经认真询问特定应用程序就提出了这个问题，那么许多潜在的漏洞就会浮出水面，其中任何一个都足以破坏该应用程序。

许多最常见的身份验证漏洞都是显而易见的。 任何人都可以在登录表单中键入词典单词，以猜测有效的密码。 在其他情况下，细微的缺陷可能潜伏在应用程序的处理过程中，只有在对复杂的多阶段登录机制进行艰苦的分析之后，才能发现和利用这些缺陷。 我们将描述这些攻击的全部范围，包括成功打破地球上一些最安全关键和防御最牢固的Web应用程序的身份验证的技术。

# 身份认证技术(Authentication Technologies)

实施身份验证机制时，Web应用程序开发人员可以使用多种技术：
- 1.基于HTML表单的身份验证
- 2.多因素机制，例如结合了密码和物理令牌的机制
- 3.客户端SSL证书和/或智能卡
- 4.HTTP基本和摘要身份验证
- 5.使用NTLM或Kerberos的Windows集成身份验证
- 6.认证服务

到目前为止，Web应用程序使用的最常见的身份验证机制使用HTML表单来捕获用户名和密码，并将其提交给应用程序。 此机制占您可能在Internet上遇到的应用程序的90％以上。

在对安全性要求更高的Internet应用程序（例如在线银行）中，此基本机制通常扩展为多个阶段，要求用户提交其他凭据，例如PIN或从秘密单词中选择的字符。 HTML表单通常仍用于捕获相关数据。

在对安全性要求最高的应用程序中，例如针对高价值个人的私人银行业务，通常会遇到使用物理令牌的多因素机制。 这些令牌通常会根据应用程序指定的输入生成一次性密码流或执行质询响应功能。 随着这项技术的成本随着时间的推移而下降，可能会有更多的应用程序采用这种机制。 但是，这些解决方案中有许多实际上并未解决针对它们而设计的威胁，主要是网络钓鱼攻击和使用客户端木马的威胁。

某些Web应用程序采用智能卡内实现的客户端SSL证书或加密机制。 由于管理和分发这些项目的开销很大，因此通常仅在应用程序用户群较小的安全关键环境中使用它们，例如，用于远程办公人员的基于Web的VPN。

基于HTTP的身份验证机制（基本，摘要和Windows集成）很少在Internet上使用。 它们在Intranet环境中更为常见，在该环境中，组织的内部用户通过提供其常规网络或域凭据即可访问公司应用程序。 然后，应用程序使用其中一种技术来处理这些凭据。

偶尔会遇到诸如Microsoft Passport之类的第三方身份验证服务，但是目前，它们还没有被大规模采用。

与身份验证有关的大多数漏洞和攻击都可以应用于所提及的任何技术。 由于基于HTML表单的身份验证的压倒性优势，我们将在这种情况下描述每个特定的漏洞和攻击。 在相关的地方，我们将指出与其他可用技术有关的特定差异和攻击方法。

[Chapter 6 Attacking Authentication(2) - Design Flaws in Authentication Mechanisms(1)](https://dm116.github.io/2020/03/09/attacking-authentication_2/)