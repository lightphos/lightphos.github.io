---
layout: post
title: Security Shepherd
date: '2018-06-02 13:31:04'
categories: 'Tech'
author: Satish Ramjee
tags:
- security
- owasp
- training
---

https://www.owasp.org/index.php/OWASP_Security_Shepherd

## Overview

The tool helps us understand and  improve awareness of application security.

It comprises of lessons and challenges to help learn penetration testing skills. 

To complete challenges set in the project, you need to find the flaw which will display a key code to enter.

The CSRF challenges require two or more people working together to craft and solve.

The github source code is a great help in solving some of the challenges.

## Top Vulnerabilities

Security Shepheard covers appreciation of the following vulnerabilities, more details on some of these below:
1. SQL Injection
2. Broken Authentication and Session Management
3. Cross Site Scripting
4. Insecure Direct Object Reference
5. Security Misconfiguration
6. Sensitive Data Exposure
7. Missing Function Level Access Control
8. Cross Site Request Forgery
9. Unvalidated Redirects and Forwards
10. Poor Data Validation
11. Insecure Data Storage
12. Unintended Data Leakage
13. Poor Authentication and Authorisation
14. Broken crypto
15. Client Side Injection
16. Lack Of Binary Protections

The following provides detailed explanation of the 2017 top ten vulnerabilities: [2017 OWASP Top 10](https://www.owasp.org/images/7/72/OWASP_Top_10-2017_(en).pdf.pdf) 

## The Code
https://github.com/OWASP/SecurityShepherd

You can watch an overview here:
<iframe width="560" height="315" src="https://www.youtube.com/embed/5_o-sU0i8Us" frameborder="0" allow="autoplay; encrypted-media" allowfullscreen></iframe>

## Zed Attack Proxy (ZAP)

This is a useful tool to intercept requests and manipulate the data to discover the weaknesses and much more. 

https://www.zaproxy.org
https://github.com/zaproxy/zaproxy/wiki/Downloads

See this on how to set it:
<iframe width="560" height="315" src="https://www.youtube.com/embed/Xp_PBH7wjiw" frameborder="0" allow="autoplay; encrypted-media" allowfullscreen></iframe>

Once you have it running you can see the UI http://localhost:8080, and configure your browser to proxy to localhost:8080.

To configure the proxy on windows 10:
![w10-proxy](/content/images/2018/06/w10-proxy.PNG)


## Security Vulnerabilities
The security shepherd tool describes and explains the following with detail and walks you through exercises to highlight the vulnerability.

- Insecure Direct Object References
When you can modify a userid to get hold of another users details.

- Poor Data Validation
Not validating submitted data. Should be done both on client and server side.

- Security Misconfiguration
Eg. when default login details are left intact for someone to exploit.

- Broken Authentication and Session Management
Commonly found in logout, password management, secret question and account update. Eg. credentials can be guessed, sessions ids are in the URL, not using SSL.

- Failure to Restrict URL Access
Hiding an admin url is not enough, there needs to be a challenge to prevent the URL being invoked by an unauthorised access.

- Cross Site Scripting (XSS)
Any data input needs parsing otherwise a script can be input which the browser will run and take hold of sessions, deface the site or forward to malicious sites. 
To show if a site is vulnerable enter one of the following into an input field, if it pops up with 'XSS', then there is vulnerable to XSS. 
```
<SCRIPT>alert('XSS')</SCRIPT>

<IMG SRC="#" ONERROR="alert('XSS')"/>

<INPUT TYPE="BUTTON" ONCLICK="alert('XSS')"/>

<IFRAME SRC="javascript:alert('XSS');"></IFRAME>
```

- Insecure Cryptographic Storage
Not applying sufficent encryption practices for sensitive data. 

- SQL Injection Lesson
Insert SQL queries in a form field with a boolean 'OR' operator followed by a true statement like 1 = 1 which can reveal all users database information. See the lesson for details.

- Cross-Site Request Forgery
Sending a forged request using the users session details to an application. Thre should be a check that it is the user who is really accessing the application. A request should have a random nonce token that is checked on each access. Also javascript request will have a "X-Requested-With" HTTP header. This can be checked for but not guaranteed on all browsers. An example of a CSRF attack is to embed an image tag like so:
```
<img src="http://www.secureBank.ie/sendMoney?giveMoneyTo=hacker&giveAmount=1000"/> 
```

- Unvalidated Redirects and Forwards
Redirecting to a url based on a variable that is not checked which can be hijacked to point to a phishing site for example.
