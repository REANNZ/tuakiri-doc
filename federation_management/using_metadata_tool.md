---
id: federation_management/using_metadata_tool
---
# Using the Metadata Tool to manage Tuakiri metadata
{:.no_toc}

Tuakiri has migrated from legacy tooling (Federation Registry run in combination with the SAML Service) to a new system, centered around the Metadata Tool, developed by SUNET (Swedish NREN) to operate SWAMID (Swedish R&E Identity Federation).

The Metadata Tool also provides a web interface to submitting new SAML metadata for an IdP or SP, as well as for requesting changes to already published metadata.  And authentication to the Metadata Tool is also via Tuakiri.
However, the way the metadata is submitted, as well as supporting processes, are different.

1. TOC
{:toc}

# Accessing the Metadata Tool

The Metadata Tool reuses the public hostnames originally used by the Federation Registry (as it still acts as the registry of entities registered in Tuakiri).  The hostnames are:

* **Tuakiri** (Production) : [https://registry.tuakiri.ac.nz/](https://registry.tuakiri.ac.nz/)
* **Tuakiri-TEST** : [https://registry.test.tuakiri.ac.nz/](https://registry.test.tuakiri.ac.nz/)

The Metadata Tool first presents a landing page that does not require authentication and presents a read-only view of entities registered in Tuakiri (and also of entities accepted from eduGAIN).

To request a new registration or changes to an existing registration, follow the `Access through your institution` link and authenticate with your institutional login through Tuakiri.  If your institution is not listed or you are not able to authenticate, please contact Tuakiri Support at [tuakiri@reannz.co.nz](mailto:tuakiri@reannz.co.nz)

# Requesting a New Registration

A new registration is started by uploading an XML file with the entity metadata.  This file does not have to cover all details required by Tuakiri, but it is important it has all the required technical components (certificates and SAML endpoint URLs).  Most SAML implementations produce such metadata, typically at a well-known URL.

For common implementations, the URLs take the following form (available at the specific hostname, using `*.example.org` as a placeholder here):
* **Shibboleth IdP**: `https://idp.example.org/idp/shibboleth`
* **Shibboleth SP**: `https://sp.example.org/Shibboleth.sso/Metadata`
* **Simple SAML php**: `https://sp.example.org/simplesaml/module.php/saml/sp/metadata.php/default-sp`

1. Get the XML metadata for your IdP or SP as a local file.
2. Use the Upload new XML menu item to upload the file to the Metadata Tool.  This creates a new Draft.
3. Edit the Draft within the Metadata Tool to add any missing information, specifically:
    * MDUI (Metadata User Interface extensions):
        * DisplayName (required): For an SP, name of the service.  For an IdP, name of the organisation.
        * Description (optional): For an SP, a description of the service.
        * InformationURL (optional): For an SP, a URL to a page providing further information about the service.
        * PrivacyStatementURL (optional): For an SP, a URL to a page with the service's Privacy Statement or Policy.
        * Logo (optional): either an `https` or `data:` URL, plus the logo's height and width.
    * Organization: give basic information about the organisation registering the IdP or SP:
        * OrganizationName: the **domain** name of the organisation - such as `example.org`
        * OrganizationDisplayName: the human readable name of the organisation - such as `Example University`
        * OrganizationURL: URL to the organisation's website.
    * ContactPersons: list points of contact (email addresses, ideally role-based).
        * Multiple contact entries can be given for different contact types (technical/support/security/administrative).
        * A technical contact is required.
        * A security contact is strongly recommended (required for entities connected to eduGAIN)
        * For each Contact entry, provide `GivenName`, `Surname` and `EmailAddress`.
4. `Validate` the draft metadata.
5. After all requirements are met, `Request publication` (either into Tuakiri only or Tuakiri and eduGAIN).

After submitting the publication request, the Metadata Tool will send you an email confirming the request.

Forward this email to [tuakiri@reannz.co.nz](mailto:tuakiri@reannz.co.nz) to trigger the next step.

A Tuakiri Service Desk team member will process the request and publish the metadata.

# Updating an Existing Registration

An existing entity registration can be updated by creating a Draft from the existing Published entity, making changes in the Draft copy, and submitting the draft for publication.

1. Locate the Published entity in the Metadata Tool.
2. Establish admin access to the entity.
    * If you originally registered the entity, you will already have administrative access.
    * Alternatively, you can `Request admin access`.  A request will be sent to technical and administrative contacts of the entity who will be asked to confirm your request.
3. Create a Draft copy of the metadata.
4. Make desired changes.
5. `Validate` the draft metadata.
6. `Request Publication` of the updated metadata.

Same as when submitting a new registration,  the Metadata Tool will send you an email confirming the request.

Forward this email to [tuakiri@reannz.co.nz](mailto:tuakiri@reannz.co.nz) to trigger the next step.

A Tuakiri Service Desk team member will process the request and publish the updated metadata.


# Removing existing metadata

It is also possible to request removal of published metadata.


1. Locate the Published entity in the Metadata Tool.
2. Establish admin access to the entity (same as above when updating metadata)
3. `Request removal` of the metadata.

Same as in the other scenarios above,  the Metadata Tool will send you an email confirming the request.

Forward this email to [tuakiri@reannz.co.nz](mailto:tuakiri@reannz.co.nz) to trigger the next step.

A Tuakiri Service Desk team member will process the request and remove the entity  metadata.

