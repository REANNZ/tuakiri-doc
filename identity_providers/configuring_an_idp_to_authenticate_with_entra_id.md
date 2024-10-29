---
id: identity_providers/configuring_an_idp_to_authenticate_with_entra_id
---

# Configuring an IdP to authenticate with Entra ID
{:.no_toc}

Traditionally, a Shibboleth IdP was configured with a login form asking for a
username and password that get checked against an on-prem LDAP server
(typically Active Directory).
While that works and is easy to set up, it does not easily support MFA.

In the meantime, most Tuakiri members have adopted Entra ID and use it for
accessing a number of applications.
The security settings on Entra ID also allow configuring MFA (as required or
optional) and MFA thus can become included in the authentication flow.

Shibboleth IdP (from version 4.2+) supports SAML proxying - and can authenticate
against an upstream SAML IdP (acting as a SAML SP for the upstream authentication).

Also, as the upstream SAML IdP (Entra ID) can pass attributes alongside the
authentication, the attribute resolver on the Tuakiri IdP can then be
configured to construct user attributes based on what was received from Entra ID,
instead of getting the attributes from a local LDAP server (Active Directory).
This would eliminate the dependency of the IdP on the local LDAP
server - but is an optional step and authentication can be moved to Entra ID
without changing the local lookup.

