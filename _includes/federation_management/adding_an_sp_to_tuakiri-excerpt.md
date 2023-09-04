
> **Note**  
In order to register a Service Provider with the Federation Registry, it is **highly recommended** that you are able to log in with a user account authorised by an IdP or Virtual Home already registered with the federation.
>
It is possible to add an SP to the federation without an account but to become the administrator of that SP or later review the SP registration entry or to make any changes (in the textual description or the technical details - endpoint URLs, certificates, attributes required, etc), you **will** need an account.

Navigate to the Tuakiri federation management site [https://registry.tuakiri.ac.nz/federationregistry](https://registry.tuakiri.ac.nz/federationregistry) (or, for Tuakiri-TEST federation, [https://registry.test.tuakiri.ac.nz/federationregistry](https://registry.test.tuakiri.ac.nz/federationregistry)).

If you **do not** have an account with an IdP registered in the federation and you **do not** have an account with the Tuakiri VHO, start the SP registration (without logging in), by clicking the **Create Service Provider** link in the blue menu bar.

Otherwise, click **Login** and login using your IdP. Start the registration by clicking **Subscribers** > **Service Providers** > **Create**.

The registration form first displays a check-list of required information.  Please check that you have all the information the check-list asks for readily available, otherwise the registration form may time out while you gather missing information.

Please note that on the registration form, you'll be asked to select the organization you are registering the SP under.  If you have not registered your organization into Tuakiri (or Tuakiri-TEST) yet, please complete that process first, following our instructions for [Creating an Organization in the Tuakiri Federation](https://reannz.atlassian.net/wiki/spaces/Tuakiri/pages/3815539860/Creating+an+Organization+in+the+Tuakiri+Federation).

1.  If you are filling this form without having logged in, you'll have to enter your details: **Given Name**, **Surname** and **Email** (otherwise, these are prefilled). (Do not use a shared mailbox, alias or mailing list when entering an email address because the confirmation email contains a single-use link and may cause some confusion should more than one person attempt to use it.)  
      
    
2.  Enter the information that describes the Service Provider being registered:
    *   Select your **Organization**; if the organization you wish to host the Service Provider under does not exist, follow this [procedure to create one](https://reannz.atlassian.net/wiki/spaces/Tuakiri/pages/3815539860/Creating+an+Organization+in+the+Tuakiri+Federation). (_Required_)
    *   Enter a **Display Name** and **Description** for your Service Provider. (_Required_)
    *   Enter a **Service URL** for accessing your Service Provider in the form `[http://sp.example.org](http://sp.example.org)`. The service URL is typically the base URL for accessing the Service Provider. (_Required_)
    *   Optionally enter a URL in the **Service Logo URL** to a image or logo representing the service the Service Provider is authenticating. (_Optional_)
3.  Enter the basic SAML configuration:
    *   Choose the Shibboleth SP version that is installed on the Service Provider.
    *   Enter the Service Provider's base URL for the **Host**, without a trailing slash, just as `**[https://sp.example.org](https://sp.example.org)**`. The Federation Registry will automatically create all of the SAML2 endpoints from this base URL.  You can skip over the **Advanced SAML2 Registration** section.
    *   Alternatively, if your deployment does not fit the basic SAML configurator (either your implementation is not among the pre-configured ones, or you use custom endpoint URLs),  please fill in the entityID and the endpoint URLs in the **Advanced SAML2 Registration** section.
4.  Copy and paste in the back-channel certificate(s) generated when installing the Shibboleth SP software.
    
    For Shibboleth SP 2.x, the certificate is usually located in `/etc/shibboleth/sp-cert.pem`.
    
    > **Note**  
    Starting with Shibboleth 3.0, the two separate certificates are generated for encryption and signing.  These certificates are stored as `/etc/shibboleth/sp-signing-cert.pem` and `/etc/shibboleth/sp-encrypt-cert.pem`
    >
    As of October 2018, Federation Registry supports including separate signing and encryption certificates on the registration form.  If you are registering an SP that has a single certificate used for both signing and encryption, copy the same certificate into both fields.  If your SP does not support encryption, leave the encryption certificate field blank (but a signing certificate is required).
    
    When pasting the certificate into the form, please take care that no line breaks, spaces or other characters are introduced during the cut-paste process.
    
    *   Please note that it is highly recommended that the `CN` in the certificate matches the hostname the service provider is being registered under. If this is an alias and your system thinks of itself with a different hostname, we recommend you instead generate a new certificate with the correct hostname: run the following, substituting the externally visible hostname for `sp.example.org`:
        
        ```
        cd /etc/shibboleth
        # for a Shibboleth 2.x system:
        ./keygen.sh -f -u shibd -g shibd -y 20 -h sp.example.org -e https://sp.example.org/shibboleth
        # or for a Shibboleth 3.x system:
        ./keygen.sh -f -n sp-signing -u shibd -g shibd -y 20 -h sp.example.org -e https://sp.example.org/shibboleth
        ./keygen.sh -f -n sp-encrypt -u shibd -g shibd -y 20 -h sp.example.org -e https://sp.example.org/shibboleth 
        ```
        
5.  Select the attributes **Requested** and mark which are **Required**. For each attribute requested give a good explanation for why the attribute is requested. This information will later be displayed to users as justification for why the information is being released.
    
    > **Note**  
    Persistent NameID
    >
    Please note that with the [IdPv3 upgrade](https://reannz.atlassian.net/wiki/spaces/Tuakiri/pages/3815539009/Upgrading+a+2.x+IdP+to+3.x), Tuakiri is moving from passing Persistent NameIDs in the eduPersonTargetedID attribute to passing them as a Persistent SAML2 NameID.  When registering a new SP requesting a persistent NameID, please request both the eduPersonTargetedID attribute (for interoperability with existing V2 IdPs), as well as NameID of Persistent format.  You will be able to add the SAML 2.0 Persistent NameIDFormat after your SP registration is approved - or please get in touch with the [Tuakiri Support](mailto:reannz@tuakiri.ac.nz).
    
    > **Note**  
    schac attributes
    >
    Please note that as of 2.6.0, Shibboleth SP includes attributes from the schac schema in the default configuration.  The names used for the attributes there are slightly different from what has been used in the attribute-map.xml file provided by Tuakiri for use with earlier versions of Shibboleth SP.  For compatibility with 2.6.0, we have adjusted the names in [attribute-map.xml](https://github.com/REANNZ/Tuakiri-public/raw/master/shibboleth-sp/attribute-map.xml) to match the names used by the 2.6.0 default configuration.
    >
    **homeOrganization** is becoming **schacHomeOrganization****  
    homeOrganizationType** is becoming **schacHomeOrganizationType**
    
    > **Note**  
    eduPersonEntitlement attribute
    >
    Please note: if intending to request the eduPersonEntitlement attribute, you cannot add the attribute to the list of requested attributes on the registration page; you'll have to add it separately later.
    >
    Also, because of the nature of this attribute, you also have to include a specific requested value (or a regular expression matching a set of values), but an eduPersonEntitlement attribute request without specific values is not considered complete and will be ignored by the Federation Registry.
    
6.  Click **Submit** and wait for a confirmation email.

> **Note**  
**Please note:** Once the registration is approved, the Federation Registry will send an email with an invite code to claim administrative rights over the SP being registered.
>
It is important to follow the instructions in the email to get the administrative privileges over the SP.  These privileges are required for making any subsequent changes to the SP registration.
>
Note that the invite code can only be used once - but once the original recipient has administrative privileges, these can be used to grant the same administrative privileges to additional users as required.

## ECP support

If your SP should support [ECP](https://reannz.atlassian.net/wiki/spaces/Tuakiri/pages/3815538794/ECP) (access via non-browser clients), then also register support for ECP:

*   After your SP registration is complete, log into the Federation Registry again (in the same way as above)
*   Open the entry for your SP (under Subscribers -> Service Providers or directly from the Dashboard)
*   Under EndPoints -> Assertion Consumer Service, add a new Endpoint:
    *   Select Binding: `urn:oasis:names:tc:SAML:2.0:bindings:PAOS`
    *   Enter Location: `[https://sp.example.org/Shibboleth.sso/SAML2/ECP](https://sp.example.org/Shibboleth.sso/SAML2/ECP)` (substituting `sp.example.org` with your SP hostname)
    *   Enter Index: `4` (value `4` matches the value in the Shibboleth SP internal metadata in the default configuration)
*   Remember to also [configure support for ECP](https://reannz.atlassian.net/wiki/spaces/Tuakiri/pages/3815538788/Installing+Shibboleth+SP+on+RedHat+based+Linux#InstallingShibbolethSPonRedHatbasedLinux-ECP) in your `/etc/shibboleth/shibboleth2.xml` file.

