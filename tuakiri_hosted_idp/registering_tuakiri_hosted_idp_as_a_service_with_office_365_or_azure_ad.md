---
redirect_from: Registering+Tuakiri+Hosted+IdP+as+a+Service+with+Office+365+or+Azure+AD
id: tuakiri_hosted_idp/registering_tuakiri_hosted_idp_as_a_service_with_office_365_or_azure_ad.md
---
# Registering Tuakiri Hosted IdP as a Service with Office 365 or Azure AD

The [Tuakiri Hosted IdP](https://reannz.atlassian.net/wiki/spaces/Tuakiri/pages/3815538725/Tuakiri+Hosted+IdP) runs a SAML Identity Provider (IdP) as a SAML proxy, facing Tuakiri as an IdP and facing an upstream IdP as a Service Provider (SP).

A Tuakiri Hosted IdP instance needs to be registered with the upstream IdP as a Service Provider - and also needs the metadata of the upstream IdP.

This page documents the steps required to register a Tuakiri Hosted IdP instance as a Service with Office 365 / Azure AD.

These instructions are based on [upstream documentation for registering a Custom SAML application](https://docs.microsoft.com/en-gb/azure/active-directory/manage-apps/add-application-portal) and on experience actually following the instructions - however, the registration process changes over time, so please bear in mind these instructions might become outdated.

# Prerequisites

Before starting the process, you will need:

*   The entityID of the Hosted IdP instance.  This will likely be (with `example.org` replaced by your organisations domain):
    *   for Tuakiri-TEST: `https://idp-test.example.org/idp/shibboleth`
    *   for Tuakiri (Production): `https://idp.example.org/idp/shibboleth`
*   The assertion consumer service (ACS) URL:
    *   for Tuakiri-TEST: `https://hosted-login.test.tuakiri.ac.nz/hosting/example.org/idp/profile/Authn/SAML2/POST/SSO`
    *   for Tuakiri (Production): `https://hosted-login.tuakiri.ac.nz/hosting/example.org/idp/profile/Authn/SAML2/POST/SSO`
*   Administrator privileges in your Office 365 / Azure AD account.
    *   This means one of the following roles: Global Administrator, Cloud Application Administrator, Application Administrator

  

# Registration

Office 365 / Azure AD has a number of popular service preconfigured - however, to add the SP side of a Tuakiri Hosted IdP instance, we will be adding a Custom SAML application.

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
5.  Assign Users and Groups (as appropriate)
    *   Ideally, you'd assign a group representing all users to the application (allow access to all users)
    *   Or it might be a group representing just Staff
    *   Or, if Azure AD licensing does not permit use of groups, select individual users
6.  Navigate back to the just-created Application via breadcrumbs at the top
7.  Select Setup Single Sign On, select SAML
8.  On the **Set up Single Sign-on with SAML**  page
    *   Edit **Basic SAML Configuration**: EntityID, Assertion Consumer Service (ACS) as per above
    *   Leave blank Sign on URL, Relay State, Logout URL
    *   User Attributes and Claims: the attributes selected by default are OK for minimum viable set of attributes
        *   Assuming these include givenname, surname, emailaddress, name
        *   If available and desired, include also other attributes that map to [Tuakiri Attributes](https://reannz.atlassian.net/wiki/spaces/Tuakiri/pages/3815538694/Attributes) - such as phoneNumber and address.
    *   Download Federation Metadata XML from SAML Signing Certificate section

Once the registration is complete, confirm this to [Tuakiri support](mailto:tuakiri@reannz.co.nz) and send through your IdP metadata - alongside with other information required on the [Tuakiri Hosted IdP](https://reannz.atlassian.net/wiki/spaces/Tuakiri/pages/3815538725/Tuakiri+Hosted+IdP) page.
