---
  title:  yekpay

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

# Prerequisites

This protocol can be used with the following payment methods: Yekpay Payment, Yekpay
Payment Verify The following communication protocols should already be in place between
merchant (or PSP) and Yekpay:

**PR -- Payment Request**, calling request method with necessary data to initiate a transaction. If the data is in the correct/expected format, the customer is redirected to payment page, where he has to enter the SHETAB or Master/Visa card data.

**VP -- Verify Payment**, calling verify method with necessary data to verify a transaction. If the data is in the correct/expected format, transaction status changed from pending to
complete.

**ER -- Exchange Rate**, calling exchange method with necessary data to get exchange rate. If
the data is in the correct/expected format, last rate between two currencies are showed.

# Overview

This protocol can be used by any third party to check order information and approve payments initiated by our merchants.

First of all, Yekpay must receive a PR request with the buyer redirected from the merchant's website to initiate a new order. The payment method may or may not be specified in the PR request parameters.

The buyer finishes the order registration process on Yekpay's side and, depending on the payment
method implementation, he may be redirected to a third party/acquirer to make the payment.

Before charging the buyer, the acquirer may check the order information in Yekpay's system.

After the acquirer charges the buyer, it must also inform Yekpay about this operation by using the pay protocol so that Yekpay will update its order status and send the IPN to the merchant.

The "request" and "verify" requests are made server to server.

## Diagram

![overview,img](/images/content/overview.png)

# 1- Customer Checkout

## Description

When customer is in your checkout page, you must prepare basic (first name, last name, mobile email)and billing (address, country, city, postal code) info, to have a fast and secure payment.

## Diagram

![Customer Checkout,img](/images/content/customerCheckout.png)

# 2- Payment Request

