+++

title = 'CSRF چیست و چرا اهمیت دارد؟'

slug = 'what-is-csrf'

date = 2026-09-29T12:26:21+03:30

type = 'article'

tags = ['CSRF', 'Django', 'امنیت وب', 'امنیت']

draft = false

categories = ['امنیت']

description = 'CSRF چیست؟ در این مقاله با حملات Cross-Site Request Forgery، نحوه انجام آن‌ها، دلیل آسیب‌پذیری وب‌سایت‌ها و روش محافظت در Django آشنا می‌شویم.'

author = 'امیرمسعود خردمندی'
+++

## Csrf چیست و چرا اهمیت دارد؟

این مقاله رو بخاطر این نوشتم که تا مدت زیادی حقیقتا خودمم نمیدونستم چرا باید از **CSRF** استفاده کنم بنابراین تصمیم گرفتم خیلی مفصل تر راجع بهش صحبت کنم

بریم از پایه توضیح اش بدیم:

* فرض کن تو در سایت masoudkheradmandi.ir لاگین کردی

*هرچند ما اصلا login نداریم تو سایت امون :)*

بعد من برای شمای کاربر در مرورگرت یه session قرار میدم

```text
sessionid=abc123
```

حالا شما وارد یه سایت دیگه میشی

```text
hell.com
```

و حالا اون سایت یه فرم داره به این شکل:

```html
<form action="https://masoudkheradmandi.ir/change-email/" method="POST">
    <input type="hidden" name="email" value="hacker@gmail.com">
</form>

<script>
    document.forms[0].submit();
</script>
```

وقتی این صفحه توسط مرورگر شما باز بشه، جاوااسکریپت فرم رو submit می‌کنه و یک درخواست `POST` به `masoudkheradmandi.ir` ارسال می‌شه.

نکته‌ی مهم اینجاست:

مرورگر شما هنگام ارسال این درخواست، **cookie مربوط به `masoudkheradmandi.ir` را هم همراه درخواست ارسال می‌کند.**

در نتیجه سرور ممکنه درخواست رو چیزی شبیه این ببینه:

```http
POST /change-email/ HTTP/1.1
Host: masoudkheradmandi.ir
Cookie: sessionid=abc123

email=hacker@gmail.com
```

و این برای سرور یعنی درخواست واقعا از سمت کاربر لاگین شده اومده

و به همین راحتی ایمیل کاربر لاگین شده تغییر کرد.

به اینکار **Cross-Site Request Forgery** که همون CSRF هستش

> یعنی سایت مخرب کاری می‌کند که مرورگر تو، بدون اینکه واقعاً قصدش را داشته باشی، یک درخواست معتبر به سایت موردنظر بفرستد.


نکته:
سایت ها به کوکی همدیگه دسترسی ندارن و مستقیما نمیتونن کوکی همدیگه رو بخونن
بخاطر
***Same-Origin Policy*** مرورگر.

حالا فهمیدم CSRF چیه
### بریم ببینیم Django چجوری با CSRF_TOKEN جلوی این حمله رو میگیره؟

شما وقتی مینویسی:
‍‍
```html
<form method="POST">
    {% csrf_token %}

    <input type="text" name="username">

    <button type="submit">
        Submit
    </button>
</form>
```

جنگو یه hidden input در فرم قرار میده

```text
<input type="hidden"
       name="csrfmiddlewaretoken"
       value="abcXYZ123...">
```

پس درخواست چیزی شبیه این می‌شود:

```
POST /change-email/

Cookie:
sessionid=...
csrftoken=abcXYZ123...

Body:
csrfmiddlewaretoken=abcXYZ123...
email=test@example.com
```

#### سوال اینه چرا هکر نمیتونه این درخواست رو شبیه سازی کنه؟

فرض رو بر این بزار سایت hell.com این درخواست رو بفرسته

```html
<form action="https://masoudkheradmandi.ir/change-email/" method="POST">

    <input type="hidden"
           name="csrfmiddlewaretoken"
           value="???">

</form>
```

هکر که نمیدونه Token واقعی چیه
بنابراین جنگو بهش میگه:
> CSRF token is missing or incorrect.

و درخواست رد میشه

در واقع CSRF Token داره میگه این درخواست POST از صفحه خودمون اومده و ما تولیدش کردیم و میتونیم بهش اعتماد کنیم

CSRF Token برای احراز هویت کاربر نیست.
فقط داره میگه

> *این درخواست احتمالاً از یک صفحه‌ی معتبر همین سایت آمده است.*

حالا سوالی که هست اینه
#### جنگو در چه قسمتی این مورد رو هندل میکنه؟

در فایل settings.py

```python
MIDDLEWARE = [
    ...
    'django.middleware.csrf.CsrfViewMiddleware',
    ...
]
```
این middleware کاملا درخواست های وورودی رو بررسی میکنه و میتونین این مورد رو خودتون بخونین




اگه خوشتون اومده باشه، بعداً میتونیم بیشتر راجع بهش صحبت کنیم.



من صفحه های دیگه ایی هم دارم:

[💬 Telegram Channel](https://t.me/djangoup)

[🎥 YouTube](https://www.youtube.com/@MasoudKheradmandi)