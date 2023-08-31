---
redirect_from: Registering+Tuakiri+Hosted+IdP+as+a+Service+with+Google+Apps+or+GSuite
id: tuakiri_hosted_idp/registering_tuakiri_hosted_idp_as_a_service_with_google_apps_or_gsuite
---
# Registering Tuakiri Hosted IdP as a Service with Google Apps or GSuite

The [Tuakiri Hosted IdP](https://reannz.atlassian.net/wiki/spaces/Tuakiri/pages/3815538725/Tuakiri+Hosted+IdP) runs a SAML Identity Provider (IdP) as a SAML proxy, facing Tuakiri as an IdP and facing an upstream IdP as a Service Provider (SP).

A Tuakiri Hosted IdP instance needs to be registered with the upstream IdP as a Service Provider - and also needs the metadata of the upstream IdP.

This page documents the steps required to register a Tuakiri Hosted IdP instance as a Service with Google Apps / GSuite.

These instructions are based on [upstream documentation for registering a Custom SAML application](https://support.google.com/a/answer/6087519) and on experience actually following the instructions - however, the registration process changes over time, so please bear in mind these instructions might become outdated.

# Prerequisites

Before starting the process, you will need:

*   The entityID of the Hosted IdP instance.  This will likely be (with `example.org` replaced by your organisations domain):
    *   for Tuakiri-TEST: `https://idp-test.example.org/idp/shibboleth`
    *   for Tuakiri (Production): `https://idp.example.org/idp/shibboleth`
*   The assertion consumer service (ACS) URL:
    *   for Tuakiri-TEST: `https://hosted-login.test.tuakiri.ac.nz/hosting/example.org/idp/profile/Authn/SAML2/POST/SSO`
    *   for Tuakiri (Production): `https://hosted-login.tuakiri.ac.nz/hosting/example.org/idp/profile/Authn/SAML2/POST/SSO`
*   Super-administrator privileges in your Google Apps / GSuite account.

  

# Registration

GSuite admin console as a number of popular service preconfigured - however, to add the SP side of a Tuakiri Hosted IdP instance, we will be adding a Custom SAML application.

Please repeat this process twice, separately for TEST and PROD registration.

1.  From the Admin console Home page, go to **Apps** and then **Web and mobile apps**.  
      
    
2.  Click **Add App** and then **Add private SAML app** (can be also labelled **Setup my own custom app**) - do not select an application from the list.  
      
    
3.  On the App Details page, enter:
    
    *   Name: <Your organisation> Tuakiri Login
    *   Logo (optionally, may not be shown): upload your organisation's logo as the app icon
    *   Click **Continue**   
          
        
4.  You should be presented with a **Google IdP Information**  screen.  Please download the **IdP metadata** and click **Continue**  (or **Next** )  
      
    
5.  On the **Service Provider Details** screen:
    
    *   Enter the ACS URL and Entity ID as per above
        
    *   Leave **Start URL** blank
    *   Tick the **Signed Response** checkbox
    *   For **NameID**: select  **Basic Information** / **Email** 
    *   For **NameIDFormat**: select emailAddress
    *   Click **Continue**   
          
        
6.  On the Attribute Mapping page, select all available information - this should at the very least include:
    
    |     |     |     |
    | --- | --- | --- |
    | SAML Attribute Name | Category | Source Attribute |
    | email | Basic Information | Primary Email |
    | givenName | Basic Information | First Name |
    | surname | Basic Information | Last Name |
    
    and if desired can also include e.g.:
    
    |     |     |     |
    | --- | --- | --- |
    | SAML Attribute Name | Category | Source Attribute |
    | phoneNumber | Contact Information | Phone Number |
    | address | Contact Information | Address |
    
    When the mapping is complete, click **Finish**.  
      
    
7.  You also need to **Enable** the app for your users.
    *   From the Admin console Home page, go to **Apps** and then **Web and mobile apps** and then select your just registered app.
    *   Click **User access**.
    *   Click **On for everyone** and then click **Save**.

Once the registration is complete, confirm this to [Tuakiri support](mailto:tuakiri@reannz.co.nz) and send through your IdP metadata - alongside with other information required on the [Tuakiri Hosted IdP](https://reannz.atlassian.net/wiki/spaces/Tuakiri/pages/3815538725/Tuakiri+Hosted+IdP) page.
