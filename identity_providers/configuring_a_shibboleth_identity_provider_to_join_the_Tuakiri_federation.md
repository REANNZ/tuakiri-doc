---
redirect_from: Configuring+a+Shibboleth+Identity+Provider+to+join+the+Tuakiri+Federation
id: identity_providers/configuring_a_shibboleth_identity_provider_to_join_the_Tuakiri_federation
---
# Configuring a Shibboleth Identity Provider to join the Tuakiri Federation
{:.no_toc}

For a Shibboleth Identity Provider to join one of the Tuakiri Federations (Test/Dev or Production), the following steps have to be done:

*   Registering the IdP in the Federation Registry
*   Configuring the IdP to load the federation metadata
*   Configuring the IdP to release the attributes required by the federation.

There are two federations available, both fully operational:

*   **Tuakiri Test/Development Environment** (**Tuakiri-TEST**) for testing deployments and developing new features
*   **Tuakiri Production Federation Service** (**Tuakiri**) for ready-for-production Identity Providers and services.

We recommend first registering a Test system into Tuakiri-TEST and after successful testing, register a production-ready system into Tuakiri Production.

1. TOC
{:toc}

# Federation Details

{% include federation_management/tuakiri_metadata-excerpt.md %}

# Registering an IdP into the Federation Registry

{% include identity_providers/idp_excerpt_register-idp-into-FR.md %}

## ECP support

If supporting ECP, advertise also your ECP SSO EndPoint:

{% include identity_providers/idp_excerpt_idp-register-ecp.md indent="" %}

