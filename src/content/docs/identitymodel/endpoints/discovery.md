---
title: Discovery Endpoint
description: Documentation for using the OpenID Connect discovery endpoint client library, including configuration, validation, and caching features
sidebar:
  order: 2
  label: Discovery
redirect_from:
  - /foss/identitymodel/endpoints/discovery/
---

The client library for the [OpenID Connect discovery
endpoint](https://openid.net/specs/openid-connect-discovery-1_0.html) is
provided as an extension method for `HttpClient`. The
`GetDiscoveryDocumentAsync` method returns a `DiscoveryResponse` object
that has both strong and weak typed accessors for the various elements
of the discovery document.

You should always check the `IsError` and `Error` properties before
accessing the contents of the document.

Example:

```csharp
var client = new HttpClient();

var disco = await client.GetDiscoveryDocumentAsync("https://demo.duendesoftware.com");
if (disco.IsError) throw new Exception(disco.Error);
```

Standard elements can be accessed by using properties:

```csharp
var tokenEndpoint = disco.TokenEndpoint;
var keys = disco.KeySet.Keys;
```

Custom elements (or elements not covered by the standard properties) can
be accessed like this:

```csharp
// returns string or null
var stringValue = disco.TryGetString("some_string_element");

// return a nullable boolean
var boolValue = disco.TryGetBoolean("some_boolean_element");

// return array (maybe empty)
var arrayValue = disco.TryGetStringArray("some_array_element");

// returns JToken
var rawJson = disco.TryGetValue("some_element");
```

### Discovery Policy

By default, the discovery response is validated before it is returned to the client, validation includes:

-   enforce that HTTPS is used (except for localhost addresses)
-   enforce that the issuer matches the authority
-   enforce that the protocol endpoints are on the same DNS name as the `authority`
-   enforce the existence of a keyset

Policy violation errors will set the `ErrorType` property on the
`DiscoveryResponse` to `PolicyViolation`.

All the standard validation rules can be modified using the
`DiscoveryPolicy` class, e.g. disabling the issuer name check:

```csharp
var disco = await client.GetDiscoveryDocumentAsync(new DiscoveryDocumentRequest
{
    Address = "https://demo.duendesoftware.com",
    Policy = 
    {
        ValidateIssuerName = false
    }
});
```

When the URIs in the discovery document are on a different base address than the issuer URI, you may encounter the error *Endpoint is on a different host than authority*.
For such scenario, additional endpoint base addresses can be configured:

```csharp
var disco = await client.GetDiscoveryDocumentAsync(new DiscoveryDocumentRequest
{
    Address = "https://demo.duendesoftware.com",
    Policy = 
    {
        AdditionalEndpointBaseAddresses = [ "https://auth.domain.tld" ]
    }
});
```

You can also customize validation strategy based on the authority with
your own implementation of `IAuthorityValidationStrategy`. By default,
comparison uses ordinal string comparison. To switch to `Uri` comparison:

```csharp
var disco = await client.GetDiscoveryDocumentAsync(new DiscoveryDocumentRequest
{
    Address = "https://demo.duendesoftware.com",
    Policy = 
    {
        AuthorityValidationStrategy = new AuthorityUrlValidationStrategy()
    }
});
```

### Caching The Discovery Document

You should periodically update your local copy of the discovery
document, to be able to react to configuration changes on the server.
This is especially important for playing nice with automatic key
rotation.

The `DiscoveryCache` class can help you with that.

The following code will set up the cache, retrieve the document the
first time it is needed, and then cache it for 24 hours:

```csharp
var cache = new DiscoveryCache("https://demo.duendesoftware.com");
```

You can then access the document like this:

```csharp
var disco = await cache.GetAsync();
if (disco.IsError) throw new Exception(disco.Error);
```

You can specify the cache duration using the `CacheDuration` property and also specify a custom discovery policy by
passing in a `DiscoveryPolicy` to the constructor.

### Caching And HttpClient Instances

By default, the discovery cache will create a new instance of `HttpClient` every time it needs to access the discovery
endpoint. You can modify this behavior in two ways, either by passing in a pre-created instance into the constructor,
or by providing a function that will return an `HttpClient` when needed.

The following code will set up the discovery cache in the ASP.NET Core service provider and will use the
`HttpClientFactory` to create clients:

```csharp
services.AddSingleton<IDiscoveryCache>(r =>
{
    var factory = r.GetRequiredService<IHttpClientFactory>();
    return new DiscoveryCache(Constants.Authority, () => factory.CreateClient());
});
```
