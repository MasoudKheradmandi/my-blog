+++
title = 'PartialIndex چیست؟'
date = 2026-09-18T18:48:45+03:30
draft = false
slug="what-is-partial-index"
type = 'article'
tags = ['database','partialindex','RDBSM','امیرمسعود خردمندی']
description = 'partial index چیست و چه اهمیتی دارد؟'
categories=["database"]
author = "امیرمسعود خردمندی"
+++

### ایندکس جزئی (Partial Index) چیست؟


ایندکس جزئی یکی از قابلیت‌های پیشرفته در پایگاه‌های داده‌ است که به شما امکان می‌دهد تنها بخشی از رکوردهای جدول را ایندکس کنید، نه تمام آن‌ها(بهتر بگم ینی یک value خاص رو ایندکس میکنیم) این نوع ایندکس با استفاده از یک شرط خاص (مانند یک عبارت WHERE) تعیین می‌شود و فقط رکوردهایی که شرط ذکر شده را دارند اوورده میشود


❗️مزایای ایندکس جزئی:

- **عملکرد بهینه‌تر**: ایندکس جزئی حجم کمتری دارد و بنابراین جستجوهای مرتبط با شرط خاص سریع‌تر انجام می‌شوند.

- **کاهش فضای ذخیره‌سازی:** به دلیل این‌که تنها بخشی از داده‌ها ایندکس می‌شوند، فضای کمتری در پایگاه داده مصرف می‌شود.

- **بهینه برای شرایط خاص:** اگر کوئری‌های شما معمولاً شامل یک شرط خاص (مثلاً status = True) باشه ، ایندکس جزئی میتونه خیلی مؤثر باشه

*برای مثال شما یک Table Product رو تصور کنید که یک ستون active داره و شما هر دفعه اونایی که True هستن رو نمایش میدین*

یه نمونه کد در جنگو هم ببینیم

```python

class YourModel(models.Model):
    .
    .
    active = models.BooleanField()

    class Meta:
        indexes = [
            models.Index(fields=['active'], name='active_true_idx', condition=models.Q(active=True)),
        ]

```




من امیر مسعود خردمندی هستم و صفحه های دیگه ایی هم دارم:

[💬 Telegram Channel](https://t.me/djangoup)

[🎥 YouTube](https://www.youtube.com/@MasoudKheradmandi)