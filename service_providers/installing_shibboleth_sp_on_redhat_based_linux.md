---
redirect_from: Installing+Shibboleth+SP+on+RedHat+based+Linux
id: service_providers/installing_shibboleth_sp_on_redhat_based_linux
---
# Installing Shibboleth SP on RedHat based Linux

# Introduction

There is already a significant amount of documentation on installing a Shibboleth SP notably:

*   Understanding Shibboleth: how it all fits together: [https://wiki.shibboleth.net/confluence/display/CONCEPT/FlowsAndConfig](https://wiki.shibboleth.net/confluence/display/CONCEPT/FlowsAndConfig) (useful for terminology and understanding how the Shibboleth SP uses session cookies)

*   Installation: [https://wiki.shibboleth.net/confluence/display/SP3/Installation](https://wiki.shibboleth.net/confluence/display/SP3/Installation) (for installing on Linux, Mac, or Windows)

*   SWITCH SP Installation manual for Debian and Ubuntu: [https://www.switch.ch/aai/guides/sp/installation/](https://www.switch.ch/aai/guides/sp/installation/)

*   Configuration reference: [https://wiki.shibboleth.net/confluence/display/SP3/Configuration](https://wiki.shibboleth.net/confluence/display/SP3/Configuration)

This page draws on the above documents and gives the series of steps to install a Shibboleth SP and get it working in the Tuakiri federation.

This documentation now covers Shibboleth SP 3.x - though it does not significantly differ from 2.x for which this documentation was originally written.  To upgrade from 2.x to 3.2, please see our [Shibboleth SP 2.x to 3.x Upgrade Manual](https://reannz.atlassian.net/wiki/spaces/Tuakiri/pages/3815539872/Upgrading+2.x+Shibboleth+SP+to+3.x).

 This documentation has been tested on RHEL/CentOS 6 and 7, but should work on other RedHat-based systems as well.

We recommend installing the most recent Shibboleth SP version. Version 3.4.1 is the latest version as of June 2023. We recommend updating existing deployments to the most recent version to get fixes for known vulnerabilities - please see the list of [security advisories](https://wiki.shibboleth.net/confluence/display/SP3/SecurityAdvisories).

  

/\*<!\[CDATA\[\*/ div.rbtoc1693281281822 {padding: 0px;} div.rbtoc1693281281822 ul {list-style: disc;margin-left: 0px;} div.rbtoc1693281281822 li {margin-left: 0px;padding-left: 0px;} /\*\]\]>\*/

*   1 [Introduction](#InstallingShibbolethSPonRedHatbasedLinux-Introduction)
*   2 [Prerequisites](#InstallingShibbolethSPonRedHatbasedLinux-Prerequisites)
    *   2.1 [Firewall settings](#InstallingShibbolethSPonRedHatbasedLinux-Firewallsettings)
    *   2.2 [Time synchronization](#InstallingShibbolethSPonRedHatbasedLinux-Timesynchronization)
    *   2.3 [Dependencies](#InstallingShibbolethSPonRedHatbasedLinux-Dependencies)
    *   2.4 [SELinux](#InstallingShibbolethSPonRedHatbasedLinux-SELinux)
*   3 [Installation](#InstallingShibbolethSPonRedHatbasedLinux-Installation)
*   4 [Federation Membership](#InstallingShibbolethSPonRedHatbasedLinux-FederationMembership)
    *   4.1 [ECP support](#InstallingShibbolethSPonRedHatbasedLinux-ECPsupport)
    *   4.2 [Configuration](#InstallingShibbolethSPonRedHatbasedLinux-Configuration)
*   5 [Special considerations](#InstallingShibbolethSPonRedHatbasedLinux-Specialconsiderations)
    *   5.1 [HTTP/HTTPS access](#InstallingShibbolethSPonRedHatbasedLinux-HTTP/HTTPSaccess)
    *   5.2 [RedHat Enterprise Linux 6 and 7](#InstallingShibbolethSPonRedHatbasedLinux-RedHatEnterpriseLinux6and7)
    *   5.3 [ECP](#InstallingShibbolethSPonRedHatbasedLinux-ECP)
*   6 [Logging](#InstallingShibbolethSPonRedHatbasedLinux-Logging)
*   7 [Protecting a resource](#InstallingShibbolethSPonRedHatbasedLinux-Protectingaresource)
*   8 [Finishing up](#InstallingShibbolethSPonRedHatbasedLinux-Finishingup)
*   9 [Testing](#InstallingShibbolethSPonRedHatbasedLinux-Testing)

# Prerequisites

## Firewall settings

*   inbound traffic:
    *   webserver: port 80 and/or 443 are used by any browser-user
*   outbound:
    *   Shibboleth daemon (`shibd`): has to be able to connect to every remote IdP in the federation on port 8443 for back-channel communication.

## Time synchronization

The host where Shibboleth SP is running must have time synchronized.  We recommend using NTP for doing so - and synchronizing with your local NTP server.  An example of configuring NTP can be found in the [IdP Install Manual](https://reannz.atlassian.net/wiki/spaces/Tuakiri/pages/3815538813/Installing+a+Shibboleth+3.x+IdP#InstallingaShibboleth3.xIdP-Localconfiguration).

  

## Dependencies

Before starting to build and configure the Shibboleth Sevice Provider, be sure that the dependent packages (Apache, and the `mod_ssl` module for Apache) are installed:

```
yum install httpd mod_ssl
```

## SELinux

**Configuring SELinux to permit httpd-shibd communication**

  

These steps are required on an RHEL6/CentOS6 system with SELinux running in Enforcing mode - otherwise, `mod_shib` running inside Apache would not be able to communicate with `shibd`.

On RHEL7/CentOS7, the default SELinux policy already permits these actions and the following step is not required.

  

![](https://reannz.atlassian.net/wiki/images/icons/grey_arrow_down.png)Click here to expand the instructions to configure SELinux to permit httpd-shibd communication on a CentOS-6 system

To configure SELinux to allow Apache (where `mod_shib` is loaded) to connect to `shibd`:

*   Create a policy type enforcement file defining a policy module `mod_shib-to-shibd` - create `mod_shib-to-shibd.te` with the following contents:
    
    ```
    module mod_shib-to-shibd 1.0;
    
    require {
            type var_run_t;
            type httpd_t;
            type initrc_t;
            class sock_file write;
            class unix_stream_socket connectto;
    }
    
    #============= httpd_t ==============
    allow httpd_t initrc_t:unix_stream_socket connectto;
    allow httpd_t var_run_t:sock_file write;
    ```
    

*   Compile, package and load the module with:
    
    ```
    checkmodule -m -M -o mod_shib-to-shibd.mod mod_shib-to-shibd.te 
    semodule_package -o mod_shib-to-shibd.pp -m mod_shib-to-shibd.mod
    semodule -i mod_shib-to-shibd.pp 
    ```
    

# Installation

Shibboleth SP is available for RedHat and derivative distributions via yum repositories maintained by the Shibboleth Project.  The repository configuration files are generated by the shibbolet.net download site based on the target Linux distribution.  You can either download the `.repo`  file directly by passing in the distribution name as per the examples below, or you can download it via a browser from [https://shibboleth.net/downloads/service-provider/latest/RPMS/](https://shibboleth.net/downloads/service-provider/latest/RPMS/) and then copy it to the target system.

*   Add the `yum` repository for your distribution (example for CentOS 7):
    
    ```
    wget -O /etc/yum.repos.d/shibboleth.repo https://shibboleth.net/cgi-bin/sp_repo.cgi?platform=CentOS_7 
    ```
    
      
    The table below includes links for additional supported distributions (taken from the download form linked above)
    
    |     |     |
    | --- | --- |
    | Platform (OS) | URL |
    | CentOS 8 and RHEL 8 | [https://shibboleth.net/cgi-bin/sp\_repo.cgi?platform=CentOS\_8](https://shibboleth.net/cgi-bin/sp_repo.cgi?platform=CentOS_8) |
    | CentOS 7 and RHEL 7 | [https://shibboleth.net/cgi-bin/sp\_repo.cgi?platform=CentOS\_7](https://shibboleth.net/cgi-bin/sp_repo.cgi?platform=CentOS_7) |
    | Rocky Linux 8 | [https://shibboleth.net/cgi-bin/sp\_repo.cgi?platform=rockylinux8](https://shibboleth.net/cgi-bin/sp_repo.cgi?platform=rockylinux8) |
    | Rocky Linux 9 | [https://shibboleth.net/cgi-bin/sp\_repo.cgi?platform=rockylinux9](https://shibboleth.net/cgi-bin/sp_repo.cgi?platform=rockylinux9) |
    | Amazon Linux 2 | [https://shibboleth.net/cgi-bin/sp\_repo.cgi?platform=amazonlinux2](https://shibboleth.net/cgi-bin/sp_repo.cgi?platform=amazonlinux2) |
    | Amazon Linux 2023 | [https://shibboleth.net/cgi-bin/sp\_repo.cgi?platform=amazonlinux2023](https://shibboleth.net/cgi-bin/sp_repo.cgi?platform=amazonlinux2023) |
    
    RHEL 7 RPMs
    
    The Shibboleth Project provides binary packages for CentOS systems, but due to licensing restrictions, cannot build packages for RHEL 7 and above - full details are at [https://wiki.shibboleth.net/confluence/display/SP3/RPMInstall](https://wiki.shibboleth.net/confluence/display/SP3/RPMInstall).
    
    For RHEL 7 systems, please use the binary-compatible CentOS repository.
    
    For RHEL version 8 and above, Rocky Linux is a supported alternative.
    
*   Install latest version via `yum`:
    
    ```
    yum install shibboleth
    ```
    

# Federation Membership

{% include federation_management/adding_an_sp_to_tuakiri-excerpt %}

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
    
      
      
    

  

*   With earlier versions of Shibboleth SP (2.x), it was necessary to work around issues with rotation of logs generated by the `mod_shib`  module running inside Apache.  In Shibboleth SP 3.x, this module logs via syslog and this is no longer an issue.  If deploying a 2.x installation or explicitly logging to file, expand this section (otherwise archived for historical purposes only).
    
    ![](https://reannz.atlassian.net/wiki/images/icons/grey_arrow_down.png)Click here to expand...
    
    Workaround : move the log rotation from the module to logrotate.
    
    *   Otherwise, SELinux rules would not permit the log rotation (by design Apache is allowed to only _append_ to logs, but cannot remove them – incl. renaming).
    *   And there is also a race condition in the log rotation.  This has been reported upstream as [SSPCPP-757](https://issues.shibboleth.net/jira/browse/SSPCPP-757) - and we recommend to move log rotation out of `mod_shib` to `logrotate`.  
          
        
    *   Edit `/etc/shibboleth/native.logger`  and:
        *   replace `RollingFileAppender` with `FileAppender`
        *   comment out log rotation-specific options: `maxFileSize` and `maxBackupIndex`
        *   or just replace the file with our copy with exactly these customizations: [native.logger](https://github.com/REANNZ/Tuakiri-public/raw/master/shibboleth-sp/native.logger)
    *   Install a new file into `/etc/logrotate.d/shibboleth-www` to rotate these files via `logrotate` (and reload Apache post-rotate): [shibboleth-www](https://github.com/REANNZ/Tuakiri-public/raw/master/shibboleth-sp/logrotate-redhat/shibboleth-www) containing:
        
          
        
        ```
        /var/log/shibboleth-www/*.log {
            missingok
            daily
            rotate 10
            nodateext
            size 1000000
            sharedscripts
            postrotate
                /sbin/service httpd reload > /dev/null 2>/dev/null || true
            endscript
        }
        ```
        
    *   These can be both installed with:
        
        ```
        wget -O /etc/shibboleth/native.logger https://github.com/REANNZ/Tuakiri-public/raw/master/shibboleth-sp/native.logger
        wget -O /etc/logrotate.d/shibboleth-www https://github.com/REANNZ/Tuakiri-public/raw/master/shibboleth-sp/logrotate-redhat/shibboleth-www
        ```
        
    

# Special considerations

## HTTP/HTTPS access

Normally, the Shibboleth endpoints are accessible only via HTTPS (also configured by the `handlerSSL="true"` setting above). Applications that make use of (plain) http for access to content using Shibboleth protection can run into issues if the client is using inconsistent proxy connection settings for http and https.

By default Shibboleth SP checks that the IP address stays the same - but in this case, the IP address for the http and https traffic appears to be different. The safety mechanisms then suspect the session has been hijacked and terminate the session. This can lead to the SP keeping the user in an [infinite loop](https://wiki.shibboleth.net/confluence/display/SHIB2/NativeSPLooping).

For such applications we recommend setting `consistentAddress="false"` on the [<Sessions>](https://wiki.shibboleth.net/confluence/display/SHIB2/NativeSPSessions) element:

```
consistentAddress="false"
```

  

## RedHat Enterprise Linux 6 and 7

Please note that RedHat Enterprise Linux 6 and 7 (and so also CentOS 6 and 7) come with CURL built against NSS, not OpenSSL. Using this version of the CURL libraries would break the SOAP calls Shibboleth SP is making to the IdP port 8443 (back-channel communication) for artifact resolution and attribute queries.  While initial approach taken by the Shibboleth SP project was to provide CURL version linked against OpenSSL that would "upgrade" (replace) the one that comes with the OS, this was later seen as having undesired consequences and the new approach is to instead provide "look-aside" version of the library that installs into `/opt`.

These libraries install automatically as dependencies of the main shibboleth package and no action is needed by the deployer.

Further information is available in the upstream documentation at [https://wiki.shibboleth.net/confluence/display/SHIB2/NativeSPLinuxRH6](https://wiki.shibboleth.net/confluence/display/SHIB2/NativeSPLinuxRH6)

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

# Protecting a resource

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

  

*   Start up Apache and shibd:
    
    ```
     service httpd start
     service shibd start
     chkconfig httpd on
     chkconfig shibd on
    ```
    
    On RHEL7/CentOS7 using systemd, the commands should properly be:
    
    ```
    systemctl enable httpd shibd
    systemctl start httpd shibd
    ```
    
    (but the legacy syntax invoking `service` and `chkconfig` still works and is rerouted to systemctl)
    

# Testing

1.  Place a script inside the protected directory. PHP example script such as the following is good enough:
    
    ```
    <?php print_r($_SERVER) ?>
    ```
    
2.  Access the protected directory/script ([http://your.server/secure](http://your.server/secure)) from your browser, this should trigger a complete SSO cycle where you can authenticate on your IdP
3.  Upon successful authentication, the page should display all received attributes. Make sure you have non empty **Shib-Application-ID** amongst other attributes (if your IdP release them).
4.  Check your **shibd.log** to see if there are attributes received or errors encountered.
