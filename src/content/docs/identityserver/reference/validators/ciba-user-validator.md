---
title: "Backchannel Authentication User Validator"
description: Documentation for the IBackchannelAuthenticationUserValidator interface which is used to validate request hints and identify the user for CIBA authentication requests.
sidebar:
  order: 30
redirect_from:
  - /identityserver/v5/reference/validators/ciba_user_validator/
  - /identityserver/v6/reference/validators/ciba_user_validator/
  - /identityserver/v7/reference/validators/ciba_user_validator/
---

#### Duende.IdentityServer.Validation.IBackchannelAuthenticationUserValidator

The `IBackchannelAuthenticationUserValidator` interface is used to validate request hints and identify the user for whom
the [CIBA](/identityserver/ui/ciba) request is intended.
To use CIBA, you are expected to implement this interface and register it in the ASP.NET Core service provider.

## IBackchannelAuthenticationUserValidator APIs

* **`ValidateRequestAsync`**

  Validates the backchannel login request with the provided `BackchannelAuthenticationUserValidatorContext` for the
  current request.
  Returns a `BackchannelAuthenticationUserValidationResult` object.

### BackchannelAuthenticationUserValidatorContext

Models the information to validate and identity the user for a CIBA login request.

* **`Client`**

  The `Client` making the request.

* **`LoginHintToken`**

  The login hint request parameter from the request.

* **`IdTokenHint`**

  The id token hint request parameter from the request.

* **`IdTokenHintClaims`**

  The claims contained in the validated id token hint from the request.

* **`LoginHint`**

  The login hint request parameter from the request.

* **`UserCode`**

  The user code request parameter from the request.

* **`BindingMessage`**

  The binding request parameter from the request.

### BackchannelAuthenticationUserValidationResult

Models the result of a CIBA login request.

* **`Subject`**

  The `ClaimsPrincipal` that represents the user that was successfully identified for the login request.
  This must contain the user's `"sub"` claim.

* **`Error`**

  The error if the user validation failed.

* **`ErrorDescription`**

  The error description if the user validation failed.