## URL
[https://gate.yekpay.com/api/payment/request](https://gate.yekpay.com/api/payment/request)

## Method
POST

## Inputs

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

## Diagram

![Payment Request,img](/images/content/paymetRequest.png)

<aside class="warning">Please consider that all parameters except ‘description’ are required.</aside>


# 3- Payment Authorization

## Description

After calling request method you get response in JSON that has **Code**, **Description** and **Authority** fields, if you get Code 100, you can go to next step.

## Output Codes Table

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

## Diagram

![Payment Authorization,img](/images/content/paymentAuthorization.png)

# 4- Start Payment

## Description

If you get Success message in previous step, you can start payment by calling this URL with authority that you get in request method:

## URL

[https://gate.yekpay.com/api/payment/start/{AUTHORITY}](https://gate.yekpay.com/api/payment/start/{AUTHORITY})

<aside class="notice">You must replace {Authority} with your authority.</aside>

## Diagram

![Start Payment,img](/images/content/startPayment.png)

# 5- Payment Processing

## Description

In this step we process payment (SHETAB or Credit Card) with our Gateways, **9** currencies supported that you can see details in below appendices.

Please consider that all of gateways support 3D-secure, and your customer cards must support this
standard.

After Transaction completed (with or without errors), we send **Authority** and Status with **POST** method in your callback URL that you sent in request method. 

## Diagram

![Payment Processing,img](/images/content/paymentProcessing.png)

# 6- Payment Verification 

## URL 
[https://gate.yekpay.com/api/payment/verify](https://gate.yekpay.com/api/payment/verify)

## Method
POST

## Inputs

PARAMETERS       | DESCRIPTION                                          | EXAMPLE
---------------- | ---------------------------------------------------- | ----------------------
merchantId       | 32-digits merchant code                              | XXXXXXXXXXXXXXXXXXXX
authority        | Authority code that you get before in request method | 115162456765

## Diagram

![Payment Verification,img](/images/content/paymentVerification.png)

# 7- Payment Information

## Description

In this step you get response from verify method in JSON format, if the **Code** is equal to “100”, your verification request is successful and you will get other information like **Reference**, **Gateway** , **OrderNo** and **Amount**.

## Output Codes Table

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

## Diagram

![Payment Information,img](/images/content/paymentInformation.png)


# Request Exchange Rate

## Description

## URL
[https://gate.yekpay.com/api/payment/exchange](https://gate.yekpay.com/api/payment/exchange)

## Method 
POST 

## Input Table

PARAMETERS       | DESCRIPTION                                                                      | EXAMPLE
---------------- | -------------------------------------------------------------------------------- | ----------------------
merchantId       | 32-digits merchant code that you can get from Yekpay after submit payment gateway| XXXXXXXXXXXXXXXXXXXX
FromCurrencyCode | Currency code with ISO 8581 according to currency table                          | 978
ToCurrencyCode   | Currency code with ISO 8581 according to currency table                          | 364

## Output Table

CODE    | DESCRIPTION                             | AUTHORITY
--------| --------------------------------------- | ---------------
-1      | The parameters are incomplete           | 0
-2      | Merchant code is incorrect              | 0
-3      | Merchant code is not active             | 0
-4      | Currencies is not invalid               | 0
-100    | Unknown error                           | 0
100     | Success                                 | 0.14000

<aside class="success">
In successful case you get an additional parameter named Rate, that is the exchange rate between two currencies.
</aside>

## Get User Location

# Description

This API will be use in order to find user location based on user IP, you can call this API when you want to start a **payment transaction**, because Yekpay Payment Gateway Services isn’t available in all countries.

## URL
[https://gate.yekpay.com/api/payment/country](https://gate.yekpay.com/api/payment/country)

## Method
POST

## Input Table
PARAMETERS    | DESCRIPTION                             | EXAMPLE
--------------| --------------------------------------- | ---------------
ip            | User IP address                         | 78.46.162.156

## Output Table

CODE    | DESCRIPTION                             | REFERENCE
--------| --------------------------------------- | ---------------
-1      | The parameters are incomplete           | 
-11     | Your IP isn’t valid                     | 
-12     | Your IP is unknown                      | 
-100    | Unknown error                           | 
100     | Success                                 | DE

<aside class="success">
In successful case you get an additional parameter named Country, that is the 2-letter Country code.
</aside>

## Appendix A: Currencies

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

## Appendix B: Resources

You can visit Yekpay GitHub page in order to access sample codes (e.g. PHP, C#, Python).

Also, we have some plugins for popular CMS like WooCommerce, Magento, Prestashop, etc.

# Introduction

Welcome to the Kittn API! You can use our API to access Kittn API endpoints, which can get information on various cats, kittens, and breeds in our database.

We have language bindings in Shell, Ruby, Python, and JavaScript! You can view code examples in the dark area to the right, and you can switch the programming language of the examples with the tabs in the top right.

This example API documentation page was created with [Slate](https://github.com/slatedocs/slate). Feel free to edit it and use it as a base for your own API's documentation.

# Authentication

> To authorize, use this code:

```ruby
require 'kittn'

api = Kittn::APIClient.authorize!('meowmeowmeow')
```

```python
import kittn

api = kittn.authorize('meowmeowmeow')
```

```shell
# With shell, you can just pass the correct header with each request
curl "api_endpoint_here"
  -H "Authorization: meowmeowmeow"
```

```javascript
const kittn = require('kittn');

let api = kittn.authorize('meowmeowmeow');
```

> Make sure to replace `meowmeowmeow` with your API key.

Kittn uses API keys to allow access to the API. You can register a new Kittn API key at our [developer portal](http://example.com/developers).

Kittn expects for the API key to be included in all API requests to the server in a header that looks like the following:

`Authorization: meowmeowmeow`

<aside class="notice">
You must replace <code>meowmeowmeow</code> with your personal API key.
</aside>

# Kittens

## Get All Kittens

```ruby
require 'kittn'

api = Kittn::APIClient.authorize!('meowmeowmeow')
api.kittens.get
```

```python
import kittn

api = kittn.authorize('meowmeowmeow')
api.kittens.get()
```

```shell
curl "http://example.com/api/kittens"
  -H "Authorization: meowmeowmeow"
```

```javascript
const kittn = require('kittn');

let api = kittn.authorize('meowmeowmeow');
let kittens = api.kittens.get();
```

> The above command returns JSON structured like this:

```json
[
  {
    "id": 1,
    "name": "Fluffums",
    "breed": "calico",
    "fluffiness": 6,
    "cuteness": 7
  },
  {
    "id": 2,
    "name": "Max",
    "breed": "unknown",
    "fluffiness": 5,
    "cuteness": 10
  }
]
```

This endpoint retrieves all kittens.

### HTTP Request

`GET http://example.com/api/kittens`

### Query Parameters

Parameter | Default | Description
--------- | ------- | -----------
include_cats | false | If set to true, the result will also include cats.
available | true | If set to false, the result will include kittens that have already been adopted.

<aside class="success">
Remember — a happy kitten is an authenticated kitten!
</aside>

## Get a Specific Kitten

```ruby
require 'kittn'

api = Kittn::APIClient.authorize!('meowmeowmeow')
api.kittens.get(2)
```

```python
import kittn

api = kittn.authorize('meowmeowmeow')
api.kittens.get(2)
```

```shell
curl "http://example.com/api/kittens/2"
  -H "Authorization: meowmeowmeow"
```

```javascript
const kittn = require('kittn');

let api = kittn.authorize('meowmeowmeow');
let max = api.kittens.get(2);
```

> The above command returns JSON structured like this:

```json
{
  "id": 2,
  "name": "Max",
  "breed": "unknown",
  "fluffiness": 5,
  "cuteness": 10
}
```

This endpoint retrieves a specific kitten.

<aside class="warning">Inside HTML code blocks like this one, you can't use Markdown, so use <code>&lt;code&gt;</code> blocks to denote code.</aside>

### HTTP Request

`GET http://example.com/kittens/<ID>`

### URL Parameters

Parameter | Description
--------- | -----------
ID | The ID of the kitten to retrieve

## Delete a Specific Kitten

```ruby
require 'kittn'

api = Kittn::APIClient.authorize!('meowmeowmeow')
api.kittens.delete(2)
```

```python
import kittn

api = kittn.authorize('meowmeowmeow')
api.kittens.delete(2)
```

```shell
curl "http://example.com/api/kittens/2"
  -X DELETE
  -H "Authorization: meowmeowmeow"
```

```javascript
const kittn = require('kittn');

let api = kittn.authorize('meowmeowmeow');
let max = api.kittens.delete(2);
```

> The above command returns JSON structured like this:

```json
{
  "id": 2,
  "deleted" : ":("
}
```

This endpoint deletes a specific kitten.

### HTTP Request

`DELETE http://example.com/kittens/<ID>`

### URL Parameters

Parameter | Description
--------- | -----------
ID | The ID of the kitten to delete

