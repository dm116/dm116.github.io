---
layout:     post
title:      Chapter 9 Attacking Data Stores(1)-Injecting into Interpreted Contexts
subtitle:   注入解释的上下文
date:       2020-03-14
author:     D
header-img: 
catalog: true
mermaid: true
tags: [web hacking]
---

参考:**The Web Application Hackers Handbook** Chapter 9

几乎所有应用程序都依赖数据存储来管理在应用程序内处理的数据。 在许多情况下，这些数据驱动核心应用程序逻辑，持有用户帐户，权限，应用程序配置设置等。数据存储的发展已远远超过了被动数据存储容器。 大多数以结构化格式保存数据，可使用预定义的查询格式或语言对其进行访问，并包含内部逻辑以帮助管理该数据。

通常，应用程序对数据存储的所有类型的访问以及在处理属于不同应用程序用户的数据时使用通用特权级别。 如果攻击者可以干扰应用程序与数据存储的交互，以使其能够检索或修改不同的数据，则通常可以绕过对应用程序层施加的数据访问的任何控制。

刚刚描述的原理可以应用于任何种类的数据存储技术。 因为这是一本实用的手册，所以我们将重点介绍开发实际应用程序中存在的漏洞所需的知识和技术。 到目前为止，最常见的数据存储是SQL数据库，基于XML的存储库和LDAP目录。 还涵盖了其他地方看到的实际示例。

在介绍这些关键示例时，我们将描述您可以用来识别和利用这些缺陷的实际步骤。 在理解每种新型注射过程中，存在概念上的协同作用。 掌握了利用漏洞的这些表现的基本知识后，您应该确信，当遇到新的注射类别时，您可以利用这种理解。 确实，您应该能够设计出其他手段来攻击其他人已经研究过的方法。

# 注入解释的上下文(Injecting into Interpreted Contexts)

一种解释语言是一种执行语言，其中涉及一个运行时组件，该组件解释该语言的代码并执行其中包含的指令。 相反，编译语言是一种其生成时将其代码转换为机器指令的语言。 在运行时，这些指令直接由运行它的计算机的处理器执行。

原则上，任何语言都可以使用解释器或编译器来实现，并且区别不是该语言本身的固有属性。 尽管如此，大多数语言通常仅以这两种方式之一来实现，而用于开发Web应用程序的许多核心语言是使用解释器来实现的，包括SQL，LDAP，Perl和PHP。

由于解释语言的执行方式，出现了一系列称为代码注入的漏洞。 在任何有用的应用程序中，用户提供的数据都将被接收，处理和操作。 因此，解释器处理的代码是程序员编写的指令和用户提供的数据的混合。 在某些情况下，攻击者通常可以通过提供所使用的解释语言的语法内具有特殊意义的某些语法来提供突破数据上下文的精心设计的输入。 结果是该输入的一部分被解释为程序指令，其执行方式与原始程序员编写的方式相同。 因此，成功的攻击通常会完全破坏目标应用程序的组件。

另一方面，在本机编译语言中，旨在执行任意命令的攻击通常非常不同。 注入代码的方法通常不利用用于开发目标程序的语言的任何语法特征，并且注入的有效载荷通常包含机器代码，而不是用该语言编写的指令。 有关针对本机编译软件的常见攻击的详细信息，请参见第16章。

## 绕过登录(Bypassing a Login)

应用程序访问数据存储区的过程通常是相同的，无论该访问是由非特权用户或应用程序管理员的操作触发的。 该Web应用程序充当对数据存储的自由访问控制，构造查询以根据用户的帐户和类型检索，添加或修改数据存储中的数据。 修改查询（而不仅仅是查询中的数据）的成功注入攻击可以绕过应用程序的自由访问控制并获得未授权的访问。

如果对安全敏感的应用程序逻辑由查询结果控制，则攻击者可能会修改查询以更改应用程序的逻辑。 让我们看一个典型的示例，其中在后端数据存储中查询用户表中与用户提供的凭据匹配的记录。 许多实现基于表单的登录功能的应用程序都使用数据库来存储用户凭据，并执行简单的SQL查询来验证每次登录尝试。 这是一个典型的例子：
```
SELECT * FROM users WHERE username = 'marcus' and password = 'secret'
```
该查询使数据库检查users表中的每一行，并提取出其中`username`列的值为`marcus`且`password`列的值为`secret`的每条记录。 如果将用户的详细信息返回给应用程序，则表示登录尝试成功，并且该应用程序为该用户创建了经过身份验证的会话。

在这种情况下，攻击者可以将用户名或密码字段注入以修改应用程序执行的查询，从而颠覆其逻辑。 例如，如果攻击者知道应用程序管理员的用户名是admin，那么他可以通过提供任何密码和以下用户名来以该用户身份登录：
```
admin'--
```
这将导致应用程序执行以下查询：
```
SELECT * FROM users WHERE username = 'admin'--' AND password = 'foo'
```
请注意，注释序列(--)导致查询的其余部分被忽略，因此执行的查询等效于：
```
SELECT * FROM users WHERE username = 'admin'
```
因此将跳过密码检查。

假设攻击者不知道管理员的用户名。 在大多数应用程序中，数据库中的第一个帐户是管理用户，因为该帐户通常是手动创建的，然后用于通过应用程序生成所有其他帐户。 此外，如果查询返回了多个用户的详细信息，则大多数应用程序将仅处理返回其详细信息的第一个用户。 攻击者通常可以通过提供用户名利用此行为以数据库中的第一位用户身份登录：
```
' OR 1=1--
```
这将导致应用程序执行查询：
```
SELECT * FROM users WHERE username = '' OR 1=1--' AND password = 'foo'
```
由于有注释符号，因此等效于：
```
SELECT * FROM users WHERE username = '' OR 1=1
```
返回所有应用程序用户的详细信息。

**NOTE**
注入已解释的上下文以更改应用程序逻辑是一种通用的攻击技术。 LDAP查询，XPath查询，消息队列实现或任何自定义查询语言中可能会出现相应的漏洞。

**HACK STEPS**<br>
注入解释性语言是一个广泛的主题，涉及许多不同类型的漏洞，并可能影响Web应用程序支持基础结构的每个组件。 检测和利用代码注入漏洞的详细步骤取决于目标语言和应用程序开发人员采用的编程技术。 但是，在每种情况下，通用方法如下：
- 1.提供可能在特定解释语言的上下文中引起问题的意外语法。
- 2.识别应用程序响应中可能表明存在代码注入漏洞的任何异常情况。
- 3.如果收到任何错误消息，请检查这些错误消息以获取有关服务器上发生的问题的证据。
- 4.如有必要，以相关方式系统地修改您的初始输入，以尝试确认或反对对漏洞的初步诊断。
- 5.构造概念验证测试，以可验证的方式执行安全命令，以最终证明存在可利用的代码注入漏洞。
- 6.通过利用目标语言和组件的功能来实现目标来利用此漏洞。

[Chapter 8 Attacking Access Controls(3)-Securing Access Controls](https://dm116.github.io/2020/03/13/attacking-access-controls_3/)<br>
[Chapter 9 Attacking Data Stores(2)-Injecting into SQL(1)](https://dm116.github.io/2020/03/13/attacking-data-stores_2_1/)<br>

