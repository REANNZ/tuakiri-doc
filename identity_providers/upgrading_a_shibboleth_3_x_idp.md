---
redirect_from: Upgrading+a+Shibboleth+3.x+IdP
id: identity_providers/upgrading_a_shibboleth_3_x_idp
---
# Upgrading a Shibboleth 3.x IdP

Shibboleth IdP 3.x installer had been designed to make upgrades seamless by separating user-managed and system-managed files.

Rerunning the installer would update all system-managed files, while leaving the user-managed files untouched.

Based on upstream [upgrade instructions](https://wiki.shibboleth.net/confluence/display/IDP30/Upgrading), the basic steps are:

*   Get the new version:
    
    ```
    NEW_IDP_VERSION=3.x.x
    cd /root/inst
    wget http://shibboleth.net/downloads/identity-provider/${NEW_IDP_VERSION}/shibboleth-identity-provider-${NEW_IDP_VERSION}.tar.gz
    tar xzf shibboleth-identity-provider-${NEW_IDP_VERSION}.tar.gz
    cd shibboleth-identity-provider-${NEW_IDP_VERSION}
    ```
    
*   Run the installer (wrapped with a Tomcat restart) - and also fix permissions right after running the installer:
    
    ```
    service tomcat stop
    ./bin/install.sh
    chown -R tomcat.tomcat /opt/shibboleth-idp/
    service tomcat start
    ```
    
    Running the installer also rebuild the WAR file and may trigger reloading the web application.  As the web application reload may at times malfunction, we recommend temporarily stopping Tomcat while running the installer.
    
    *   Or to run these as a single command with minimal downtime:
        
        ```
        service tomcat stop ; ./bin/install.sh < /dev/null ; chown -R tomcat.tomcat /opt/shibboleth-idp/ ; service tomcat start
        ```
        
*   Review changes in default configuration files between the new and previous version:
    
    ```
    cd /root/inst
    OLD_IDP_VERSION=3.x.y
    diff -r -u shibboleth-identity-provider-${OLD_IDP_VERSION}/conf/ shibboleth-identity-provider-${NEW_IDP_VERSION}/conf/ | less
    ```
    
      
    
    *   Apply any relevant changes to the configuration files in `/opt/shibboleth-idp/conf` (e.g., to make use of new features).
    *   This page will be updated as future versions of the IdP are released to document which changes are required for correct operation within Tuakiri.  
          
        
*   Fix file permissions on IdP files:
    
    ```
    chown -R tomcat.tomcat /opt/shibboleth-idp
    
    # and for SELinux:
    restorecon -R /opt/shibboleth-idp
    ```
    
*   To make sure the WAR file gets updated with any changes to the content included in the web application, run the build script, combined with a restart of Tomcat
    
    ```
    service tomcat stop
    /opt/shibboleth-idp/bin/build.sh
    service tomcat start
    ```
    
*   The updated version of the IdP should be running
*   To properly record the change, edit `/etc/profile.d/shib.sh` and update `IDP_VERSION` to the new IdP version.
