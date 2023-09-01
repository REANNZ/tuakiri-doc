
Normally, the Shibboleth endpoints are accessible only via HTTPS (also configured by the `handlerSSL="true"` setting above). Applications that make use of (plain) http for access to content using Shibboleth protection can run into issues if the client is using inconsistent proxy connection settings for http and https.

By default Shibboleth SP checks that the IP address stays the same - but in this case, the IP address for the http and https traffic appears to be different. The safety mechanisms then suspect the session has been hijacked and terminate the session. This can lead to the SP keeping the user in an [infinite loop](https://wiki.shibboleth.net/confluence/display/SHIB2/NativeSPLooping).

For such applications we recommend setting `consistentAddress="false"` on the [<Sessions>](https://wiki.shibboleth.net/confluence/display/SHIB2/NativeSPSessions) element:

```
consistentAddress="false"
```

