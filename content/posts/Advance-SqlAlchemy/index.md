+++
title = 'وقتی SQLAlchemy دیتای قدیمی را به ما برگرداند؟'
date = 2026-09-26T18:38:00+03:30
draft = false
slug = 'advanced-sqlalchemy'
type = 'article'
tags = ['database','SQLAlchemy','Session','امیرمسعود خردمندی']
description = 'بررسی رفتار SQLAlchemy Session و Identity Map و تفاوت populate_existing با session.refresh در شرایطی که داده‌های دیتابیس تغییر کرده‌اند.'  
categories=["database"]
author = "امیرمسعود خردمندی"
+++

یه اتفاق خیلی عجیب برای ما توی پروداکشن افتاد که ساعت یک شب برای بخش خرید **ارور 500** گرفتیم.

حالا موضوع چی بود؟

فروشگاه ما به این شکل بود که محصولات از طریق یه **Provider** آپدیت موجودی و قیمت میشدن. تقریباً آپدیت هر محصول 5 ثانیه طول میکشید.

ما توی فروشگاهمون یه سری محصولات خیلی ارزون هم داشتیم که کمتر از چند سنت بودن و هفته‌ای یک یا دوبار شارژ میشدن.

یه فیچری هم زده بودیم که مشتری‌هامون میتونستن بگن:

> هر موقع این محصول موجود شد برامون خرید رو بزن.

چون خیلی سریع هم تموم میشد.

ما هم می‌دیدیم اگه موجودی والتش اندازه بود، می‌خریدیم براش و فعالش می‌کردیم روی اکانتش.

حالا این وسط یه مشتری ما با موجودی بر فرض **5$** اومده بود یه خرید بزنه و اشتراک بخره.

در لحظه‌ای که میاد اشتراک رو بخره، یه تسک دیگه میاد از حساب والت چند سنت خرید رو کم میکنه و اون سرویس هفتگی رو براش فعال میکنه.

تقریباً:

```text
11 سنت
```

حالا این وسط چه اتفاقی می‌افته؟

این دوتا Request یه‌جورایی پشت سر هم اتفاق می‌افتن.

و Wallet کاربر توی دیتابیس شده:

```text
$4.89
```

ولی خرید اون محصول مورد نظر مشتری بوده:

```text
$4.99
```

از اون جالب‌تر اینکه Query ما هم داشته می‌گفته:

```text
$5.00
```

در جایی که اصلاً نمی‌تونسته اگه موجودی کافی نبوده به این قسمت برسه!

حالا بریم بررسی کنیم داستان چی بوده.

## مشکل از SQLAlchemy Session بود

یه موضوعی هست توی SQLAlchemy در رابطه با Sessionها.

برنامه‌نویس این سرویس بنده‌خدا فکر می‌کرده وقتی داره می‌نویسه:

```python
stmt = (
    select(Wallet, User)
    .join(User, User.id == Wallet.user_id)
    .where(Wallet.user_id == user_id)
)

result = await session.execute(stmt)
wallet, user = result.one()


time.sleep(10)  # PROCESS

stmt = (
    select(Wallet, User)
    .join(User, User.id == Wallet.user_id)
    .where(Wallet.user_id == user_id)
)

result = await session.execute(stmt)
wallet, user = result.one()

```

این یعنی SQLAlchemy دوباره داره Query میزنه به دیتابیس و مقدار جدید رو می‌گیره.

اما امان از دل غافل!

شما وقتی توی یه Session یه Object رو دارین و قبلاً گرفتین، اگه دوباره Query بزنین، SQLAlchemy می‌بینه این Object رو توی Session خودش داره.

در نتیجه ممکنه همون Object موجود در Session رو برگردونه، بدون اینکه مقدار Attributeهای اون Object رو از دیتابیس دوباره پر کنه.

مثلاً:

```text
Database:
balance = 4.89

Session:
wallet.balance = 5.00
```

در این حالت شما ممکنه دوباره Query بزنین، اما همچنان این مقدار رو ببینید:

```python
wallet.balance
# 5.00
```

در حالی که مقدار واقعی دیتابیس:

```text
4.89
```

است.

## استفاده از `populate_existing=True`

برای چنین شرایطی می‌تونیم از:

```python
.execution_options(populate_existing=True)
```

استفاده کنیم:

```python
stmt = (
    select(Wallet, User)
    .join(User, User.id == Wallet.user_id)
    .where(Wallet.user_id == user_id)
    .execution_options(
        populate_existing=True
    )
)

result = await session.execute(stmt)
wallet, user = result.one()

```

اینجا به SQLAlchemy می‌گیم نتایج این Query رو حتی اگه Object مربوطه از قبل داخل Session وجود داشته باشه، دوباره روی Object موجود اعمال کن.

بنابراین اگر در این فاصله موجودی Wallet تغییر کرده باشه:

```text
قبل:

Session:
$5.00

Database:
$5.00
```

و یک Request دیگه موجودی رو تغییر بده:

```text
Database:
$4.89
```

Query با:

```python
.execution_options(
    populate_existing=True
)
```

باعث میشه مقدار Object موجود در Session هم دوباره با مقدار Query پر بشه:

```text
Session:
$4.89

Database:
$4.89
```

## ولی `session.refresh(obj)` چی؟

شاید در لحظه بگین:

> `session.refresh(obj)` هم می‌تونستیم استفاده کنیم.

که درست هم هست.

ولی این دوتا یه فرق اساسی دارن.

وقتی میای:

```python
await session.refresh(wallet)
```

میگی:

> عزیز من، برو همین `obj` مشخص رو دوباره از دیتابیس بخون و آپدیتش کن.

یعنی تمرکز روی همون Object مشخصه.

ولی وقتی از:

```python
.execution_options(
    populate_existing=True
)
```

استفاده می‌کنی، داری میگی:

> نتایج این Query رو حتی اگه Object مربوطه از قبل توی Session وجود داشته باشه، برو بگیر و دوباره روی Object موجود اعمال کن.

پس به شکل ساده:

```text
session.refresh(obj)
        ↓
همین Object مشخص رو دوباره از Database بخون
```

در مقابل:

```text
populate_existing=True
        ↓
نتایج این Query رو روی Objectهای موجود در Session هم اعمال کن
```

## یک نکته مهم

بماند که این پروژه رو ما چندین ماه داشتیم ریفکتور می‌کردیم و این اتفاق به دلیل وحشتناک بودن و مزخرف بودن Source Code افتاده بود.

در کل مهمه که توی SQLAlchemy حواستون باشه که دیتا داره از Objectی که توی Session هست خونده میشه یا Query واقعاً داره مقدار مورد نظر رو از سمت Database می‌گیره.

مخصوصاً وقتی با شرایطی مثل این‌ها روبه‌رو هستید:

* چند Request همزمان
* Background Task
* چند Worker
* تغییر داده توسط Transactionهای مختلف
* Queueها
* تغییر اطلاعات خارج از Session فعلی

این موارد می‌تونن باعث بشن چیزی که داخل Object می‌بینید با چیزی که واقعاً داخل Database وجود داره متفاوت باشه.

در چنین شرایطی باید بدونید SQLAlchemy Session دقیقاً چه رفتاری با Objectهایی که قبلاً Load شدن داره.

اگه خوشتون اومده باشه، بعداً میتونیم بیشتر راجع بهش صحبت کنیم.



من صفحه های دیگه ایی هم دارم:

[💬 Telegram Channel](https://t.me/djangoup)

[🎥 YouTube](https://www.youtube.com/@MasoudKheradmandi)