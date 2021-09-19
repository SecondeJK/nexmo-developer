---
title: Generate JWTs
description: In this step you learn how to generate valid JWTs for each User in your In-App Voice Call.
---

# Generate JWTs

You need to generate a JWT for each user. The JWT is used to authenticate the user. Run the following commands, remember to replace the `APP_ID` variable with id of your application.

For Alice:

``` shell
vonage jwt --key_file=./private.key --acl='{"paths":{"/*/users/**":{},"/*/conversations/**":{},"/*/sessions/**":{},"/*/devices/**":{},"/*/image/**":{},"/*/media/**":{},"/*/applications/**":{},"/*/push/**":{},"/*/knocking/**":{},"/*/legs/**":{}}}' --subject=Alice --app_id=APP_ID
```

And for Bob:

``` shell
vonage jwt --key_file=./private.key --acl='{"paths":{"/*/users/**":{},"/*/conversations/**":{},"/*/sessions/**":{},"/*/devices/**":{},"/*/image/**":{},"/*/media/**":{},"/*/applications/**":{},"/*/push/**":{},"/*/knocking/**":{},"/*/legs/**":{}}}' --subject=Bob --app_id=APP_ID
```

The above commands set the expiry of the JWT to one day from now, which is the maximum.

Make a note of the JWT you generated for each user:

![](/screenshots/tutorials/client-sdk/generated-jwt-key.png)

> **NOTE**: In a production environment, your application should expose an endpoint that generates a JWT for each client request.

## Further information

* [JWT guide](/concepts/guides/authentication#json-web-tokens-jwt)
