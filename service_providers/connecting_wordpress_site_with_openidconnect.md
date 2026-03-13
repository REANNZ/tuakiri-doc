---
id: service_providers/connecting_wordpress_site_with_openidconnect.md
---
# Connecting a WordPress site with Tuakiri OpenID Connect Bridge
{:.no_toc}

Tuakiri operates the [Tuakiri OpenID Connect Bridge](../tuakiri_openid_connect_bridge) to support connections from services that cannot do full SAML integration.  Historically, Tuakiri also offered the RapidConnect service, but RapidConnect has now been deprecated and services using it need to migrate to OpenID Connect.

For WordPress sites, Tuakiri had also maintained the WordPress RapidConnect plugin (initially developed by the AAF).  For connecting a WordPress site to Tuakiri with OpenID Connect, the [OpenID Connect Generic Client](https://wordpress.com/plugins/daggerhart-openid-connect-generic) WordPress plugin provides all the functionality required to connect to the Tuakiri OpenID Connect Bridge.

This page documents the steps required to connect a WordPress site to Tuakiri with OpenID Connect.

1. TOC
{:toc}

# Registration with Tuakiri

Follow the [registration instructions](../tuakiri_openid_connect_bridge#initial-registration)
in the Tuakiri OpenID Connect documentation, with the specific parameters as shown below.

Email [tuakiri@reannz.co.nz](mailto:tuakiri@reannz.co.nz), requesting to register your WordPress site, and include the following information:
* The site's URL.
* Name of the organisation operating the site.
* URL to the organisation's website (if different).
* Contact email addresses (`support` and `technical`) - should be generic email addresses not linked to a single person
* Technical information specific to the WorPress OpenID Connect Generic Client
    * the redirect URI will be of the form `https://site.example.org/wp-admin/admin-ajax.php?action=openid-connect-authorize`
    * attributes needed wil be `mail` (required), `givenname`, `surname` (optional)

Tuakiri Service Desk will create your registration entry and send you back parameters you will need to configure the plugin to connect to Tuakiri (primarily clientID and secret).

# Configuring OpenID Client in WordPress

In your WordPress admin console, install the OpenID Connect Generic Client plugin (`daggerhart-openid-connect-generic`).

Afterwards, navigate to `Settings` => `OpenID Connect Client` and change the following settings (where not specified here, leave settings at their default value):
* Start with `Quick Setup: Import from Discovery Document`, enter URL `https://openidconnect.tuakiri.ac.nz/.well-known/openid-configuration` and click `Load Configuration`.  This will populate a number of fields on the configuration page with the right values.
* `Login Button Text`: e.g. `University of Example Login`
* `Client ID`, `Client Secret Key`: values as provied by Tuakiri Support.
* `OpenID Scope`: `openid email profile`
* Login, Userinfo, Token Validation, JWKS URLs: auto-populated in the first step above.
* `Nickname Key`: `sub` (a user identifier the plugin can retrieve from the login)
* `State time limit`: `3600` (how long the login link will be valid for, default 3 minutes  are too short)
* `Link Existing Users`: check (to make WordPress map logins via the new plugin with the same email address to existing accounts)

Click `Save Changes`.

On your login page, add a login button with Shortcode `[openid_connect_generic_login_button]`.

Please test with both an  existing account and a new account.

# Removing RapidConnect Plugin

Once confirmed to work:
* Remove the RapidConnect login button from the login page.
* Deactivate the `Tuakiri Rapid Connect` plugin.
* After a while, Delete the `Tuakiri Rapid Connect` plugin.

