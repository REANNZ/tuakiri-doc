
Go to the respecting Federation Registry URL and:

*   Register an _Organisation_ for your institution (if not already registered)
    *   For Contact Details, do not use a shared mailbox, alias or mailing list when entering an email address because the confirmation email contains a single-use link and may cause some confusion should more than one person attempt to use it.
    *   For _Organization Name_, enter your **DNS domain name**.
    *   For _Organization Display Name_, enter your actual organization name.
*   Wait for the Organisation to be approved
*   Register your IdP under that Organisation
    *   Provide the Contact Details for the IdP admin (again, do not use a shared mailbox).
    *   Select the organisation and provide a name and description for your IdP.
    *   Enter the base URL for your IdP (`**[https://idp.example.org](https://idp.example.org)**`).
    *   Enter the PEM encoded certificate used by your IdP for signing Shibboleth assertions (the default is `$IDP_HOME/credentials/idp.pem`).
    *   Select the attributes the IdP will be able to release to the federation.
    *   Select supported NameID formats. By default, `[urn:oasis:names:tc:SAML:2.0:nameid-format:transient](http://urnoasisnamestcSAML:2.0:nameid-format:transient)` is already selected.
    *   Submit the details and wait for your IdP to be approved.
    *   After having your IdP registration approved, click on the link sent to you to become an Administrator of the IdP's registration.
        
        > **Note**  
        > Confirmation email
        >
        > *   It is important to click on the link in the confirmation email, as this makes the recipient of the email an administrator of the Identity Provider being registered in the Tuakiri Federation Registry.
        >     *   The link in the confirmation email can only be used once.
        >     *   Same applies for the link sent for the Organization registration.
        

