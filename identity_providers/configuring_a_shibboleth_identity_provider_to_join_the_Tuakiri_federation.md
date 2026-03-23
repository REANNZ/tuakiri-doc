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

# Configuring your IdP to load the federation metadata

The code snippets in this section have values for Tuakiri Production federation. Please update them accordingly as per the table above if configuring your IdP to join the Tuakiri TEST/DEV federation. (The key code snippets are for convenience given in the "Tuakiri-TEST specific" box below).

**NOTE:** Check what your _IdP home directory_ is: the directory is typically called `shibboleth-idp` - and on Debian and Ubuntu systems, it's commonly `/usr/local/shibboleth-idp`, while on RedHat and CentOS it's `/opt/shibboleth-idp`. The snippets below are referring to the IdP home directory as `$IDP_HOME`

To configure a 3.x IdP to Load the Tuakiri metadata:

{% include identity_providers/idp_excerpt_idp3-load-metadata.md %}

# Configure attribute release/filtering through the federation

To configure a 3.x IdP to Load the Tuakiri-provided attribute filter:

{% include identity_providers/idp_excerpt_attribute-release-edugain.md %}

Now your IdP should be able to access service providers within the Tuakiri federation.

# Appendix A - Alternative implementation

Loading the metadata and the attribute filter files from a remote URL makes the IdP depend on the accessibility of the remote URL. While for metadata itself, the IdP software should be sufficiently resilient, for attribute filter configuration, this is not the case. Tuakiri will be running their servers serving these XML files according to best practices. However, some sites may prefer not to take on the risk and put the XML file loading outside of the IdP, into a separate process. This section describes this alternative implementation. This implementation first downloads the XML file into a temporary file on the local machine. Once this is completed it then replaces the original configuration file with the new one, and this will be detected by the IdP and will cause a reload of this file.

{% include infra_tasks/fetching-metadata-excerpt.md %}
