---
redirect_from: Configuring+REFEDS+MFA+on+Shibboleth+SP
id: service_providers/configuring_refeds_mfa_on_shibboleth_sp
---
# Configuring REFEDS MFA on Shibboleth SP
{:.no_toc}

The REFEDS MFA Profile: [https://refeds.org/profile/mfa](https://refeds.org/profile/mfa) is the vendor-agnostic approach to [signalling Multi-Factor Authentication](../multi-factor_authentication_with_refeds_mfa_profile) in R&E identity federations like Tuakiri (and among such federations when connected to [eduGAIN](edugain_resources/index)).

A Service Provider (SP) can signal the MFA requirement by including an AuthnContextClassRef with value of `[https://refeds.org/profile/mfa](https://refeds.org/profile/mfa)` in the SAML SSO requests.

An Identity Provider (IdP) responds to such requests by enforcing an MFA login - and confirming that by including the REFEDS MFA AuthnContextClassRef in the SAML response.

The SP still needs to check the response AuthnContextClassRef - as in many cases, it would be possible to alter the login flow and bypass MFA.

This page (drawing on the [upstream MFA documentation](https://shibboleth.atlassian.net/wiki/spaces/SP3/pages/2114781453/Requiring+Multi-Factor+Authentication)) documents the configuration steps required on Shibboleth SP.

1. TOC
{:toc}

# Requesting REFEDS MFA

Requesting a specific authentication method depends on how the session is initiated - whether explicitly (by redirecting to the Session Initiator) or implicitly (by accessing a page requiring a Shibboleth session).

## Requesting REFEDS MFA in explicitly initiated sessions

In the redirect to the Session Initiator (typically `/Shibboleth/Login` ), include the query string parameter `authnContextClassRef=https://refeds.org/profile/mfa` in the URL.

Example: [https://attributes.tuakiri.ac.nz/Shibboleth.sso/Login?target=/auth/login&authnContextClassRef=https://refeds.org/profile/mfa](https://attributes.tuakiri.ac.nz/Shibboleth.sso/Login?target=/auth/login&authnContextClassRef=https://refeds.org/profile/mfa)

## Requesting REFEDS MFA in implicitly initiated sessions

In your Apache config requesting a Shibboleth session, include the `authnContextClassRef` [content setting](https://shibboleth.atlassian.net/wiki/spaces/SP3/pages/2065334723/ContentSettings) with value `https://refeds.org/profile/mfa` .

Example:

```
    <Location /auth/login>
        AuthType shibboleth
        require shibboleth
        ShibRequestSetting requireSession true
        ShibRequestSetting authnContextClassRef https://refeds.org/profile/mfa
    </Location>
```

# Check REFEDS MFA

The above so far only includes REFEDS MFA in the SSO request sent to the IdP.  It is important to also check that it is included in the IdP response.

This can be done with the Apache `require` directive - in the simplest case replacing the stub `require shibboleth` used in the example above.  (If multiple `require` directives were needed, they'd have to be wrapped in a `RequireAll` block to join them with a logical **and** , otherwise Apache would default to a logical **or** ).

If the session is established with a different `authnContextClassRef` value (which, if REFEDS MFA was _requested_ in the SSO request would be a misconfiguration of the IdP, or deliberate tinkering with the session setup), Apache will block further access with an _HTTP 401 Authorization Required_ error page.

We recommend customising the error response page to make it clear login was rejected because MFA was not used.

The following example extends the above sample code requesting MFA to also check REFEDS MFA and overrides the 401 error message: 

```
    <Location /auth/login>
        AuthType shibboleth 
        require authnContextClassRef https://refeds.org/profile/mfa
        ShibRequestSetting requireSession true
        ShibRequestSetting authnContextClassRef https://refeds.org/profile/mfa
        ErrorDocument 401 "<b>Access denied</b>: Multi-Factor Authentication is required to access this resource"
    </Location>
```

# MFA-specific error handling

When a SAML SSO request asking for REFEDS MFA (via an `authContextClassRef` ) is sent to an IdP not supporting the specific authentication method, the IdP will send back a SAML response indicating a login failure, which will be rendered by the SP as a generic error message.

![](https://reannz.atlassian.net/wiki/download/attachments/3815538751/ShibbolethSP-sessionError-generic.png?api=v2)

To improve the user experience, we recommend to customise the error handler.  The easiest way is to replace the `/etc/shibboleth/sessionError.html` template with a customised one provided here: [https://raw.githubusercontent.com/REANNZ/Tuakiri-public/master/shibboleth-sp/mfa/sessionError.html](https://raw.githubusercontent.com/REANNZ/Tuakiri-public/master/shibboleth-sp/mfa/sessionError.html)

This template customises the original one by:

*   including a note that the error likely occurred because MFA was requested by the SP but not supported by the IdP
*   recommending to contact the user's organisation IT service desk
*   providing contact details for the IdP (if available in the IdP's metadata)

![](https://reannz.atlassian.net/wiki/download/attachments/3815538751/ShibbolethSP-sessionError-MFA.png?api=v2)

Replacing `/etc/shibboleth/sessionError.html` will work well for simple sites that request REFEDS MFA for all sessions.  The missing support for MFA will be the most likely cause for an IdP to report an error back to the SP.

It is also possible to apply this template only for specific URLs, or use a custom error handling page that the SP would redirect to.  For further information, please see the [upstream documentation on error handling](https://shibboleth.atlassian.net/wiki/spaces/SP3/pages/2065334361/Errors) or contact [tuakiri@reannz.co.nz](mailto:tuakiri@reannz.co.nz) .

# Further reading

The documentation above covers the simple case.  Further information is available in the upstream documentation on [Requiring MFA on Shibboleth SP](https://shibboleth.atlassian.net/wiki/spaces/SP3/pages/2114781453/Requiring+Multi-Factor+Authentication) and in the [SP configuration documentation](https://shibboleth.atlassian.net/wiki/spaces/SP3/).

Please get in touch with [tuakiri@reannz.co.nz](mailto:tuakiri@reannz.co.nz) to request further advice with your planned deployment.
