---
redirect_from: ECP
id: ecp
---
# ECP

The Enhanced Client/Proxy profile is an extension to the SAML 2.0 protocol suite that adds support for non-browser tools and applications to establish a Shibboleth session.

Enabling ECP sessions requires several steps to be completed:

1.  [Enable ECP in the Shibboleth SP configuration](https://reannz.atlassian.net/wiki/spaces/Tuakiri/pages/3815538788/Installing+Shibboleth+SP+on+RedHat+based+Linux#InstallingShibbolethSPonRedHatbasedLinux-ECP) on the service to be accessed via ECP
2.  [Register ECP support for this SP in the Federation Registry](https://reannz.atlassian.net/wiki/spaces/Tuakiri/pages/3815539065/Adding+a+Service+Provider+to+the+Tuakiri+Federation#AddingaServiceProvidertotheTuakiriFederation-ECPsupport)
3.  [Enable ECP on all IdPs accessing this service](https://reannz.atlassian.net/wiki/spaces/Tuakiri/pages/3815538790/Installing+a+Shibboleth+2.x+IdP#InstallingaShibboleth2.xIdP-ECPsupport)

For testing ECP, there is a bash and python implementation of ECP at [https://wiki.shibboleth.net/confluence/display/SHIB2/Contributions#Contributions-Other%2CRelated%2CContributions](https://wiki.shibboleth.net/confluence/display/SHIB2/Contributions#Contributions-Other%2CRelated%2CContributions)

*   References:
    *   IdP ECP profile configuration: [https://wiki.shibboleth.net/confluence/display/SHIB2/IdPSAML2ECPProfileConfig](https://wiki.shibboleth.net/confluence/display/SHIB2/IdPSAML2ECPProfileConfig)
    *   SAML 2.0-Errata-5 notes on ECP and Metadata: [http://docs.oasis-open.org/security/saml/v2.0/errata05/csprd01/saml-v2.0-errata05-csprd01.html#\_\_RefHeading\_\_8068\_1983180497](http://docs.oasis-open.org/security/saml/v2.0/errata05/csprd01/saml-v2.0-errata05-csprd01.html#__RefHeading__8068_1983180497)