While the general aspects of the configuration changes required are covered in the
[Shibboleth IdP documentation SAML proxying documentation](https://shibboleth.atlassian.net/wiki/spaces/IDP5/pages/3199505973/SAMLAuthnConfiguration),
this page documents the specifics of configuring an IdP to authenticate against Entra ID (and also processing attributes if desired).

Note that SAML proxying to Entra ID is already used for all IdPs on the
[Tuakiri Hosted IdP](../tuakri_hosted_idp) service - this page makes the same
configuration techniques available to Tuakiri members still unning their own IdP.

# Registering IdP with Entra ID

The IdP needs to be registered with Entra ID as a Custom SAML application.

These instructions are based on
* Tuakiri instructions for configuring a [Tuakiri Hosted IdP with Entra ID](../tuakiri_hosted_idp/registering_tuakiri_hosted_idp_as_a_service_with_office_365_or_azure_ad) (with just slight changes for this context).
* Microsoft instructions for configuring a [Custom SAML application](https://docs.microsoft.com/en-gb/azure/active-directory/manage-apps/add-application-portal) in Entra ID (which also formed the basis for the Tuakiri HostedIdP instructions).

## Prerequisites

Before starting the process, you will need:

*   The entityID of the Hosted IdP instance.  This will likely be (with `example.org` replaced by your organisations domain):
    *   for Tuakiri-TEST: `https://idp-test.example.org/idp/shibboleth`
    *   for Tuakiri (Production): `https://idp.example.org/idp/shibboleth`
*   The assertion consumer service (ACS) URL (may be different if your IdP runs on a different base URL, possibly different from entityID base URL):
    *   for Tuakiri-TEST: `https://idp-test.example.org/idp/profile/Authn/SAML2/POST/SSO`
    *   for Tuakiri (Production): `https://idp.example.org/idp/profile/Authn/SAML2/POST/SSO`
*   Application Administrator privileges in your Entra ID / Azure portal account.
    *   This means one of the following roles: Global Administrator, Cloud Application Administrator, Application Administrator


## Registration

Entra  ID / Azure has a number of popular service preconfigured - however, to add the SP side of a Shibboleth IdP instance, we will be adding a Custom SAML application.

Please repeat this process twice, separately for TEST and PROD registration.

1.  Start from [https://portal.azure.com/](https://portal.azure.com/) and navigate to **Enterprise Applications**
    
    *   this can be done by searching for **Enterprise Applications** in the search box on the top of the screen
    *   or by selecting **All Services** from the top-left corner menu, and then selecting **Identity**, and then **Enterprise Applications**
2.  From the **Enterprise Applications** screen, click **New Application**.
    
3.  You will be presented with a list of pre-configured applications.
    
    *   Do not select from the list, instead, click **Create your own Application**
        
    *   and then select  3rd option: **Integrate any other application (Non-Gallery)**
        
    *   and enter a Name - e.g., **Tuakiri Login TEST** (for TEST) or **Tuakiri Login** (for PROD)
        
4.  At this point, the Application gets created and gets an Application ID and Object ID assigned
5.  Assign User Groups as appropriate
    *   This may be a group representing all users to the application (allow access to all users)
    *   If migrating from an on-prem LDAP server, the group should cover the same set of users as your existing LDAP base DN and search filter.
6.  Navigate back to the just-created Application via breadcrumbs at the top
7.  Select Setup Single Sign On, select SAML
8.  On the **Set up Single Sign-on with SAML**  page
    *   Edit **Basic SAML Configuration**: EntityID, Assertion Consumer Service (ACS) as per above
    *   Leave blank Sign on URL, Relay State, Logout URL
    *   User Attributes and Claims: the default configuration includes some core attributes/claims (`givenName`, `surname`, `email`, `userprincipalname`).
        However, if migrating from authenticating against on-prem LDAP with a username/password, additional attributes may be needed:
        *   As the `userprincipalname` (UPN) attribute is likely different from the username used with on-prem LDAP (UPN often takes the form `fistname.lastname@example.org`), to preserve the identifier attributes the users get when authenticating to services, **it is crucial** to also include an attribute corresponding to the on-prem username.
        *   If aiming to use the Entra ID login to replace the attribute lookup from on-prem LDAP, include also all attributes currently used from the on-prem LDAP.
        *   Consult `/opt/shibboleth-idp/conf/attribute-resolver.xml` to see what attributes are being used.
        *   Note that the attributes do not have to come in exactly the same format, as long as it is possible to derive the same outgoing values from the information received.
        *   For example, if the `staff` value of `eduPersonAffiliation` was derived from presence of a specific group name in the `memberOf` attribute, a suitable alternative may be to instead pass a boolean `isStaff` attribute, or pass `staff` value in a custom affiliation attribute.  (Or completely construct `eduPersonAffiliation` in Entra ID).
    *   Download Federation Metadata XML from SAML Signing Certificate section

For easier handling, we recommend processing the received metadata with `filter-idp-metadata.xslt` stylesheet that:
* strips out Entra-specific roles and only leaves in IDPSSODescriptor role.
* strips out `ds:Signature` element not needed for metadata trusted from a local file
* reformats the metadata for better readability

Please download the [filter-idp-metadata.xslt](https://github.com/REANNZ/Tuakiri-public/raw/refs/heads/master/scripts/filter-idp-metadata.xslt) and use it with:

    xsltproc -o metadata-filtered.xml filter-idp-metadata.xslt metadata-orig.xl


# Configuring authentication via SAML proxying

* Copy upstream metadata file into `/opt/shibboleth-idp/metadata/EntraID_Test.xml` (or `EntraID_Prod.xml`)
* Load upstream metadata in `/opt/shibboleth-idp/conf/metadata-providers.xml`
  
  ```
  <MetadataProvider id="UpstreamMetadata"  xsi:type="FilesystemMetadataProvider" metadataFile="/opt/shibboleth-idp/metadata/EntraID_Test.xml"/>
  ```

* Set upstream IdP entityId in `/opt/shibboleth-idp/conf/authn/authn.properties` (to value from the metadata downloaded after registering the applicaiton)
  
  ```
  idp.authn.SAML.proxyEntityID = https://sts.windows.net/abcdefg0-1234-abcd-cdef-123456789abc/
  ```

    > **Note**
    > For this setting to work, your IdP needs to be configured (in `/opt/shibboleth-idp/conf/idp.properties`) to load the `/opt/shibboleth-idp/conf/authn/auth.properies` file - either explictly in `idp.additionalProperties` or via `idp.searchForProperties=true`

* Create incoming attribute mappings: create `/opt/shibboleth-idp/conf/attributes/custom/username.rule` with the following content (assuming the username attribute will be called `username` in the IdP config and `onpremiseusername` in the SAML assertion sent by Entra ID):
  
  ```
  id=username
  transcoder=SAML2StringTranscoder
  saml2.name=onpremiseusername
  saml2.nameFormat=
  saml2.encodeType=False
  ```

    * Similar rules should be added for any other attributes/claims sent by Entra ID
    * The `saml2.nameFormat` value should be set to blank to match how Entra ID encodes attributes (to suppress the default format of `urn:oasis:names:tc:SAML:2.0:attrname-format:uri` not used by Entra ID).

* Configure attribute filter for upstream IdP to accept attributes sent by Entra ID - edit `/opt/shibboleth-idp/conf/attribute-filter.xml` and add (adjusting the Issuer value to match the entityID from your Entra ID metadata and `attributeID` to the name used in IdP config to represent the username sent by Entra ID):
  
  ```
  <AttributeFilterPolicy id="proxy">
      <PolicyRequirementRule xsi:type="Issuer" value="https://sts.windows.net/abcdefg0-1234-abcd-cdef-123456789abc/"/>

      <AttributeRule attributeID="username" permitAny="true" />
  </AttributeFilterPolicy>
  ```

* Configure the IdP to set the principal name (authenticated username) based on the attribute received from Entra ID:
  * Edit `/opt/shibboleth-idp/conf/c14n/subject-c14n.xml` and uncomment the `c14n/attribute` flow 
  * Edit `/opt/shibboleth-idp/conf/c14n/subject-c14n.properties` and set the following:
  
  ```
  # Disable c14n-stage use of attribute-resolver
  idp.c14n.attribute.resolutionCondition = shibboleth.Conditions.FALSE
  # Enable getting attributes from Subject (without attribute resolver)
  idp.c14n.attribute.resolveFromSubject = true
  # Set source attribute ID
  idp.c14n.attribute.attributeSourceIds = onpremisessamaccountname
  ```

  * In case your IdP uses older style of configuration files (from IdP 4.0.x.), do the same also in `/opt/shibboleth-idp/conf/c14n/attribute-sourced-subject-c14n-config.xml` (bean `shibboleth.c14n.attribute.AttributeSourceIds`)

* Make IdP session cookies set `sameSite=none` (to avoid issues where cookies would not be visible when being redirected back from Entra ID).
  In `/opt/shibboleth-idp/conf/idp.properties`, set:
  
  ```
  idp.cookie.sameSiteCondition = shibboleth.Conditions.TRUE
  ```

* Adjust logout pages:
  * Make sure the logout pages have been adjusted as recommended in the [IdP Install Manual Configuring Single Logout section](../identity_providers/installing_a_shibboleth_3_x_idp#configuring-single-logout) (step 2, customise logout page).
  * We also recommend adding a message instructing users to log out of their Entra ID account into the `logout.vm`, `logout-propagate.vm`, and `logout-complete.vm` templates in `/opt/shibboleth-idp/views` (into the body of each page after the main message).  The text can e.g. be:
  
  ```
  <p>
  <br>
  We also recommend logging out of your primary account to prevent
  a future login from reusing the existing primary account session.
  <br>
  <a href="https://login.microsoftonline.com/logout.srf" target="_blank">
  <strong>
  Click here to log out of your primary account.
  </strong>
  </a>
  </p>
  ```

* Switch to SAML authentication flow: in `/opt/shibboleth-idp/conf/idp.properties` (or `/opt/shibboleth-idp/conf/authn/auth.properies`, depending on which IdP version your configuration files come from), change `idp.authn.flows` from `Password` to `SAML`::
  
  ```
  idp.authn.flows = SAML
  ```

* Disable Artifact profile for outgoing messages (such as logout): edit `/opt/shibboleth-idp/conf/idp.properties` and set:
  
  ```
  idp.artifact.enabled = false
  ```

* To prevent the IdP from exposing Entra ID as the `AuthenticatingAuthority` (which could disrupt how identity of users is seen in some services), edit `/opt/shibboleth-idp/conf/relying-party.xml` and in `shibboleth.DefaultRelyingParty` (and all beans under `RelyingPartyOverrides` beans), in the `SAML2.SSO` profile bean, add `p:suppressAuthenticatingAuthority="true"`.  (Note: requires IdP 4.2.0+)

* Now that the IdP authenticates against Entra ID which supports MFA, it is possible to configure the IdP to support [REFEDS MFA](../multi-factor_authentication_with_refeds_mfa_profile).
  The only step required is to configure the IdP to map the Entra ID authentication context value signalling MFA (`http://schemas.microsoft.com/claims/multipleauthn`) to the REFEDS MFA value `https://refeds.org/profile/mfa` accepted by R&E Identity Federations.  Please follow the instructions in our guide for [configuring REFEDS MFA on IdP proxing to Entra ID](../identity_providers/configuring_refeds_mfa_on_shibboleth_idp#refeds-mfa-on-idp-proxying-to-azuread).

