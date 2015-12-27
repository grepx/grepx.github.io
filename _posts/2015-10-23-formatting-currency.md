---
layout: post
title:  Formatting In-App Purchase currency in Android
date:   2015-10-24 21:49:16
categories: android ui java play in-app purchase date format
---
In most cases you shouldn't have to manually format In-App Purchase prices in an Android app. Assuming you are using Google Play for your in-app purchases you should just use the `price` property of an [SkuDetails record][getSkuDetails] from the [`getSkuDetails()`][getSkuDetails] method.

However, sometimes the need will arise to manually format the currency value received back from Google Play. In my case, the marketing guys wanted to take a yearly subscription price and then divide it by 12 to get a less scary monthly calculation of the price to show the user.

This can be easily done using the `price_currency_code` property and Java's [`Currency`][javaCurrency] and [`NumberFormat`][javaNumberFormat] classes like so:

```java
double price = skuDetails.priceValue / 12d;
Currency currency = Currency.getInstance(skuDetails.currency);
NumberFormat format = NumberFormat.getCurrencyInstance();
format.setCurrency(currency);
String formattedCurrency = format.format(price);
```

Under the hood `getCurrencyInstance()` initialises using the default [`Locale`][javaLocale] for the device, so the currency will be formatted properly according to the user's locale, which may not be quite what you expect.

For instance, British Pounds are formatted as £1.00 on a device running a English (British English) locale, but 1,00 GBP on a Spanish (Spain) locale, since they don't know what a £ sign means. This the correct behavior though, and it's the same as the value that Google Play returns in this situation.

In case you're wondering; the library I am using to call the Google Play In-App Purchase API here is the [android-inapp-billing-v3][inAppBilling] project, a lightweight, developer friendly wrapper for the rather arcane API spec that Google publishes in their documentation.

[getSkuDetails]:      http://developer.android.com/google/play/billing/billing_reference.html#getSkuDetails
[javaCurrency]:   http://docs.oracle.com/javase/7/docs/api/java/util/Currency.html
[javaNumberFormat]: http://docs.oracle.com/javase/7/docs/api/java/text/NumberFormat.html
[inAppBilling]: https://github.com/anjlab/android-inapp-billing-v3
[javaLocale]: http://docs.oracle.com/javase/7/docs/api/java/util/Locale.html
