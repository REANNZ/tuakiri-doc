# Multi-Factor Authentication with REFEDS MFA profile

For many use cases, the authentication requirements are increasing and Multi-Factor Authentication (MFA) is being required.

In SAML, the authentication method is expressed with the authentication context class reference ( `AuthnContextClassRef` ) - in both requests and responses.

To provide a vendor-agnostic (and technology-agnostic) value for expressing that MFA was used, suitable for R&E identity federations, REFEDS has developed the REFEDS MFA Profile: [https://refeds.org/profile/mfa](https://refeds.org/profile/mfa) (this is both an identifier of the profile AND a documentation link).

This profile can be:

1.  Requested by Service Providers to initiate an MFA login.
2.  Received by IdPs to enforce the MFA login (if not configured by default anyway)
3.  Asserted by IdPs to confirm MFA was indeed used as part of the login.
4.  Checked by SPs to only allow a session if MFA is asserted by the IdP.

The following pages provide documentation for:

*   [IdPs to configure support for MFA](https://reannz.atlassian.net/wiki/spaces/Tuakiri/pages/3815538737/Configuring+REFEDS+MFA+on+Shibboleth+IdP) (enforcing and asserting)
*   [SPs to configure support for MFA](https://reannz.atlassian.net/wiki/spaces/Tuakiri/pages/3815538751/Configuring+REFEDS+MFA+on+Shibboleth+SP) (requesting and checking)

To test the overall experience, the following links initiate a login to the Tuakiri Attribute Validator with MFA requested and checked (and will thus fail with IdPs not supporting MFA):

*   Tuakiri (PROD) Attribute Validator with MFA: [https://attributes.tuakiri.ac.nz/auth/mfalogin](https://attributes.tuakiri.ac.nz/auth/mfalogin)
*   Tuakiri-TEST Attribute Validator with MFA: [https://attributes.test.tuakiri.ac.nz/auth/mfalogin](https://attributes.test.tuakiri.ac.nz/auth/mfalogin)