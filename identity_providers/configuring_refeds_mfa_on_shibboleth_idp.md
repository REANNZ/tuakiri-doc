# Configuring REFEDS MFA on Shibboleth IdP

The REFEDS MFA Profile: [https://refeds.org/profile/mfa](https://refeds.org/profile/mfa) is the vendor-agnostic approach to [signalling Multi-Factor Authentication](https://reannz.atlassian.net/wiki/spaces/Tuakiri/pages/3815538745/Multi-Factor+Authentication+with+REFEDS+MFA+profile) in R&E identity federations like Tuakiri (and among such federations when connected to [eduGAIN](https://reannz.atlassian.net/wiki/spaces/Tuakiri/pages/3815539055/eduGAIN+resources)).

Service Providers signal the MFA requirement by including an AuthnContextClassRef with value of `https://refeds.org/profile/mfa` in the SSO requests.

When receiving such value, the IdP needs to enforce an MFA login flow (if not enabled by default), and upon successful MFA authentication, include the REFEDS MFA AuthnContextClassRef in the SAML response.

How this is done very much depends how how authentication is done on the IdP - and what MFA method the IdP has access to.

/\*<!\[CDATA\[\*/ div.rbtoc1693282355471 {padding: 0px;} div.rbtoc1693282355471 ul {list-style: disc;margin-left: 0px;} div.rbtoc1693282355471 li {margin-left: 0px;padding-left: 0px;} /\*\]\]>\*/

*   1 [Meeting REFEDS MFA Criteria](#ConfiguringREFEDSMFAonShibbolethIdP-MeetingREFEDSMFACriteria)
*   2 [REFEDS MFA on IdP proxying to AzureAD](#ConfiguringREFEDSMFAonShibbolethIdP-REFEDSMFAonIdPproxyingtoAzureAD)
*   3 [REFEDS MFA on IdP with Password authentication and MFA](#ConfiguringREFEDSMFAonShibbolethIdP-REFEDSMFAonIdPwithPasswordauthenticationandMFA)
*   4 [REFEDS MFA on IdP with no MFA](#ConfiguringREFEDSMFAonShibbolethIdP-REFEDSMFAonIdPwithnoMFA)

# Meeting REFEDS MFA Criteria

The REFEDS MFA profile asks that "two of the four distinct types of factors" (defined in ITU-T X.1254) are used and that "factors used are independent" (access to one factor does not by itself grant access to other factors).

Please read the Criteria in the REFEDS MFA profile at [https://refeds.org/profile/mfa](https://refeds.org/profile/mfa) before proceeding to assert the REFEDS MFA by your IdP.

Further supporting material is also available at [https://wiki.refeds.org/display/PRO/MFA+Profile+FAQ](https://wiki.refeds.org/display/PRO/MFA+Profile+FAQ)

# REFEDS MFA on IdP proxying to AzureAD

Some IdPs are configured as a SAML proxy, using the SAML authentication flow (introduced in IdP 4.x) to pass the SSO request to an upstream IdP (acting as an SP towards the upstream IdP).

Typically (in deployments seen in Tuakiri), the upstream IdP is Microsoft AzureAD.

AzureAD supports conceptually same functionality (trigger MFA via AuthnContextClassRef, issue the AuthnContextClassRef when MFA is used) - but uses a vendor-specific value, `https://schemas.microsoft.com/claims/multipleauthn` .

Configuring REFEDS MFA on an IdP proxying to AzureAD simply means:

*   Mapping REFEDS MFA authnContextClassRef in incoming SSO requests to the Microsoft vendor-specific value to be passed on in the proxied SSO request.
*   Mapping Microsoft vendor-specific authnContextClassRef value in incoming SSO responses to the REFEDS MFA value to be included in the SSO response sent to the original requester.
*   Configuring the SAML authentication flow as supporting the REFEDS MFA value.

Based on [contributed upstream documentation](https://shibboleth.atlassian.net/wiki/spaces/KB/pages/1467056889/Using+SAML+Proxying+in+the+Shibboleth+IdP+to+connect+with+Azure+AD#UsingSAMLProxyingintheShibbolethIdPtoconnectwithAzureAD-ProxyTask6.HandlingREFEDSAuthnContextRequests(optional)), the exact steps are:

*   Edit `/opt/shibboleth-idp/conf/authn/authn-comparison.xml` and:
    *    First, check that your version of the file has entries with ids `shibboleth.PrincipalProxyRequestMappings` and `shibboleth.PrincipalProxyResponseMappings` .  
        *   If not (your IdP has an outdated copy of this file, likely from initial IdP 3.x install), replace the file with `/opt/shibboleth-idp/dist/conf/authn/authn-comparison.xml` .
        *   It is unlikely any customisation would have been done to his file and replacing with an up-to-date stock copy is perfectly fine.
    *   Insert the following entry into the `shibboleth.PrincipalProxyRequestMappings`  list: 
        
        ```
                <entry>
                     <key>
                        <bean parent="shibboleth.SAML2AuthnContextClassRef"
                              c:classRef="https://refeds.org/profile/mfa" />
                    </key>
                    <list>
                        <bean parent="shibboleth.SAML2AuthnContextClassRef"
                              c:classRef="http://schemas.microsoft.com/claims/multipleauthn" />
                    </list>
                </entry>
        ```
        
    *   Insert the following entry into the `shibboleth.PrincipalProxyResponseMappings`  list: 
        
        ```
                <entry>
                     <key>
                        <bean parent="shibboleth.SAML2AuthnContextClassRef"
                              c:classRef="http://schemas.microsoft.com/claims/multipleauthn" />
                    </key>
                    <list>
                        <bean parent="shibboleth.SAML2AuthnContextClassRef"
                              c:classRef="https://refeds.org/profile/mfa" />
                    </list>
                </entry>
        ```
        
*   Add the following setting into `/opt/shibboleth-idp/conf/authn/authn.properties` (adding REFEDS MFA to the default list of supported authentication methods): 
    
    ```
    idp.authn.SAML.supportedPrincipals = \
        saml2/https://refeds.org/profile/mfa, \
        saml2/urn:oasis:names:tc:SAML:2.0:ac:classes:PasswordProtectedTransport, \
        saml2/urn:oasis:names:tc:SAML:2.0:ac:classes:Password, \
        saml1/urn:oasis:names:tc:SAML:1.0:am:password
    ```
    

# REFEDS MFA on IdP with Password authentication and MFA

If the IdP handles authentication locally (via the `Password`  flow) and has an additional flow for MFA, supporting REFEDS MFA should be possible and would involve:

*   triggering the MFA flow based on REFEDS MFA (in addition to existing triggers)
    *   this may involve mapping REFEDS MFA to different value, or using it as-is
*   configuring the MFA flow to set REFEDS MFA upon successful completion
*   configuring the Password flow as supporting REFEDS MFA (in a similar way as done above for the SAML flow)

Please get in touch with [tuakiri@reannz.co.nz](mailto:tuakiri@reannz.co.nz) to work out the specific details

  

# REFEDS MFA on IdP with no MFA

Sorry, your IdP needs to support MFA first.  But, if your user base is synchronised to AzureAD and MFA is already available there, one suitable solution might be to switch your IdP to proxy authentication to AzureAD - turning it into the first, easy case covered above.

Please get in touch with [tuakiri@reannz.co.nz](mailto:tuakiri@reannz.co.nz) to get started.
