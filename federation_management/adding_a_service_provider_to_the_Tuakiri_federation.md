---
redirect_from: Adding+a+Service+Provider+to+the+Tuakiri+Federation
id: federation_management/adding_a_service_provider_to_the_Tuakiri_federation
---
# Adding a Service Provider to the Tuakiri Federation

# Introduction

Even with Shibboleth SP installed, a Service Provider only becomes useful after registering it into a federation.

The Tuakiri federations to consider are:

*   **[Tuakiri](https://registry.tuakiri.ac.nz/federationregistry)****:** suitable for production systems only
*   **[Tuakiri-TEST](https://registry.test.tuakiri.ac.nz/federationregistry)****:** suitable for test and development systems

This document will cover adding a Service Provider to the Tuakiri or Tuakiri-TEST federation.

# Federation Details

  

|     |     |     |
| --- | --- | --- |
| Federation name | Tuakiri | Tuakiri TEST |
| Metadata name | `**[tuakiri.ac.nz](http://tuakiri.ac.nz)**` | `**[test.tuakiri.ac.nz](http://test.tuakiri.ac.nz)**` |
| Metadata distribution point | [https://directory.tuakiri.ac.nz/metadata/tuakiri-metadata-signed.xml](https://directory.tuakiri.ac.nz/metadata/tuakiri-metadata-signed.xml) | [https://directory.test.tuakiri.ac.nz/metadata/tuakiri-test-metadata-signed.xml](https://directory.test.tuakiri.ac.nz/metadata/tuakiri-test-metadata-signed.xml) |
| Metadata signing certificate | [https://directory.tuakiri.ac.nz/metadata/tuakiri-metadata-cert.pem](https://directory.tuakiri.ac.nz/metadata/tuakiri-metadata-cert.pem) | [https://directory.test.tuakiri.ac.nz/metadata/tuakiri-test-metadata-cert.pem](https://directory.test.tuakiri.ac.nz/metadata/tuakiri-test-metadata-cert.pem) |
| Federation Registry URL | [https://registry.tuakiri.ac.nz/federationregistry/](https://registry.tuakiri.ac.nz/federationregistry/) | [https://registry.test.tuakiri.ac.nz/federationregistry/](https://registry.test.tuakiri.ac.nz/federationregistry/) |
| Discovery Service / WAYF URL | [https://directory.tuakiri.ac.nz/ds/DS](https://directory.tuakiri.ac.nz/ds/DS) | [https://directory.test.tuakiri.ac.nz/ds/DS](https://directory.test.tuakiri.ac.nz/ds/DS) |

# Registration

{% include federation_management/adding_an_sp_to_tuakiri-excerpt %}

# Configuring Shibboleth SP

For full information (including setup instructions for a new install), please see [Installing Shibboleth SP on RedHat based Linux](https://reannz.atlassian.net/wiki/spaces/Tuakiri/pages/3815538788/Installing+Shibboleth+SP+on+RedHat+based+Linux)

The key part of it (relevant for an already setup SP just joining Tuakiri) is:
