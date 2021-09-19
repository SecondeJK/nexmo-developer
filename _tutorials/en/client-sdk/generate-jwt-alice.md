---
title: Generate a JWT
description: In this step you learn how to generate a valid JWT for your Client SDK Application.
---

# Generate a JWT

The Client SDK uses [JWTs](/concepts/guides/authentication#json-web-tokens-jwt) for authentication. The JWT identifies the user name, the associated application ID and the permissions granted to the user. It is signed using your private key to prove that it is a valid token.

> **NOTE**: We'll be creating a one-time use JWT on this page for testing. In production apps, your server should expose an endpoint that generates a JWT for each client request.

You are generating a JWT using the Vonage CLI by running the following command but remember to replace the `APP_ID` variable with your own value:

``` shell
vonage jwt --key_file=./private.key --acl='{"paths":{"/*/users/**":{},"/*/conversations/**":{},"/*/sessions/**":{},"/*/devices/**":{},"/*/image/**":{},"/*/media/**":{},"/*/applications/**":{},"/*/push/**":{},"/*/knocking/**":{},"/*/legs/**":{}}}' --subject=Alice --app_id=APP_ID
```

The generated JWT will be valid for the next 6 hours.

![](/screenshots/tutorials/client-sdk/generated-jwt-key.png)

## Further information

* [online JWT generator](/jwt)
* [JWT guide](/concepts/guides/authentication#json-web-tokens-jwt)
