---
  title:  یک پی

language_tabs: # must be one of https://git.io/vQNgJ
  - shell
  - ruby
  - python
  - javascript
  - php

toc_footers:
  - <a href='#'>Sign Up for a Developer Key</a>
  - <a href='https://github.com/slatedocs/slate'>Documentation Powered by Slate</a>

includes:
  - errors
  - images
  - image_tag


search: true

code_clipboard: true
---

# پیش نیازها

این سند حاوی پروتکل هایی است که از متدهای پرداخت به شرح ذیل استفاده می کند:
متد پرداخت یک پی:
متد پرداخت یک پی، پروتکل هایی را که مابین کسب و کارها ) یا ارایه دهندگان خدمات پرداخت( و یک پی هستند،تایید می کند:

**Payment Request** متد درخواست پرداخت، تراکنش را با داده های کاربر شروع می کند. اگر داده ها در وضعیت صحیح و یا مورد قبول باشند، مشتری به صفحه پرداخت ارجاع داده می شود تا بتواند اطلاعات کارت شتاب یا کارت بین المللی خود را وارد کند.

**Verify Payment** از متد تایید پرداخت، بلافاصله پس از متد درخواست و جهت تایید تراکنش استفاده می شود. اگر داده ها و عملیات پرداخت در وضعیت صحیح و یا مورد قبول باشد، وضعیت تراکنش از در حالت انتظار به تایید، تغییر می کند.

**Exchange Rate** از این متد برای تبدیل ارز بین دو واحد مختلف استفاده می شود. اگر داده ها در وضعیت صحیح و یا مورد قبول باشد، آخرین نرخ بین دو ارز نمایش داده خواهد شد.

**Get User Location** از این متد برای دریافت اطلاعات مکانی کاربران از روی آی پی استفاده می کنیم.


# نمای کلی
پروتکل درگاه پرداخت یک پی می تواند توسط هرگونه شخص ثالثی برای بررسی اطلاعات سفارش و تایید شروع پرداخت
ها مورد استفاده قرار گیرد.

در ابتدا، یک پی باید درخواست **Payment Request** را با ارجاع خریدار به وبسایت کسب و کار برای ثبت سفارش
جدید دریافت کند.

زمانی که خریدار فرآیند ثبت درخواست را از سمت یک پی به پایان می رساند، بسته به نحوه پیاده سازی متد پرداخت،
ممکن است برای تکمیل پرداخت به صفحه شخص ثالث یا فروشنده فراخوانده شود.

همچنین فروشنده می تواند اطلاعات سفارش را قبل از شارژ خریدار در سیستم یک پی بررسی و متد **Payment Verify**
را صدا کند.

## نمودار

![overview,img](/images/content/overviewFa.png)

# 1 . تسویه حساب مشتری

## توضیحات
زمانی که مشتری در صفحه تسویه حساب سایت شما باشد، باید امکان ورود اطلاعات هویتی )از قبیل نام، نام خانوادگی،
شماره موبایل و ایمیل( و اطلاعات صورتحساب )از قبیل آدرس، کشور، شهر و کدپستی( را داشته باشد تا بتواند پرداختی
سریع و امن انجام دهد.

## نمودار

![Customer Checkout,img](/images/content/customerCheckoutFa.png)



# 2 . درخواست پرداخت

## لینک درخواست
[https://gate.yekpay.com/api/payment/request](https://gate.yekpay.com/api/payment/request)

## متد
POST

## ورودی 

PARAMETERS       | DESCRIPTION                                      | EXAMPLE
---------------- |------------------------------------------------- | --------------------------------
merchantId       | 32-digits merchant code                          | XXXXXXXXXXXXXXXXXXXX
amount           | Amount of your order (with Decimal (15,2)format) | 799.00
fromCurrencyCode | Origin currency code                             | 978
toCurrencyCode   | Destination currency code                        | 364
orderNumber      | Unique order id for each merchant                | 125548
callback         | Callback URL of merchant website                 | https://example.com/callback.php
firstName        | First name of your customer                      | John
lastName         | Last name of your customer                       | Doe
email            | Email of your customer                           | test@example.com
mobile           | Mobile of your customer                          | +44123456789
address          | Billing address                                  | Alhamida st Al ras st
postalCode       | Billing postal code                              | 64785
country          | United Arab Emirates                             | Billing country
city             | Billing city                                     | Dubai
description      | Name of your products or your services           | Apple mac book air 2017

## نمودار

![Payment Request,img](/images/content/paymentRequestFa.png)

<aside class="warning">در نظر داشته باشید تمامی پارامترها به جز " **description** " الزامی هستند.</aside>


# 3۰ اعتبار سنجی تراکنش

## توضیحات

‫پس از فراخوانی متد درخواست، پاسخی با فرمت JSON محتوی کد، توضیحات و فیلدهای اعتبارسنجی دریافت خواهید کرد. در صورت دریافت کد عدد 100 ، می توانید وارد مرحله بعد شوید.‬

## کدهای خروجی

CODE    | DESCRIPTION                             | AUTHORITY
--------| --------------------------------------- | ---------------
-1      | The parameters are incomplete           | 0
-2      | Merchant code is incorrect              | 0
-3      | Merchant code is not active             | 0
-4      | Currencies is not valid                 | 0
-5      | Maximum/Minimum amount is not valid     | 0
-6      | Your IP is restricted                   | 0
-7      | Order id must be unique                 | 0
-100    | Unknown error                           | 0
100     | Success                                 | XXXXXXXXXXXX

## نمودار

![Payment Authorization,img](/images/content/paymentAuthorizationFaa.png)

# 4 . شروع فرآیند تراکنش

## توضیحات
اگر در مرحله قبل پیغام "موفقیت آمیز" دریافت کرده اید، می توانید با کد **Authority** که در متد درخواست دریافت کردهاید و فراخوانی آدرس زیر، فرآیند پرداخت را شروع کنید.

## آدرس

[https://gate.yekpay.com/api/payment/start/{AUTHORITY}](https://gate.yekpay.com/api/payment/start/{AUTHORITY})

<aside class="notice">
درنظر داشته باشید {AUTHORITY} موجود در لینک فوق باید با authority دریافتی شما جایگزین شود.
</aside>

## نمودار

![Start Payment,img](/images/content/startPaymentFa.png)

# 5 . پردازش تراکنش

## توضیحات

در این مرحله فرآیند تراکنش )توسط کارت شتاب یا کارت بین المللی( توسط درگاه پرداخت ما پردازش می شود. اطلاعات ارزهای قابل استفاده ) 9 ارز ی که توسط ما پشتیبانی می شون د(، در ضمیمه های بعد در دسترس خواهند بود.
لطفا در نظر داشته باشید که تمامی درگاه های پرداخت بین المللی ما از گزینه  3 D-secure پشتیبانی می کنند و این گزینه نیز باید برای کارت مشتری شما فعال باشد.

زمانی که فرآیند تراکنش تکمیل شد، )با یا بدون خطا( اعتبارسنجی و وضعیت تراکنش با متد **POST** به آدرسی که در متد درخواست توسط شما فرستاده شده بود، ارسال می شود.

## نمودار

![Payment Processing,img](/images/content/paymentProcessingFa.png)



# 6۰ تاکید تراکنش

## آدرس 
[https://gate.yekpay.com/api/payment/verify](https://gate.yekpay.com/api/payment/verify)

## متد
POST

## ورودی

PARAMETERS       | DESCRIPTION                                          | EXAMPLE
---------------- | ---------------------------------------------------- | ----------------------
merchantId       | 32-digits merchant code                              | XXXXXXXXXXXXXXXXXXXX
authority        | Authority code that you get before in request method | 115162456765

## نمودار

![Payment Verification,img](/images/content/paymentVerificationFa.png)

# 7۰ اطلاعات تراکنش

## توضیحات
در این مرحله پاسخی از سمت متد تایید تراکنش با فرمت JSON دریافت خواهید کرد، که اگر این کد برابر " 100 " باشد، درخواست تایید شما موفقیت آمیز بوده اطلاعاتی مانند **Reference** ، **Gateway**, **OrderNo** و **Amount** دریافت خواهید کرد.

## کدهای خروجی

CODE    | DESCRIPTION                             | REFERENCE
--------| --------------------------------------- | ---------------
-1      | The parameters are incomplete           | 0
-2      | Merchant code is incorrect              | 0
-3      | Merchant code is not active             | 0
-8      | Currencies is not valid                 | 0
-9      | Maximum/Minimum amount is not valid     | 0
-10     | Your IP is restricted                   | 0
-100    | Unknown error                           | 0
100     | Success                                 | XXXXXXXXXXXX

## نمودار

![Payment Information,img](/images/content/paymentInformationFa.png)


# استعلام نرخ ارز

## توضیحات

## آدرس
[https://gate.yekpay.com/api/payment/exchange](https://gate.yekpay.com/api/payment/exchange)

## متد 
POST 

## ورودی ها

PARAMETERS       | DESCRIPTION                                                                      | EXAMPLE
---------------- | -------------------------------------------------------------------------------- | ----------------------
merchantId       | 32-digits merchant code that you can get from Yekpay after submit payment gateway| XXXXXXXXXXXXXXXXXXXX
FromCurrencyCode | Currency code with ISO 8581 according to currency table                          | 978
ToCurrencyCode   | Currency code with ISO 8581 according to currency table                          | 364

## کدهای خروجی

CODE    | DESCRIPTION                             | AUTHORITY
--------| --------------------------------------- | ---------------
-1      | The parameters are incomplete           | 0
-2      | Merchant code is incorrect              | 0
-3      | Merchant code is not active             | 0
-4      | Currencies is not invalid               | 0
-100    | Unknown error                           | 0
100     | Success                                 | 0.14000

<aside class="success">
در صورت موفق بودن شما اطلاعات اضافی مانند **Rate** را دریافت می کنید که نرخ تبدیل بین دو ارز مورد نیاز شما می باشد.
</aside>

## دریافت اطلاعات مکانی کاربر

# توضیحات

به دلیل در دسترس نبودن درگاه پرداخت یک پی در تمامی کشورها، شما می توانید از این API به واسطه IP کاربر برای تشخیص موقعیت مکانی کاربر جهت شروع عملیات تراکنش، استفاده کنید 

## آدرس
[https://gate.yekpay.com/api/payment/country](https://gate.yekpay.com/api/payment/country)

## متد
POST

##  ورودی ها
PARAMETERS    | DESCRIPTION                             | EXAMPLE
--------------| --------------------------------------- | ---------------
ip            | User IP address                         | 78.46.162.156

## کدهای خروجی

CODE    | DESCRIPTION                             | REFERENCE
--------| --------------------------------------- | ---------------
-1      | The parameters are incomplete           | 
-11     | Your IP isn’t valid                     | 
-12     | Your IP is unknown                      | 
-100    | Unknown error                           | 
100     | Success                                 | DE

<aside class="success">
در صورت موفق بودن درخواست شما ئارامتر اضافی با نام **Country** دریافت می کنید که کشور درخواست دهنده را با فرمت دو حرفی مشخص می کند.
</aside>

## ضمیمه اول: کشورها

Currency    | Name                        | Code
------------| ----------------------------| ---------------
EUR         | Euro                        | 978
IRR         | Iranian Rial                | 364
CHF         | Switzerland Franc           | 756
AED         | United Arab Emirates Dirham | 784
CNY         | Chinese Yuan                | 156
GBP         | British Pound               | 826
JPY         | Japanese 100 Yens           | 392
RUB         | Russian Ruble               | 643
TRY         | Turkish New Lira            | 494

## ضمیمه دوم : منابع

You can visit Yekpay GitHub page in order to access sample codes (e.g. PHP, C#, Python).

Also, we have some plugins for popular CMS like **WooCommerce, Magento, Prestashop, etc**.

