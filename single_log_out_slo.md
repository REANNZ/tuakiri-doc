# Single Log Out (SLO)

Single Log Out (SLO) is a concept parallel to the Single Sign On (SSO) concept: if a user can seamlessly establish sessions with a number of services, there should also be a way to seamlessly terminate such sessions.

However, there is a number of gotchas in SLO - including issues like temporarily unreachable services, and terminating application-level sessions derived from the original session.  Please refer to the upstream Shibboleth Project documentation on [SLO Issues](https://wiki.shibboleth.net/confluence/display/SHIB2/SLOIssues) for further information.

The Shibboleth IdP (versions 2.4.0+) supports at least a minimalist SLO implementation:

*   It is possible to terminate the session at the IdP, so that no further SP sessions can be established.
*   It is possible to initiate logout at an SP where the user has a current session.  The SP can send an SLO message to the IdP and terminate the session there as well.
*   However, the IdP will not be propagating the SLO to any additional SPs.
*   By default, the SLO message from the SP to the IdP is _asynchronous_ and the flow ends at the IdP Logout page.
*   The IdP Logout page displays the list of SPs the user has accessed _from within this IdP session_ - and informs the user that the only secure way to close all sessions is to close the browser window.
*   It is also possible to do a synchronous SP to IdP SLO flow that redirects back to the SP, where the SP can either display a message confirming the SP and IdP sessions have been terminated, or can redirect the user to an application-level page confirming the successful logout.

Nonetheless, except for the case where the user has established a session with only one SP where the session (including application-level if used) has been successfully terminated, the only reliable way to close all sessions is to close the browser window.  Details on this minimalist implementation are available at [https://wiki.shibboleth.net/confluence/display/SHIB2/IdPEnableSLO](https://wiki.shibboleth.net/confluence/display/SHIB2/IdPEnableSLO).

# SLO configuration on a Service Provider

No configuration is necessary and on Shibboleth SP implementations version 2.5.0+, SLO works out of the box.

The only work required for basic functionality to work is adding a Logout button to the application pointing to the initiator (see the section on [Initiating Single Logout](https://reannz.atlassian.net/wiki/spaces/Tuakiri/pages/3815539051#SingleLogOut(SLO)-InitiatingSingleLogout) below).

The only configuration work to be done would be tweaking the Logout Initiator for advanced deployment scenarios:

*   Changing the list of SLO protocol bindings supported (and their order of priority).
*   Switching between asynchronous (default) and synchronous mode.

Both of these can be set as attributes on the `<Logout>` element in `/etc/shibboleth/shibboleth2.xml`. They get passed as attributes to the SAML2 LogoutInitiator that gets created by the Logout element.  The fully unfolded configuration with settings identical to default is:

```
 <Logout asynchronous="true" outgoingBindings="urn:oasis:names:tc:SAML:2.0:bindings:HTTP-Redirect urn:oasis:names:tc:SAML:2.0:bindings:HTTP-POST urn:oasis:names:tc:SAML:2.0:bindings:SOAP">SAML2 Local</Logout>
```

*   Setting `asynchronous="false"` would make the flow return back to the SP (this otherwise only happens for the SOAP binding which cannot be done asynchronously).
*   Changing the list of bindings and their order affects which SLO protocol gets used - the first binding that is also advertised in the IdP metadata gets used.

For further information, please see the following documentation from the Shibboleth Project wiki:

*   NativeSPLogoutInitiator: [https://wiki.shibboleth.net/confluence/display/SHIB2/NativeSPLogoutInitiator](https://wiki.shibboleth.net/confluence/display/SHIB2/NativeSPLogoutInitiator)
*   NativeSPServiceLogout: [https://wiki.shibboleth.net/confluence/display/SHIB2/NativeSPServiceLogout](https://wiki.shibboleth.net/confluence/display/SHIB2/NativeSPServiceLogout)
*   NativeSPSingleLogoutService: [https://wiki.shibboleth.net/confluence/display/SHIB2/NativeSPSingleLogoutService](https://wiki.shibboleth.net/confluence/display/SHIB2/NativeSPSingleLogoutService)

# SLO configuration on an Identity Provider

Please see the [relevant section in the IdP installation manual](https://reannz.atlassian.net/wiki/spaces/Tuakiri/pages/3815538790/Installing+a+Shibboleth+2.x+IdP#InstallingaShibboleth2.xIdP-ConfiguringSingleLogout).

Please note that if the above steps have NOT been done for an IdP, then:

*   SP-initiated SLO will not work: the SP will report an error message saying that the IdP is not advertising SLO endpoints in the metadata - and would revert to local logout only.
*   IdP-initiated logout (terminating just the session at the IdP) would still work on IdPs version 2.4.0+ (assuming the profile handler is enabled, which holds for configuration file templates shipped with 2.4.0+ - but the handler might not be enabled if the IdP is running with configuration files created by an earlier version of the IdP software).
    *   Note also that if the logout page has not been branded, the user would end up on a page that very clearly says "This page must be branded" - so do not advertise the SLO endpoint URLs in federation metadata and do not advertise the Logout URL to the users unless you have branded the page.

# Initiating Single Logout

*   To terminate just the session at the IdP (and see the list of SPs accessed within this session), point your browser to a URL of this form on your IdP: https://idp.inst.ac.nz/idp/profile/Logout
*   To initiate SLO at an SP, go to a URL on the SP of the form: https://sp.example.org/Shibboleth.sso/Logout
*   The URL can be also customized with a return address to redirect to (instead of displaying the confirmation screen).  However, by default, the SLO would use an asynchronous message to the IdP and the flow would end at the IdP Logout page.  The user would be returned to the return URL only if the SLO is done in synchronous mode and the flow returns back to the SP.  To set the return URL, pass it in the `return` parameter as a query string to the Logout initiator - e.g.: https://sp.example.org/Shibboleth.sso/Logout?return=https://sp.example.org/logout-completed.html