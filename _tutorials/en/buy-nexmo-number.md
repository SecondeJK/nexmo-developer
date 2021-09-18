---
title: Buy a Vonage number
description: In this step you learn how to purchase a Vonage number.
---

# Buy a Vonage number 

## Using the Dashboard

First you can browse [your existing numbers](https://dashboard.nexmo.com/your-numbers).

If you have no spare numbers you can [buy one](https://dashboard.nexmo.com/buy-numbers).

## Using the Vonage CLI

You can purchase a number using the Vonage CLI. The following command purchases an available number in the US. Specify [an alternate two-character country code](https://www.iban.com/country-codes) to purchase a number in another country.

```bash
$ nexmo number:buy --country_code US --confirm
```

Make a note of the number returned, as you'll need it later.
