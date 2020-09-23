--- 
layout: post
title:  Vulerabilities in multi-factor authentication(多因素身份验证中的漏洞)
subtitle:
date: 2020-09-23
author: D
header-img:
catalog: true
tags: [multi-factor, authentication]
---

# 1. 2FA simple bypass(绕过两因素验证)
例子: 你已經有一個網站的有效用戶名和密碼,但是你卻沒有用戶的`2FA`驗證碼.<br>
Your credentials: `wiener:peter`<br>
Victim's credentials: `carlos:montoya`<br>
- 1.登錄你自己的帳號.你的2FA驗證碼會發送到你的email.點擊<kbd>Email client</kbd>按鈕查看你的郵件.
- 2.驗證完成，登錄後，去到你的"My account"頁面且記錄下該網址.
- 3.登出你的帳號
- 4.使用受害者的憑證登錄.
- 5.當提示要求輸入驗證碼的時候，手工修改URL導航到 `/my-account`.該驗證就會被繞過了.

# 2. 2FA broken logic(有缺陷的两因素验证逻辑) 
有時候2FA出現邏輯漏洞,當一個用戶完成了第一步登錄,網頁並沒有充分的去驗證是否是同一個用戶進行的第二步.
比如:用戶登錄的第一步如下:
```
POST /login-steps/first HTTP/1.1
Host: vulnerable-website.com
...
username=carlos&password=qwerty
```
在進行登錄的第二步之前,用戶被分配一個與帳號相關的cookie如下:
```
HTTP/1.1 200 OK
Set-Cookie: account=carlos
```
然後進行登錄的第二步:
```
GET /login-steps/second HTTP/1.1
Cookie: account=carlos
```
當提交驗證碼,請求使用cookie決定訪問哪一個賬戶:
```
POST /login-steps/second HTTP/1.1
Host: vulnerable-website.com
Cookie: account=carlos
...
verification-code=123456
```
 如果是這種情況，攻擊者可以登錄他們自己的帳號但是呢修改cookiel里面`account`的值,
 可以把它修改爲任意值當提交驗證碼的時候:
 ```
 POST /login-steps/second HTTP/1.1
 Host: vulerable-website.com
 Cookie: account=victim-user
 ...
 verification-code=123456
 ```
 這樣子是非常危險的，如果攻擊者可以暴力破解驗證碼,那麼他們可以登錄任何帳號僅僅需要
 知道受害者的用戶名,而不用知道受害者的密碼.

# 3. 2FA bypass using a brute-force attack(暴力破解2FA验证码)