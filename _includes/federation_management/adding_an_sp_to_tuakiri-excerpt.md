
> **Note**  
> In order to be able to submit a registration request for a Service Provider through the Metadata Tool, it is **required** that you are able to log in with a Tuakiri login, either with a user account at a Tuakiri member organisation or at the Tuakiri Virtual Home.
>
> The Tuakiri Service Desk can either provision you with an account on the Virtual Home, or perform the registration on your behalf.
>
> Please contact [Tuakiri Service Desk](mailto:tuakiri@reannz.co.nz) if you do not have a valid account with an IdP registered in the federation and/or the Tuakiri Virtual Home.

Start the registration by navigating to the Tuakiri federation management site [https://registry.tuakiri.ac.nz/](https://registry.tuakiri.ac.nz/) (or, for Tuakiri-TEST federation, [https://registry.test.tuakiri.ac.nz/](https://registry.test.tuakiri.ac.nz/)).

Follow the instructions for [Using the Metadata Tool]({{ 'federation_management/using_metadata_tool' | relative_url}}).

A few special points to consider for an SP:

1. The Metadata Tool also collects information about registered services that is used to produce the
[Tuakiri Service Catalogue](https://www.reannz.co.nz/products-and-services/federated-identity-and-access-management/tuakiri/tuakiri-service-catalogue).
Part of this information is a Service URL - URL that users can use to access your service.
Please record this URL in the Metadata Tool (in the ServiceInfo section) to allow your service to be included in the Service Catalogue.

2. Requested Attributes: in the Metadata Tool, Requested Attributes are added in the AttributeConsumingService section.  Please copy pre-defined attributes from the provided list.
    
    > **Note**  
    > Persistent NameID
    >
    > Please note that with the [IdPv3 upgrade]({{ 'identity_providers/installing_a_shibboleth_3_x_idp' | relative_url}}), Tuakiri is moving from passing Persistent NameIDs in the eduPersonTargetedID attribute to passing them as a Persistent SAML2 NameID.  When registering a new SP requesting a persistent NameID, please request both the eduPersonTargetedID attribute (for interoperability with IdPs that have not migrated to SAML2 NameID), as well as NameID of Persistent format.  This can be done by including the SAML 2.0 Persistent NameIDFormat (`urn:oasis:names:tc:SAML:2.0:nameid-format:persistent`) in your SP metadata.  If not sure, please get in touch with the [Tuakiri Service Desk](mailto:reannz@tuakiri.ac.nz).
    
    > **Note**  
    > eduPersonEntitlement attribute
    >
    > Please note: if intending to request the eduPersonEntitlement attribute, the attribute request will have to be augmented with the specific values requested.  Please get in touch with the [Tuakiri Service Desk](mailto:reannz@tuakiri.ac.nz) if requesting `eduPersonEntitlement`.
