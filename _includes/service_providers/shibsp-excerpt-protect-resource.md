
You can protect a resource with Shibboleth SP by adding the following directives into your Apache configuration. By default, a sample configuration snippet protecting the `/secure` URL on the server is included in `/etc/httpd/conf.d/shib.conf`:

```
<Location /secure>
  AuthType shibboleth
  ShibRequestSetting requireSession 1
  require shib-session
</Location>
```

You can add additional access control directives either to this file or anywhere else in the Apache configuration, as it fits with your application.

Another frequently used technique is _lazy sessions_ - access is granted also for unauthenticated users, but if a session exists, the attributes in the session are passed through to the application - and the application can then make access control decision (and initiate a login where needed).

Applying lazy sessions (making the Shibboleth sessions visible) to the whole application can be achieved e.g. with:

```
<Location />
  AuthType shibboleth
  ShibRequestSetting requireSession 0
  require shibboleth
</Location>
```

Apache 2.2 deployments

Because the way authentication modules (like `mod_shib`) link into Apache has changed substantially between Apache 2.2 and 2.4, the directives to protect a resource with mod\_shib has changed as well.

The module provides the `ShibCompatWith24` directive to emulate the Apache 2.4 behavior on Apache 2.2 and we recommend using this directive on new deployments (if they are with Apache 2.2) - the configuration will otherwise be ready for Apache 2.4.

However, this directive is **only** available with Apache 2.2 and is **not** available on Apache 2.4, so only use it on actual Apache 2.2 deployments.

![](https://reannz.atlassian.net/wiki/images/icons/grey_arrow_down.png)Click here to expand Apache 2.2-specific code snippets.

Protecting a resource with eager protection in Apache 2.2:

```
<Location /secure>
  AuthType shibboleth
  ShibCompatWith24 On
  ShibRequestSetting requireSession 1
  require shib-session
</Location>
```

Protecting a resource with lazy sessions in Apache 2.2:

```
<Location />
  AuthType shibboleth
  ShibCompatWith24 On
  ShibRequestSetting requireSession 0
  require shibboleth
</Location>
```

  

Note that in this case, to actually trigger a login, the application would have to redirect the user to a Session Initiator - a default one is located at `/Shibboleth.sso/Login`  (see the links below for more details).  
You are welcome to use the Tuakiri logo with the Login link - please visit our [Logos](https://reannz.atlassian.net/wiki/spaces/Tuakiri/pages/3815538811/Logos) page to get a suitably sized Tuakiri logo.

For further information, please see the following pages in the Shibboleth SP documentation:

*   Protecting a resource: basic concepts: [https://wiki.shibboleth.net/confluence/display/SHIB2/NativeSPProtectContent](https://wiki.shibboleth.net/confluence/display/SHIB2/NativeSPProtectContent)
*   Protecting a resource with Apache directives: [https://wiki.shibboleth.net/confluence/display/SHIB2/NativeSPhtaccess](https://wiki.shibboleth.net/confluence/display/SHIB2/NativeSPhtaccess)
*   Shibboleth SP Apache module configuration reference: [https://wiki.shibboleth.net/confluence/display/SHIB2/NativeSPApacheConfig](https://wiki.shibboleth.net/confluence/display/SHIB2/NativeSPApacheConfig)
*   Shibboleth SP configuration reference: [https://wiki.shibboleth.net/confluence/display/SHIB2/NativeSPConfiguration](https://wiki.shibboleth.net/confluence/display/SHIB2/NativeSPConfiguration)
*   Integrating Shibboleth SP with your application: [https://wiki.shibboleth.net/confluence/display/SHIB2/NativeSPEnableApplication](https://wiki.shibboleth.net/confluence/display/SHIB2/NativeSPEnableApplication)
*   Shibboleth configuration How-Tos: [https://wiki.shibboleth.net/confluence/display/SHIB/ConfigurationHowTos](https://wiki.shibboleth.net/confluence/display/SHIB/ConfigurationHowTos) 
*   Session creation parameters (when using a session initiator): [https://wiki.shibboleth.net/confluence/display/SHIB2/NativeSPSessionCreationParameters](https://wiki.shibboleth.net/confluence/display/SHIB2/NativeSPSessionCreationParameters)

