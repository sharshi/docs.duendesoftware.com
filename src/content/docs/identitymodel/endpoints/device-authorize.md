---
title: Device Authorization Endpoint
description: Documentation for OAuth 2.0 device flow authorization endpoint using HttpClient extension methods
sidebar:
  order: 7
  label: Device Authorization
redirect_from:
  - /foss/identitymodel/endpoints/device_authorize/
---

The client library for the [OAuth 2.0 device
flow](https://tools.ietf.org/html/rfc7662) device authorization is
provided as an extension method for *HttpClient*.

The following code sends a device authorization request:

```csharp
var client = new HttpClient();

var response = await client.RequestDeviceAuthorizationAsync(new DeviceAuthorizationRequest
{
    Address = "https://demo.duendesoftware.com/connect/device_authorize",
    ClientId = "device"
});
```

The response is of type *DeviceAuthorizationResponse* and has properties
for the standard response parameters. You also have access to the
raw response and to a parsed JSON document (via the *Raw* and
*Json* properties).

Before using the response, you should always check the *IsError*
property to make sure the request was successful:

```csharp
if (response.IsError) throw new Exception(response.Error);

var userCode = response.UserCode;
var deviceCode = response.DeviceCode;
var verificationUrl = response.VerificationUri;
var verificationUrlComplete = response.VerificationUriComplete;
```
