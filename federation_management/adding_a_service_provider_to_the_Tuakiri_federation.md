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

{% include federation_management/tuakiri_metadata-excerpt.md %}

# Registration

{% include federation_management/adding_an_sp_to_tuakiri-excerpt.md %}

# Configuring Shibboleth SP

For full information (including setup instructions for a new install), please see [Installing Shibboleth SP on RedHat based Linux](https://reannz.atlassian.net/wiki/spaces/Tuakiri/pages/3815538788/Installing+Shibboleth+SP+on+RedHat+based+Linux)

The key part of it (relevant for an already setup SP just joining Tuakiri) is:

{% include service_providers/shibsp-excerpt-conf-metadata.md %}
