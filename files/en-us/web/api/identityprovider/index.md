---
title: IdentityProvider
slug: Web/API/IdentityProvider
page-type: web-api-interface
status:
  - experimental
browser-compat: api.IdentityProvider
---

{{APIRef("FedCM API")}}{{SeeCompatTable}}{{SecureContext_Header}}

The **`IdentityProvider`** interface of the [Federated Credential Management (FedCM) API](/en-US/docs/Web/API/FedCM_API) represents an {{glossary("Identity provider", "IdP")}} and provides access to related information and functionality.

{{InheritanceDiagram}}

## Static methods

- {{domxref("IdentityProvider.close_static", "close()")}} {{experimental_inline}}
  - : Provides a manual signal to the browser that an IdP sign-in flow is finished. This is needed to, for example, close the IdP sign-in dialog when sign-in is completely finished and the IdP has finished collecting data from the user.
- {{domxref("IdentityProvider.getUserInfo_static", "getUserInfo()")}} {{experimental_inline}}
  - : Returns information about a previously-signed in user on their return to an IdP, which can be used to provide a personalized welcome message and sign-in button.

## Examples

### Basic `IdentityProvider.getUserInfo()` usage

The following example shows how the {{domxref("IdentityProvider.getUserInfo_static", "getUserInfo()")}} method can be used to return information on a previously-signed in user from a specific IdP.

```js
// Iframe displaying a page from the https://idp.example origin
const userInfo = await IdentityProvider.getUserInfo({
  configURL: "https://idp.example/fedcm.json",
  clientId: "client1234",
});

// IdentityProvider.getUserInfo() returns an array of user information.
if (userInfo.length > 0) {
  // Returning accounts should be first, so the first account received
  // is guaranteed to be a returning account
  const name = userInfo[0].name;
  const givenName = userInfo[0].given_name;
  const displayName = givenName || name;
  const picture = userInfo[0].picture;
  const email = userInfo[0].email;

  // …

  // Render a personalized sign-in button using the information returned above
}
```

## Specifications

{{Specifications}}

## Browser compatibility

{{Compat}}

## See also

- [Federated Credential Management API](https://privacysandbox.google.com/cookies/fedcm) on privacysandbox.google.com (2023)
