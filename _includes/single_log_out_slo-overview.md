
The Shibboleth IdP (versions 2.4.0+) supports at least a minimalist SLO implementation:

*   It is possible to terminate the session at the IdP, so that no further SP sessions can be established.
*   It is possible to initiate logout at an SP where the user has a current session.  The SP can send an SLO message to the IdP and terminate the session there as well.
*   However, the IdP will not be propagating the SLO to any additional SPs.
*   By default, the SLO message from the SP to the IdP is _asynchronous_ and the flow ends at the IdP Logout page.
*   The IdP Logout page displays the list of SPs the user has accessed _from within this IdP session_ - and informs the user that the only secure way to close all sessions is to close the browser window.
*   It is also possible to do a synchronous SP to IdP SLO flow that redirects back to the SP, where the SP can either display a message confirming the SP and IdP sessions have been terminated, or can redirect the user to an application-level page confirming the successful logout.

Nonetheless, except for the case where the user has established a session with only one SP where the session (including application-level if used) has been successfully terminated, the only reliable way to close all sessions is to close the browser window.  Details on this minimalist implementation are available at [https://wiki.shibboleth.net/confluence/display/SHIB2/IdPEnableSLO](https://wiki.shibboleth.net/confluence/display/SHIB2/IdPEnableSLO).

