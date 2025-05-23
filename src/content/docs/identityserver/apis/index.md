---
title: Protecting APIs
description: Learn how to secure and protect your APIs using Duende IdentityServer's token-based authentication and authorization
date: 2020-09-10T08:20:20+02:00
sidebar:
  label: Overview
  order: 1
redirect_from:
  - /identityserver/v5/apis/
  - /identityserver/v5/apis/add_apis/
  - /identityserver/v6/apis/
  - /identityserver/v6/apis/add_apis/
  - /identityserver/v7/apis/
  - /identityserver/v7/apis/add_apis/
---

Duende IdentityServer issues tokens for accessing resources.

These resources are very often HTTP-based APIs, but could be also other "invocable" functionality like messaging endpoints, gRPC services or even good old XML Web Services. See the [issuing tokens](/identityserver/tokens/) section on more information on access tokens and how to request them.

## Adding API Endpoints to IdentityServer

It's a common scenario to add additional API endpoints to the application hosting IdentityServer.
These endpoints are typically protected by IdentityServer itself.

For simple scenarios, we give you some helpers. See the advanced section to understand more of the internal plumbing.

:::note
You could achieve the same by using either Microsoft's `JwtBearer` handler. But this requires more configuration and creates dependencies on external libraries that might lead to conflicts in future updates.
:::

Start by registering your API as an `ApiScope`, (or resource) e.g.:

```cs
var scopes = new List<ApiScope>
{
    // local API
    new ApiScope(IdentityServerConstants.LocalApi.ScopeName),
};
```

...and give your clients access to this API, e.g.:

```cs
new Client
{
    // rest omitted
    AllowedScopes = { IdentityServerConstants.LocalApi.ScopeName },   
}
```

:::note
The value of `IdentityServerConstants.LocalApi.ScopeName` is `IdentityServerApi`.
:::

To enable token validation for local APIs, add the following to your IdentityServer startup:

```cs
// Program.cs
builder.Services.AddLocalApiAuthentication();
```

To protect an API controller, decorate it with an `Authorize` attribute using the `LocalApi.PolicyName` policy:

```cs
[Route("localApi")]
[Authorize(LocalApi.PolicyName)]
public class LocalApiController : ControllerBase
{
    public IActionResult Get()
    {
        // omitted
    }
}
```

Authorized clients can then request a token for the `IdentityServerApi` scope and use it to call the API.

## Discovery
You can also add your endpoints to the discovery document if you want, e.g.like this::

```cs
// Program.cs
builder.Services.AddIdentityServer(options =>
{
    options.Discovery.CustomEntries.Add("local_api", "~/localapi");
})
```

## Advanced
Under the covers, the `AddLocalApiAuthentication` helper does a couple of things:

* adds an authentication handler that validates incoming tokens using IdentityServer's built-in token validation engine (the name of this handler is `IdentityServerAccessToken` or `IdentityServerConstants.LocalApi.AuthenticationScheme`
* configures the authentication handler to require a scope claim inside the access token of value `IdentityServerApi`
* sets up an authorization policy that checks for a scope claim of value `IdentityServerApi`

This covers the most common scenarios. You can customize this behavior in the following ways:

* Add the authentication handler yourself by calling `services.AddAuthentication().AddLocalApi(...)`
    * this way you can specify the required scope name yourself, or (by specifying no scope at all) accept any token from the current IdentityServer instance
* Do your own scope validation/authorization in your controllers using custom policies or code, e.g.:


```cs
// Program.cs
builder.Services.AddAuthorization(options =>
{
    options.AddPolicy(IdentityServerConstants.LocalApi.PolicyName, policy =>
    {
        policy.AddAuthenticationSchemes(IdentityServerConstants.LocalApi.AuthenticationScheme);
        policy.RequireAuthenticatedUser();
        // custom requirements
    });
});
```

## Claims Transformation
You can provide a callback to transform the claims of the incoming token after validation.
Either use the helper method, e.g.:

```cs
// Program.cs
builder.Services.AddLocalApiAuthentication(principal =>
{
    principal.Identities.First().AddClaim(new Claim("additional_claim", "additional_value"));

    return Task.FromResult(principal);
});
```

...or implement the event on the options if you add the authentication handler manually.


