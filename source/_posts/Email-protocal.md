---
title: Email protocal
date: 2022-03-23 15:04:08
categories:
- 协议
tags:
- email
---

| Name            | Usage                                                                            |
| :-------------- | :------------------------------------------------------------------------------- |
| SMTP            | simple mail transfer protocal, 相当于中转站，将邮件发送到客户端                  |
| POP3            | Post Office Protocol 3，将邮件从服务器下载到本地，同时删除邮件。是接收邮件的协。 |
| IMAP            | Internet Mail Access Protocol, 支持文件夹功能                                    |
| Exchange server | 微软提供的邮件服务                                                               |

## python 案例

```python
import smtplib

smtp = smtplib.SMTP("mail.sap.corp", 587)
smtp.starttls() # this seting is required
smtp.login("sf-pla-usermanagement-authentication-mails","Authentication1")
smtp.sendmail('noreply+sf_pla_usermanagement_authentication@sap.corp', 'jiabin.zheng01@sap.com', message)
smtp.quit()
```

```config
spring.mail.host=mail.sap.corp
spring.mail.port=587
spring.mail.username=sf-pla-usermanagement-authentication-mails
spring.mail.password=Authentication1
spring.mail.properties.mail.smtp.auth=true
spring.mail.properties.mail.smtp.starttls.enable=true
spring.mail.properties.mail.smtp.starttls.required=true
spring.mail.properties.mail.smtp.from=noreply+sf_pla_usermanagement_authentication@sap.corp
```