
As of version 2.6.0, the Federation Registry automatically registers the ECP endpoint on new registrations, so no explicit action should be required.  To add an ECP endpoint to an existing IdP registration, perform the following:

In the Federation Registry registration for your IdP:

*   Add a new "Single Sin On Service" Endpoint
*   Select Binding: `[urn:oasis:names:tc:SAML:2.0:bindings:SOAP](http://urnoasisnamestcSAML:2.0:bindings:SOAP)`
*   Enter Location: `[https://idp.example.org/idp/profile/SAML2/SOAP/ECP](https://idp.example.org/idp/profile/SAML2/SOAP/ECP)` (substituting your IdP hostname)

