---
redirect_from: Install+Shibboleth+SP+on+Debian+Based+linux
id: service_providers/install_shibboleth_sp_on_debian_based_linux
---
# Install Shibboleth SP on Debian Based linux

# Introduction

There is already a significant amount of documentation on installing a Shibboleth SP notably:

*   Understanding Shibboleth: how it all fits together: [https://wiki.shibboleth.net/confluence/display/CONCEPT/FlowsAndConfig](https://wiki.shibboleth.net/confluence/display/CONCEPT/FlowsAndConfig) (useful for terminology and understanding how the Shibboleth SP uses session cookies)

*   Installation: [https://wiki.shibboleth.net/confluence/display/SP3/Installation](https://wiki.shibboleth.net/confluence/display/SP3/Installation) (for installing on Linux, Mac, or Windows)

*   SWITCH SP Installation manual for Debian and Ubuntu: [https://www.switch.ch/aai/guides/sp/installation/](https://www.switch.ch/aai/guides/sp/installation/)

*   Configuration reference: [https://wiki.shibboleth.net/confluence/display/SP3/Configuration](https://wiki.shibboleth.net/confluence/display/SP3/Configuration)

This page draws on the above documents and gives the series of steps to install a Shibboleth SP and get it working in the Tuakiri federation.

This documentation now covers Shibboleth SP 3.x - though it does not significantly differ from 2.x for which this documentation was originally written.  To upgrade from 2.x to 3.2, please see our [Shibboleth SP 2.x to 3.x Upgrade Manual](https://reannz.atlassian.net/wiki/spaces/Tuakiri/pages/3815539872/Upgrading+2.x+Shibboleth+SP+to+3.x).

This documentation is written based on and tested on Ubuntu 20.04 Server x86\_64, but should work on other Debian-based distributions as well.

We recommend installing the most recent Shibboleth SP version. Version 3.4.1 is the latest version as of June 2023. We recommend updating existing deployments to the most recent version to get fixes for known vulnerabilities - please see the list of [security advisories](https://wiki.shibboleth.net/confluence/display/SP3/SecurityAdvisories).

Please note that SP 3.x has only been released for Ubuntu 20.04 and above and has not been released for older versions.  On hosts running older versions of Ubuntu, following this manual will install Shibboleth 2.6.x.

/\*<!\[CDATA\[\*/ div.rbtoc1693281190460 {padding: 0px;} div.rbtoc1693281190460 ul {list-style: disc;margin-left: 0px;} div.rbtoc1693281190460 li {margin-left: 0px;padding-left: 0px;} /\*\]\]>\*/

*   1 [Introduction](#InstallShibbolethSPonDebianBasedlinux-Introduction)
*   2 [Prerequsites](#InstallShibbolethSPonDebianBasedlinux-Prerequsites)
    *   2.1 [Firewall settings](#InstallShibbolethSPonDebianBasedlinux-Firewallsettings)
    *   2.2 [Dependencies](#InstallShibbolethSPonDebianBasedlinux-Dependencies)
    *   2.3 [Time synchronization](#InstallShibbolethSPonDebianBasedlinux-Timesynchronization)
*   3 [Installation](#InstallShibbolethSPonDebianBasedlinux-Installation)
*   4 [Federation Membership](#InstallShibbolethSPonDebianBasedlinux-FederationMembership)
    *   4.1 [ECP support](#InstallShibbolethSPonDebianBasedlinux-ECPsupport)
    *   4.2 [Configuration](#InstallShibbolethSPonDebianBasedlinux-Configuration)
*   5 [Special considerations](#InstallShibbolethSPonDebianBasedlinux-Specialconsiderations)
    *   5.1 [HTTP/HTTPS access](#InstallShibbolethSPonDebianBasedlinux-HTTP/HTTPSaccess)
    *   5.2 [ECP](#InstallShibbolethSPonDebianBasedlinux-ECP)
*   6 [Logging](#InstallShibbolethSPonDebianBasedlinux-Logging)
*   7 [Protecting a Resource](#InstallShibbolethSPonDebianBasedlinux-ProtectingaResource)
*   8 [Finishing up](#InstallShibbolethSPonDebianBasedlinux-Finishingup)
*   9 [Testing](#InstallShibbolethSPonDebianBasedlinux-Testing)

# Prerequsites

## Firewall settings

*   inbound traffic:
    *   webserver: port 80 and/or 443 are used by any browser-user
*   outbound:
    *   Shibboleth daemon (`shibd`): has to be able to connect to every remote IdP in the federation on port 8443 for back-channel communication.

  

## Dependencies

Before starting to build and configure the Shibboleth Service Provider, be sure that the Apache2 package is installed, and required modules (socache\_shmcb and ssl) are enabled:

```
sudo apt-get install -y apache2
sudo a2enmod ssl
sudo a2ensite default-ssl
```

## Time synchronization

The host where Shibboleth SP is running must have time synchronized.  We recommend using NTP for doing so - and synchronizing with your local NTP server.  An example of configuring NTP can be found in the [IdP Install Manual](https://reannz.atlassian.net/wiki/spaces/Tuakiri/pages/3815538813/Installing+a+Shibboleth+3.x+IdP#InstallingaShibboleth3.xIdP-Localconfiguration).

# Installation

Shibboleth SP is available in the standard Ubuntu and Debian repositories.  While the package stays at the major/minor version that was initially released with the Ubuntu/Debian version, it does get security updates backported.   However, as 18.04 only has ShibSP 2.6.1, we strongly recommend running at least Ubuntu 20.04 (which gets ShibSP 3.0.4).

Note that the [SWITCH repository](https://pkg.switch.ch/switchaai/) used to provide more up-to-date ShibSP packages for Ubuntu and Debian, but this repository has been decommissioned (no updates after 2021-11-30 and will be switched off after 2022-11-30).

  

The key steps are:

*   Install Shibboleth SP software:
    
    ```
    sudo apt-get install --install-recommends libapache2-mod-shib
    ```
    

*   Create backup copies of configuration files.  Contrary to other distributions, the SWITCH packages do not keep copies of the original files.  Make the pristine copies now, for easier tracking of future changes to the configuration files:
    
    ```
    for FILE in attribute-map.xml attribute-policy.xml protocols.xml security-policy.xml shibboleth2.xml native.logger shibd.logger ; do sudo cp /etc/shibboleth/$FILE /etc/shibboleth/$FILE.dist ; done
    ```
    

*   And also explicitly generate the back-channel key: substituting the publicly visible hostname of your SP for sp.example.org in this command:
    
    ```
    # for a Shibboleth 2.x system:
    shib-keygen -u _shibd -g _shibd -y 20 -h sp.example.org -e https://sp.example.org/shibboleth
    # or for a Shibboleth 3.x system:
    shib-keygen -n sp-signing -u _shibd -g _shibd -y 20 -h sp.example.org -e https://sp.example.org/shibboleth
    shib-keygen -n sp-encrypt -u _shibd -g _shibd -y 20 -h sp.example.org -e https://sp.example.org/shibboleth
    ```
    

# Federation Membership

{% federation_management/include adding_an_sp_to_tuakiri-excerpt %}

## Configuration

Edit `/etc/shibboleth/shibboleth2.xml:`

*   Replace all instances of `sp.example.org` with your hostname.

*   In the [<Sessions>](https://wiki.shibboleth.net/confluence/display/SHIB2/NativeSPSessions)element:
    *   Make session handler use SSL: set `handlerSSL="true"`  
        **Recommended**: go even further and in the `Sessions` element, change the `handlerURL` from a relative one (`"/Shibboleth.sso"` to an absolute one - `handlerURL="[https://sp.example.org/Shibboleth.sso](https://sp.example.org/Shibboleth.sso)"`. In the URL, use the hostname used in the endpoint URLs registered in the Federation Registry. This makes sure the server is always issuing correct endpoint URLs in outgoing requests, even when users refer to the server with alternative names. This is in particular important when there are multiple hostnames resolving to your server (such as one prefixed with "www." and one without).
    *   We also strongly recommend to configure the SP to use **secure**  cookies that would only be sent over an encrypted (https) connection.  Unless you are also using plain HTTP to access your application in authenticated mode (which is dangerous - risk of cookie theft / session hijacking), change the `cookieProps`  setting to use secure cookies:
        
        ```
        cookieProps="https"
        ```
        
    *   **IMPORTANT**: To prevent your server becoming an [Open Redirect](https://cwe.mitre.org/data/definitions/601.html), restrict the URLs acceptable as redirect targets to the same base as your server by adding the following to the Sessions element (if not already present - included in new configuration files from Shibboleth SP 3.1 onwards).  For further details, see the [Sessions element documentation](https://wiki.shibboleth.net/confluence/display/SP3/Sessions).
        
        ```
        redirectLimit="exact"
        ```
        
    *   Configure Session Initiator: locate the `<SSO>`element and:
        *   Remove reference to default `idp.example.org` - delete the `entityID` attribute
        *   Configure the Discovery Service URL in the `discoveryURL` attribute:
            
            ```
            discoveryURL="https://directory.tuakiri.ac.nz/ds/DS"
            ```
            
        *   or, alternatively, if connecting to the Tuakiri TEST federation (Staging Environment), use:
            
            ```
            discoveryURL="https://directory.test.tuakiri.ac.nz/ds/DS"
            ```
            

*   In `AttributeExtractor`, set `reloadChanges="true"`  
      
    
*   Shibboleth 2.x only: restrict cipherSuites:
    
    ![](https://reannz.atlassian.net/wiki/images/icons/grey_arrow_down.png)Click here to expand...
    
    In earlier versions (Shibboleth SP 2.x), we were recommending to configure the TLS protocols and cipher-suites acceptable on the back-channel - the [default settings](https://wiki.shibboleth.net/confluence/display/SHIB2/NativeSPApplication#NativeSPApplication-RelyingPartyAttributes) were overly permissive and insecure.
    
    Shibboleth 3.x now sets a new default, identical to our recommendation in terms of actual ciphers permitted.  So, this step is no longer needed on Shibboleth SP 3.x
    
    On Shibboleth SP 2.x, add the following XML attribute to the `<ApplicationDefaults>` element:
    
    ```
    cipherSuites="DEFAULT:!EXP:!SSLv2:!DES:!IDEA:!SEED:!RC4:!3DES:!kRSA:!SSLv3:!TLSv1:!TLSv1.1"
    ```
    
    This sets the protocols to TLSv1.2 only (banning SSLv2, SSLv3, TLSv1.0, TLSv1.1) and blocks all ciphers deemed insecure (as of October 2017).
    
*   Optionally, customize settings in the `<Errors>` element.  These settings configure the error handling pages that would be rendered to the users should an error occur.  At the very least, we recommend changing the `supportContact` attribute from `root@localhost` to your support service email address.  Documentation for advanced configuration of error handling is available at the [Shibboleth SP Errors documentation page](https://wiki.shibboleth.net/confluence/display/SHIB2/NativeSPErrors#NativeSPErrors-ConfigurationReference).

  

*   Download the metadata signing certificate for the federation metadata into `/etc/shibboleth`:
    *   For Tuakiri, run:
        
        ```
        wget https://directory.tuakiri.ac.nz/metadata/tuakiri-metadata-cert.pem -O /etc/shibboleth/tuakiri-metadata-cert.pem
        ```
        
    *   or for Tuakiri-TEST, run:
        
        ```
        wget https://directory.test.tuakiri.ac.nz/metadata/tuakiri-test-metadata-cert.pem -O /etc/shibboleth/tuakiri-test-metadata-cert.pem
        ```
        

*   Load the federation metadata: add the following (or equivalent) section into `/etc/shibboleth/shibboleth2.xml` just above the sample (commented-out) `MetadataProvider`element.
    *   For Tuakiri add:
        
        ```
                <MetadataProvider type="XML" url="https://directory.tuakiri.ac.nz/metadata/tuakiri-metadata-signed.xml"
                        backingFilePath="metadata.tuakiri.xml" reloadInterval="7200" validate="true">
                    <MetadataFilter type="RequireValidUntil" maxValidityInterval="2419200"/>
                    <MetadataFilter type="Signature" certificate="tuakiri-metadata-cert.pem" verifyBackup="false"/>
                </MetadataProvider>
        ```
        
    *   For Tuakiri-TEST, add instead:
        
        ```
                <MetadataProvider type="XML" url="https://directory.test.tuakiri.ac.nz/metadata/tuakiri-test-metadata-signed.xml"
                        backingFilePath="metadata.tuakiri-test.xml" reloadInterval="7200" validate="true">
                    <MetadataFilter type="RequireValidUntil" maxValidityInterval="2419200"/>
                    <MetadataFilter type="Signature" certificate="tuakiri-test-metadata-cert.pem" verifyBackup="false"/>
                </MetadataProvider>
        ```
        

  

*   The Shibboleth SP installation needs to be configured to map attributes received from the IdP - in `/etc/shibboleth/attribute-map.xml`. Change the attribute mapping definition by either editing the file and uncommenting attributes to be accepted, or replace the file with the recommended **Tuakiri** **[attribute-map.xml](https://github.com/REANNZ/Tuakiri-public/raw/master/shibboleth-sp/attribute-map.xml)** **file mapping** **_all_** **Tuakiri attributes** (and optionally comment out those attributes not used by your SP). This can be conveniently done with
    
    ```
    wget -O /etc/shibboleth/attribute-map.xml https://github.com/REANNZ/Tuakiri-public/raw/master/shibboleth-sp/attribute-map.xml
    ```
    
    In addition to mapping received attributes to local names (and thus accepting them), it is also possible to configure filtering rules in `attribute-policy.xml`.
    
    In most cases, this can be left as-is (the default rules do the filtering applicable to Tuakiri attributes), but additional rules can be added here.
    
    For further information, please see [https://wiki.shibboleth.net/confluence/display/SHIB2/NativeSPAttributeFilter](https://wiki.shibboleth.net/confluence/display/SHIB2/NativeSPAttributeFilter)
    
      
      
    

Log file race condiiton

With earlier versions of Shibboleth SP (2.x), it was necessary to work around issues with rotation of logs generated by the `mod_shib`  module running inside Apache.  In Shibboleth SP 3.x, this module logs via syslog and this is no longer an issue. 

On hosts that already have SP 3.x (like Ubuntu 20.04), this part can be skipped.

If deploying a 2.x installation (e.g., installing on a distribution that SP 3.x has not been released for yet, like Ubuntu 18.04), or explicitly logging to file, follow these instructions.

![](https://reannz.atlassian.net/wiki/images/icons/grey_arrow_down.png)Click here to expand...

  

*   To work around issues with rotation with logs generated by the `mod_shib`  module running inside Apache, it is necessary to move the log rotation from the module to logrotate.
    *   There is a race condition in the log rotation.  This has been reported upstream as [SSPCPP-757](https://issues.shibboleth.net/jira/browse/SSPCPP-757) - and we recommend to move log rotation out of `mod_shib` to `logrotate`.  
          
        
    *   Edit `/etc/shibboleth/native.logger`  and:
        *   replace `RollingFileAppender` with `FileAppender`
        *   comment out log rotation-specific options: `maxFileSize` and `maxBackupIndex`
        *   or just replace the file with our copy with exactly these customizations: [native.logger](https://github.com/REANNZ/Tuakiri-public/raw/master/shibboleth-sp/native.logger)
    *   Install a new file into `/etc/logrotate.d/shibboleth-www` to rotate these files via `logrotate` (and reload Apache post-rotate): [shibboleth-www](https://github.com/REANNZ/Tuakiri-public/raw/master/shibboleth-sp/logrotate-debian/shibboleth-www) containing:
        
          
        
        ```
        /var/log/shibboleth-www/*.log {
            missingok
            daily
            rotate 10
            nodateext
            size 1000000
            sharedscripts
            postrotate
                /usr/sbin/service apache2 reload > /dev/null 2>/dev/null || true
            endscript
        }
        ```
        
    *   These can be both installed with:
        
        ```
        wget -O /etc/shibboleth/native.logger https://github.com/REANNZ/Tuakiri-public/raw/master/shibboleth-sp/native.logger
        wget -O /etc/logrotate.d/shibboleth-www https://github.com/REANNZ/Tuakiri-public/raw/master/shibboleth-sp/logrotate-debian/shibboleth-www
        ```
        

  

# Special considerations

## HTTP/HTTPS access

Normally, the Shibboleth endpoints are accessible only via HTTPS (also configured by the `handlerSSL="true"` setting above). Applications that make use of (plain) http for access to content using Shibboleth protection can run into issues if the client is using inconsistent proxy connection settings for http and https.

By default Shibboleth SP checks that the IP address stays the same - but in this case, the IP address for the http and https traffic appears to be different. The safety mechanisms then suspect the session has been hijacked and terminate the session. This can lead to the SP keeping the user in an [infinite loop](https://wiki.shibboleth.net/confluence/display/SHIB2/NativeSPLooping).

For such applications we recommend setting `consistentAddress="false"` on the [<Sessions>](https://wiki.shibboleth.net/confluence/display/SHIB2/NativeSPSessions) element:

```
consistentAddress="false"
```

## ECP

If your SP should support [ECP](https://reannz.atlassian.net/wiki/spaces/Tuakiri/pages/3815538794/ECP) (access via non-browser clients), then also:

1.  Edit the <SSO> element in `/etc/shibboleth/shibboleth2.xml` and add an `ECP="true"`attribute:
    
    ```
    <SSO ECP="true" ....>
    ```
    
2.  Add support for ECP in the metadata registered in the federation (as instructed above).

# Logging

Shibboleth SP has two separate components (the `shibd` daemon and the `mod_shib` module running inside Apache), and they also have separate logging configuration and destinations.

*   The `shibd` daemon logs primarily into `/var/log/shibboleth/shibd.log` (with transaction details in `/var/log/shibboleth/transaction.log`)  
    *   Logging configuration is in `/etc/shibboleth/shibd.logger`
    *   Log files should be owned by `shibd` (the user account `shibd` daemon runs under)
*   The `mod_shib` Apache module logs into syslog (as facility `LOCAL0` ).  
    *   Logging configuration in `/etc/shibboleth/native.logger`
    *   In Shibboleth SP 2.x, `mid_shib` was logging into `/var/log/shibboleth-www/native.log` and `/var/log/shibboleth-www/native-warn.log`Log (and these files were owned by `apache`, the user account Apache httpd runs under)

# Protecting a Resource

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

# Finishing up

The shibboleth packages make shibd automatically active and enabled - so the only step required is restart Apache and Shibd after completing all changes:

```
sudo service apache2 restart
sudo service shibd restart
```

or via systemd:

```
sudo systemctl restart shibd apache2
```

  

# Testing

1.  Place a script inside the protected directory. PHP example script such as the following is good enough:
    
    ```
    <?php print_r($_SERVER) ?>
    ```
    
2.  Access the protected directory/script ([http://your.server/secure](http://your.server/secure)) from your browser, this should trigger a complete SSO cycle where you can authenticate on your IdP
3.  Upon successful authentication, the page should display all received attributes. Make sure you have non empty **Shib-Application-ID** amongst other attributes (if your IdP release them).
4.  Check your **shibd.log** to see if there are attributes received or errors encountered.
