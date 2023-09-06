---
redirect_from: ECP
id: ecp
---
# ECP

The Enhanced Client/Proxy profile is an extension to the SAML 2.0 protocol suite that adds support for non-browser tools and applications to establish a Shibboleth session.

Enabling ECP sessions requires several steps to be completed:

1.  [Enable ECP in the Shibboleth SP configuration](service_providers/installing_shibboleth_sp_on_redhat_based_linux#ecp) on the service to be accessed via ECP
2.  [Register ECP support for this SP in the Federation Registry](federation_management/adding_a_service_provider_to_the_Tuakiri_federation#ecp-support)
3.  [Enable ECP on all IdPs accessing this service](identity_providers/installing_a_shibboleth_3_x_idp#ecp-support)

For testing ECP, there is a bash and python implementation of ECP at [https://wiki.shibboleth.net/confluence/display/SHIB2/Contributions#Contributions-Other%2CRelated%2CContributions](https://wiki.shibboleth.net/confluence/display/SHIB2/Contributions#Contributions-Other%2CRelated%2CContributions)

*   References:
    *   IdP ECP profile configuration: [https://wiki.shibboleth.net/confluence/display/SHIB2/IdPSAML2ECPProfileConfig](https://wiki.shibboleth.net/confluence/display/SHIB2/IdPSAML2ECPProfileConfig)
    *   SAML 2.0-Errata-5 notes on ECP and Metadata: [http://docs.oasis-open.org/security/saml/v2.0/errata05/csprd01/saml-v2.0-errata05-csprd01.html#\_\_RefHeading\_\_8068\_1983180497](http://docs.oasis-open.org/security/saml/v2.0/errata05/csprd01/saml-v2.0-errata05-csprd01.html#__RefHeading__8068_1983180497)
