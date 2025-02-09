---
id: tuakiri_hosted_idp/renewing_certificate_for_tuakiri_hosted_idp_in_entra_id
---
# Renewing certificate for Tuakiri Hosted IdP in Entra ID

The [Tuakiri Hosted IdP](../tuakiri_hosted_idp) runs a SAML Identity Provider (IdP) as a SAML proxy, facing Tuakiri as an IdP and facing an upstream IdP as a Service Provider (SP).

The upstream IdP of the Tuakiri Hosted IdP instance may be Microsoft Entra ID.  The certificate included in the metadata produced by Microsoft Entra ID has a fixed expiry date (usually 3 years) and would stop functioning at the expiry time (and the metadata itself would expire as well).

Before the certificate and the metadata expire, it is necessary to replace the certificate, create new metadata, and replace it on the Tuakiri Hosted IdP instance.

This page documents the steps required to replace the certificate and update the metadata for the Entra ID registration of a Tuakiri Hosted IdP instance.

1. Start from [https://portal.azure.com/](https://portal.azure.com/) and navigate to **Enterprise Applications**
  * this can be done by searching for **Enterprise Applications** in the search box on the top of the screen
  * or by selecting **All Services** from the top-left corner menu, and then selecting **Identity**, and then **Enterprise Applications**
2. Search for the application representing Tuakiri Hosted IdP (may also be called `Tuakiri Login`).
  * type `tuakiri` into the search box (`Search by application name`)
  * there should likely be two results (TEST and PROD instance); repeat the subsequent steps for both of them.
3. Navigate to the application
4. Under `Manage` `=>` `Single sign-on` `=>` `SAML Certificates`, select `Edit`, `New Certificate`, and `Save`.
  * this adds the new certificate into the metadata, so far as _Inactive_.
  * close the `SAML Signing Certificate` pop up (by clicking the `X` in the top-right corner)
5. From the `SAML Certificates` panel, `Download` the `Federation Metadata XML`.
   * send the downloaded metadata file to [tuakiri@reannz.co.nz](mailto:tuakiri@reannz.co.nz) and wait for confirmation the metadata has been updated on the Tuakiri Hosted IdP service.
   * this step will make sure Tuakiri Hosted IdP is ready to accept either the old or the new certificate.

**Only after** receiving confirmation the metadata has been updated on Tuakiri Hosted IdP, proceed with the following steps (again, repeating them for both TEST and PROD instances of your application).
6. Navigate to your application again, following the same steps as above.
7. Under `Manage` `=>` `Single sign-on` `=>` `SAML Certificates`, again select `Edit`
8. In the list of certificates, identify the new one (so far marked _Inactive_) and from its context menu (`...` at the end of the line representing the certificate), select `Make certificate active`).  The previously Active certificate will now become _Inactive_.
  * The `Expiration Date` should also help tell the the old and new certificates apart.
9. Delete the old (now _Inactive_) certificate (select  `Delete Certificate` from its `...` context menu).
  * Again close the `SAML Signing Certificates` popup.
10. `Download` the `Federation Metadata XML` once more and send to [tuakiri@reannz.co.nz](mailto:tuakiri@reannz.co.nz).
   * this step will make sure Tuakiri Hosted IdP now accepts only the new certificate.

