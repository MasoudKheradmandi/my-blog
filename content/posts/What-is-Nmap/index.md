+++
title = 'Nmap چیست؟ آموزش اسکن پورت و IP با Nmap'
date = 2026-09-20T22:21:18+03:30
draft = false
description = 'Nmap چیست؟ در این مقاله با Nmap، اسکن پورت، شناسایی IPهای فعال شبکه و تشخیص سرویس‌های در حال اجرا آشنا می‌شویم.'
categories = ['شبکه']
author = "Masoud Kheradmandi"
tags = ['Nmap','شبکه','امیرمسعود خردمندی']
slug='what-is-nmap'
keywords = [
    'Nmap',
    'Nmap چیست',
    'آموزش Nmap',
    'اسکن پورت',
    'Port Scanning',
    'اسکن IP',
    'Network Scanning',
    'شبکه'
]
+++


# **nmap** چیه؟

خیلی راحت و ساده بگم یه ابزار اوپن سورسه برای اسکن کردن **ایپی** و **پورت** و **تشخیص** اینکه چه **برنامه** هایی نصب هستن

## اسکن پورت

اسکن پورت که خب تکلیفش خیلی مشخصه

چه پورت هایی روی سرور بازن؟

مثلا پستگرس `5432`
ردیس `6379`

بهمون میگه این پورت های ip مورد نظر بازه

**دستورش؟**

```bash
nmap 192.168.1.10
```

**خروجی:**

```text
22/tcp   open   ssh
80/tcp   open   http
443/tcp  open   https
```

---

## حالا اسکن کردن ip چیه؟

شاید گیج بشین که ینی چی اسکن کردن IP

هیچی عزیز من:)

داره میگه چه ایپی هایی در این شبکه قابل دسترسن

**دستورش؟**

```bash
nmap -sn PRIVATE_IP/24
```

**خروجی اش چی میشه؟**

```text
Nmap scan report for 192.168.1.1
Host is up

Nmap scan report for 192.168.1.5
Host is up

Nmap scan report for 192.168.1.20
Host is up
```

مثلا برای خودم گوشی و لپتاپ و سیستمم رو اوورد (همشون به مودمم وصلن)

اون 24 اخر هم یادتون نره بزارین —> بعدا توضیح میدم چیه :)

---

## تشخیص برنامه هایی که نصب هستن چیه؟

چیزی که من متوجه شدم  بعد از تشخیص پورت ها با دستور اول

حدس میزنه چه اپ هایی نصب هستن روی سرور و قرار نیست اگه میگه مثلا postgresql رو تشخیص نداده قطعا اون وجود نداره

صرفا پورتش بسته اس

مثلا روی سایت بنده خدایی همینجوری تست گرفتم ,*امیدوارم حلال کنه :)*, نتیجه اش این بوده

```text
nmap brsapi.ir
Starting Nmap 7.94SVN ( https://nmap.org ) at 2026-09-20 22:09 +0330
Nmap scan report for brsapi.ir (185.206.229.81)
Host is up (0.033s latency).
rDNS record for 185.206.229.81: bita.mahanserver.net
Not shown: 900 filtered tcp ports (no-response), 1 filtered tcp ports (host-unreach), 88 closed tcp ports (conn-refused)
PORT    STATE SERVICE
21/tcp  open  ftp
25/tcp  open  smtp
53/tcp  open  domain
80/tcp  open  http
110/tcp open  pop3
143/tcp open  imap
443/tcp open  https
465/tcp open  smtps
587/tcp open  submission
993/tcp open  imaps
995/tcp open  pop3s

Nmap done: 1 IP address (1 host up) scanned in 154.00 seconds
```


*من مسعود هستم و اگه از این مقاله خوشتون اومد برای دوستاتون هم بفرستین*



[💬 Telegram Channel](https://t.me/djangoup)

[🎥 YouTube](https://www.youtube.com/@MasoudKheradmandi)

created_by : Masoud Kheradmandi