The IdP also needs to be [configured to support ECP](https://reannz.atlassian.net/wiki/spaces/Tuakiri/pages/3815538790/Installing+a+Shibboleth+2.x+IdP#InstallingaShibboleth2.xIdP-ECPsupport).

# Configuring your IdP to load the federation metadata

The code snippets in this section have values for Tuakiri Production federation. Please update them accordingly as per the table above if configuring your IdP to join the Tuakiri TEST/DEV federation. (The key code snippets are for convenience given in the "Tuakiri-TEST specific" box below).

**NOTE:** Check what your _IdP home directory_ is: the directory is typically called `shibboleth-idp` - and on Debian and Ubuntu systems, it's commonly `/usr/local/shibboleth-idp`, while on RedHat and CentOS it's `/opt/shibboleth-idp`. The snippets below are referring to the IdP home directory as `$IDP_HOME`

To configure a 3.x IdP to Load the Tuakiri metadata:

{% include identity_providers/idp_excerpt_idp3-load-metadata.md %}

For archival purposes, we also keep the original instructions for configuring the Tuakiri metadata into a 2.x IdP - unfold the box below to see the IdP 2.x compatible syntax:

<details markdown="1">
<summary>Legacy IdP 2.x configuration to load Tuakiri metadata</summary>

*   Download the metadata signing certificate into `$IDP_HOME/credentials`:
    
    ```
    wget https://directory.tuakiri.ac.nz/metadata/tuakiri-metadata-cert.pem -O $IDP_HOME/credentials/tuakiri-metadata-cert.pem
    ```
    
*   In `$IDP_HOME/conf/relying-party.xml`
    *   Add the following snippet into the `ChainingMetadataProvider`:
        
        ```
                <!-- Tuakiri -->
                <metadata:MetadataProvider id="Tuakiri" xsi:type="metadata:ResourceBackedMetadataProvider">
                  <metadata:MetadataFilter xsi:type="metadata:ChainingFilter" xmlns="urn:mace:shibboleth:2.0:metadata">
                    <metadata:MetadataFilter xsi:type="metadata:SignatureValidation" xmlns="urn:mace:shibboleth:2.0:metadata"
                                    trustEngineRef="shibboleth.MetadataTrustEngine"
                                    requireSignedMetadata="true" />
                  </metadata:MetadataFilter>
                  <metadata:MetadataResource xsi:type="resource:FileBackedHttpResource"
                                  url="https://directory.tuakiri.ac.nz/metadata/tuakiri-metadata-signed.xml"
                                  file="/opt/shibboleth-idp/metadata/tuakiri-metadata.xml" />
                </metadata:MetadataProvider>
        ```
        
    *   And add the following snippet into the `<security:TrustEngine id="shibboleth.MetadataTrustEngine" xsi:type="security:StaticExplicitKeySignature">` element:
        
        ```
                <security:Credential id="Tuakiri-FederationCredentials" xsi:type="security:X509Filesystem">
                    <security:Certificate>/opt/shibboleth-idp/credentials/tuakiri-metadata-cert.pem</security:Certificate>
                </security:Credential>
        ```
        
        > **Note**  
        > Remember to uncomment the `<security:TrustEngine id="shibboleth.MetadataTrustEngine" xsi:type="security:StaticExplicitKeySignature">` element if it is still commented out (it is commented out in the default configuration).
        
</details>

  

# Configure attribute release/filtering through the federation

To configure a 3.x IdP to Load the Tuakiri-managed attribute filter:

{% include identity_providers/idp_excerpt_idp3-load-attribute-filter.md %}

For archival purposes, we also keep the original instructions for configuring the Tuakiri-managed attribute filter into a 2.x IdP - unfold the box below to see the IdP 2.x compatible syntax:

<details markdown="1">
<summary>Legacy IdP 2.x syntax to load an attribute filter</summary>

After requesting the attribute filter:

  

*   Add the following entry into `<srv:Service id="shibboleth.AttributeFilterEngine"` in `$IDP_HOME/conf/service.xml`(note that the URL varies for each IdP and has to be obtained from the federation administrators):
    
    ```
            <srv:ConfigurationResource xsi:type="resource:FileBackedHttpResource"
                                  url="https://directory.tuakiri.ac.nz/attribute-filter/<institution-domain>.xml"
                                  file="/opt/shibboleth-idp/conf/tuakiri-attribute-filter.xml" />
    ```
    
    > **Note**  
    > Note: if your `$IDP_HOME` is different from `/opt/shibboleth-idp`, change the file path in the above snippet accordingly.
    
    > **Note**  
    > If configuring this in Shibboleth IdP 2.1.x, do not use the srv: namespace prefix - i.e., use just:
    >
    > ```
    >         <ConfigurationResource xsi:type="resource:FileBackedHttpResource"
    >                       url="https://directory.tuakiri.ac.nz/attribute-filter/<institution-domain>.xml"
    >                       file="/opt/shibboleth-idp/conf/tuakiri-attribute-filter.xml" />
    >
    > ```
    
*   We also strongly recommend you configure your IdP to periodically reload this file - we recommend at 2 hour intervals. This is documented in detail in the [IdP Install Manual: Reloading configuration section](https://reannz.atlassian.net/wiki/spaces/Tuakiri/pages/3815538790/Installing+a+Shibboleth+2.x+IdP#InstallingaShibboleth2.xIdP-Enablingautomaticreload) and [Load Attribute Filter](https://reannz.atlassian.net/wiki/spaces/Tuakiri/pages/3815538790/Installing+a+Shibboleth+2.x+IdP#InstallingaShibboleth2.xIdP-LoadAttributeFilter) sections. The simple step is to add the `configurationResourcePollingFrequency="PT2H0M0.000S"` and `configurationResourcePollingRetryAttempts="10"` attributes to the `<srv:Service id="shibboleth.AttributeFilterEngine"`element. If you already have these attributes set for reloading the local configuration file - with a shorter interval, please adjust them accordingly to 2 hours for the remotely loaded attribute filter:
    
    ```
        <srv:Service id="shibboleth.AttributeFilterEngine"
    +             configurationResourcePollingFrequency="PT2H0M0.000S" configurationResourcePollingRetryAttempts="10"
                 xsi:type="attribute-afp:ShibbolethAttributeFilteringEngine">
    ```

</details>
  

Now your IdP should be able to access service providers within the Tuakiri federation.

# Appendix A - Alternative implementation

Loading the metadata and the attribute filter files from a remote URL makes the IdP depend on the accessibility of the remote URL. While for metadata itself, the IdP software should be sufficiently resilient, for attribute filter configuration, this is not the case. Tuakiri will be running their servers serving these XML files according to best practices. However, some sites may prefer not to take on the risk and put the XML file loading outside of the IdP, into a separate process. This section describes this alternative implementation. This implementation first downloads the XML file into a temporary file on the local machine. Once this is completed it then replaces the original configuration file with the new one, and this will be detected by the IdP and will cause a reload of this file.

{% include infra_tasks/fetching-metadata-excerpt.md %}
