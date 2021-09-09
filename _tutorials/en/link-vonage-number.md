---
title: Link a Vonage number
description: In this step you learn how to link a Vonage number to your application.
---

# Link a Vonage number

## Using the Dashboard

1. Find your application in the [Dashboard](https://dashboard.nexmo.com/voice/your-applications).
2. Click on the application in the Your Applications list. Then click on the Numbers tab.
3. Click the Link button to link a Vonage number with that application.

## Using the Vonage CLI

To link a number to your Vonage application run the following command.  Replace `YOUR_VONAGE_NUMBER` your Vonage number and `APPLICATION_ID` with your application id:

```
vonage apps:link APPLICATION_ID --number=YOUR_VONAGE_NUMBER
```
