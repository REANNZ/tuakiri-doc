# Creating an Organization in the Tuakiri Federation

# Introduction

In order to register Service Providers(SP), Identity Providers(IdP), and other entities in a Federation they should be affiliated with an Organization.

The federations to consider are:

*   **Tuakiri:** suitable for production (and user acceptance testing) systems for use by New Zealand (and Australian) academic community ([https://registry.tuakiri.ac.nz/federationregistry/](https://registry.tuakiri.ac.nz/federationregistry/))
*   **Tuakiri-TEST:** suitable for test and development systems for future use by New Zealand (and Australian) academic community ([https://registry.test.tuakiri.ac.nz/federationregistry/](https://registry.test.tuakiri.ac.nz/federationregistry/))

Users log into the Federation Registry to administer their Organization, IdP and SP by authenticating with their normal IdP. The login is not necessary to register an Organization, IdP, or SP as the registration can be sent in anonymous mode. An IdP authenticated login is required to modify the registration, and to accept the invitation to become an administrator that is automatically sent to the email address provided in the initial registration.

This document is focused on adding a production Organization to the Tuakiri Federation.  To adding a development or test Organization to the Tuakiri-TEST Federation, please log into the [Tuakiri-TEST Federation Registry](https://registry.test.tuakiri.ac.nz/federationregistry/) instead.

## Organization Creation

The Organization creation process is straightforward, but **can take up to two business days**to get a new Organization approved, be sure to plan for this delay.

In order to register an Organization in the Federation Registry, it is **highly recommended** that you are able to log in with a user account on an IdP or Virtual Home already registered with the federation.

It is possible to add an Organization into the federation without an account but to become the administrator of that Organization or later review the Organization entry or to make any changes, or to approve IdPs and SPs registered under the Organization, you **will** need an account.

Go to the [Tuakiri Federation Registry](https://registry.tuakiri.ac.nz/federationregistry) (or, if registering a development or test Organization to the Tuakiri-TEST Federation, please go to the [Tuakiri-TEST Federation Registry](https://registry.test.tuakiri.ac.nz/federationregistry/) instead).

If you are able to log in, click **Login** and login using your IdP. Start the registration by clicking **Subscribers** > **Organizations** > **Create**.

Otherwise, click on **Create New Organisation** in the blue menu bar.

The registration form first displays a check-list of required information.  Please check that you have all the information the check-list asks for readily available, otherwise the registration form may time out while you gather missing information.

1.  Enter the contact details of the new organization's administrator (these will be prefilled if you have logged into the Federation Registry first).
2.  Enter the new Organization's details
    *   **DNS Name:** the domain name of the organization eg. `example.org.nz`
    *   **Display Name:** the name of the organization e.g. The Example Organization of New Zealand
    *   **Website:** a URL to a website for the organization e.g. `http://www.example.org.nz/`
    *   **Organisation type:** Choose the organization type that best suites the organization.
3.  Click on submit and wait for the confirmation email
4.  Once the organization is approved, follow the instructions in the confirmation email to become an administrator of the organization (requires being able to log into the Federation Registry via Tuakiri / Tuakiri-TEST)