# Upgrading a 3.x IdP to 4.x

IdP 4.0.0 was released in March 2020 and IdP 3.x will be declared End-Of-Life on December 31st, 2020.   All production IdPs thus have to be upgraded to 4.x.

The [upstream upgrade documentation](https://wiki.shibboleth.net/confluence/display/IDP4/Upgrading) strongly recommends doing an in-place upgrade to 4.x - and given the amount of changes in 4.x, we follow this recommendation and recommend Tuakiri members do the upgrade in-place.

Already in 3.x, the process for in-place upgrades has been well tuned - and the upgrade to 4.x should be seamless as well, as long as prior to the 4.x upgrade, the IdP runs the latest 3.x (3.4.8 or 3.4.7) with no **Deprecation warnings**.

/\*<!\[CDATA\[\*/ div.rbtoc1693282829315 {padding: 0px;} div.rbtoc1693282829315 ul {list-style: disc;margin-left: 0px;} div.rbtoc1693282829315 li {margin-left: 0px;padding-left: 0px;} /\*\]\]>\*/

*   1 [Background](#Upgradinga3.xIdPto4.x-Background)
    *   1.1 [In-place upgrade vs. new install](#Upgradinga3.xIdPto4.x-In-placeupgradevs.newinstall)
*   2 [Prerequisites](#Upgradinga3.xIdPto4.x-Prerequisites)
    *   2.1 [Java](#Upgradinga3.xIdPto4.x-Java)
    *   2.2 [Application Container](#Upgradinga3.xIdPto4.x-ApplicationContainer)
    *   2.3 [Shibboleth IdP versions](#Upgradinga3.xIdPto4.x-ShibbolethIdPversions)
    *   2.4 [SharedToken module](#Upgradinga3.xIdPto4.x-SharedTokenmodule)
    *   2.5 [Migrating from JPAStorageService to JDBCStorageService](#Upgradinga3.xIdPto4.x-MigratingfromJPAStorageServicetoJDBCStorageService)
*   3 [Upgrade](#Upgradinga3.xIdPto4.x-Upgrade)

# Background

IdP 3.x introduced a simplified upgrade process for in-place upgrades - where essentially, just running the installer from the new version distribution directory is sufficient.  The installer keeps the existing local configuration (primarily the `conf` directory, but also e.g. `credentials`  and `metadata` ), while updating the system files (primarily the `system` directory), replacing them with newer versions.  This highlights the fact that any local configuration should be done in the `conf` directory only, not touching files under `system` , as any changes made in the `system` directory are lost on upgrades.  But also makes the upgrades very easy to run.

Any new configuration files that do not exist in the `conf` directory yet are copied there.

After completing the upgrade, the `dist/conf`  directory contains a pristine copy of each configuration file as of the release being upgraded to.  And all files modified in the upgrade are kept under `/opt/shibboleth-idp/old-<timestamp>` , (including the `dist/conf`  directory), allowing to compare configuration files against the pristine versions they were based on.

## In-place upgrade vs. new install

The [upstream upgrade documentation](https://wiki.shibboleth.net/confluence/display/IDP4/Upgrading) strongly recommends doing an in-place upgrade to 4.x - and given the amount of changes in 4.x, we follow this recommendation and recommend Tuakiri members do the upgrade in-place.

There are significant changes in how the IdP 4.x is configured.  When an IdP is upgraded from an 3.x install, the configuration retains the "3.x style" and is compatible with 3.x-style configuration files.

A fresh 4.x install would not activate the 3.x style and just copying 3.x-style configuration files (such as `attribute-resolver.xml` ) into a 4.x install will not work.

The differences between 3.x and 4.x include:

*   4.x uses a new configuration subsystem, the Attribute Registry, to define and configure attributes.  The Attribute Registry includes encoders - which 3.x defines in attribute-resolver.xml.  A clash between 3.x/4.x configuration styles here might result into having attributes duplicated in outgoing SAML assertions.
*   4.x uses a new default for encryption of SAML Assertions (GCM), which may cause issues with some SPs (not being able to decrypt).  Rolling out the change of encryption ciphersuite should be separate from the IdP upgrade.
*   4.x stores all secrets in a separate file, `credentials/secrets.properties` and other configuration files that need the secrets refer to the properties defined in this file.  (This includes LDAP connection credentials).

We will later provide also an installation manual for a clean 4.x install - but for now, we recommend proceeding with an upgrade from the existing 3.x installation.

# Prerequisites

## Java

While Shibboleth IdP 3.x was primarily targetting Java8 (and accepting Java7 as well), Shibboleth IdP requires Java 11.

We recommend OpenJDK 11, on CentOS available as: `java-11-openjdk-devel` 

## Application Container

The Shibboleth IdP web application runs in an Application Container, typically Tomcat or Jetty.  IdPv4 requires a newer version of the application container - Tomcat7 which was frequently used for IdP 3.x deployments will not work with IdP 4, Tomcat 8.5 is confirmed to work (though official requirements asks for Tomcat 9).  For Jetty, the required version is 9.4.

Unfortunately, on CentOS 7, only Tomcat 7 is available in the base OS repository.  And CentOS 8 does not come with any Tomcat version at all. Neither do any community-run repositories provide RPM packages for up-to-date releases of neither Tomcat nor Jetty.

There are many blog posts providing instructions for manual one-off installs.   We provide a set of instructions here as convenience - with the aim to make future updates to newer Tomcat version easier (using separate per-version directories for Tomcat binary distribution, but a shared directory for application context-descriptor files).  Feel free to use these - but also to install Tomcat or Jetty via other means.

![](https://reannz.atlassian.net/wiki/images/icons/grey_arrow_down.png)Click here to expand instructions to install newer version of Tomcat on RedHat-based systems.

The following instructions cover the installation of Tomcat 8 (8.5) on CentOS 7 - the same set of instructions should also work on CentOS 8.

The instructions are designed to work either when the Tomcat 7 RPM is already installed or not - and are designed to make future updates to newer Tomcat version easier - e.g., while there are per-version directories for Tomcat distribution, the directory for application context-descriptor files is shared across the multiple Tomcat versions.

To install Tomcat7:

*   Stop Tomcat if it's currently running:
    
    ```
    service tomcat stop
    ```
    
*   Make sure Tomcat account exists:
    
    ```
    id tomcat || useradd --system --comment "Apache Tomcat" --shell /sbin/nologin tomcat
    ```
    
*   Create directory hierarchy:
    
    ```
    mkdir -p /opt/tomcat /opt/tomcat/context
    chown -R tomcat.tomcat /opt/tomcat
    ```
    
*   Download and deploy Tomcat binary:
    
    ```
    TOMCAT_VERSION=8.5.60
    cd /opt/tomcat
    wget https://archive.apache.org/dist/tomcat/tomcat-8/v${TOMCAT_VERSION}/bin/apache-tomcat-${TOMCAT_VERSION}.tar.gz
    tar xzf apache-tomcat-${TOMCAT_VERSION}.tar.gz
    # fix up permissons overall
    chown -R tomcat.tomcat /opt/tomcat/apache-tomcat-${TOMCAT_VERSION}
    chmod -R +r /opt/tomcat/apache-tomcat-${TOMCAT_VERSION}
    # fix up permissions on dirs
    chmod +rx /opt/tomcat/apache-tomcat-${TOMCAT_VERSION}/{.,bin,conf,lib,logs,temp,webapps,work}
    ln -s /opt/tomcat/apache-tomcat-${TOMCAT_VERSION} /opt/tomcat/current
    ```
    
*   Create override settings:
    
    ```
    cat > /opt/tomcat/current/bin/setenv.sh <<EOF
    CATALINA_PID=/run/tomcat/tomcat.pid
    UMASK=0022
    JAVA_HOME=/usr/lib/jvm/java-11-openjdk
    JAVA_OPTS="-Xms1024m -Xmx2048m"
    EOF
    ```
    
*   Configure Tomcat: edit `/opt/tomcat/current/conf/server.xml` and:  
    *   comment out port 8080 HTTP Connector
    *   add port 8009 AJP connector:
        
        ```
        <Connector port="8009" address="127.0.0.1"
        enableLookups="false" redirectPort="443" protocol="AJP/1.3"
        tomcatAuthentication="false"
        secretRequired="false" />
        ```
        
    *   in Host element, set `autoDeploy="false"`  (change from `true` to avoid reloads when touching the `idp.war` file)
    *   in Host element, set `xmlBase="/opt/tomcat/context"`  - to use version-independent context descriptor directory.
*   For use with Shibboleth IdP, download JSTL jar files:
    
    ```
    wget -O /opt/tomcat/current/lib/javax.servlet.jsp.jstl-api-1.2.1.jar 'http://search.maven.org/remotecontent?filepath=javax/servlet/jsp/jstl/javax.servlet.jsp.jstl-api/1.2.1/javax.servlet.jsp.jstl-api-1.2.1.jar'
    wget -O /opt/tomcat/current/lib/javax.servlet.jsp.jstl-1.2.1.jar 'http://search.maven.org/remotecontent?filepath=org/glassfish/web/javax.servlet.jsp.jstl/1.2.1/javax.servlet.jsp.jstl-1.2.1.jar'
    ```
    
*   Assuming MySQL JDBC driver is already installed locally and deployed applications will need it, symlink it into Tomcat lib directory:
    
    ```
    ln -s /usr/share/java/mysql-connector-java.jar /opt/tomcat/current/lib/
    ```
    
*   Create systemd unit file (this file will override the unit file shipped with the Tomcat RPM if installed).  (This block assumes `TOMCAT_VERSION` is still set as per above).
    
    ```
    cat > /etc/systemd/system/tomcat.service <<EOF
    [Unit]
    Description=Tomcat ${TOMCAT_VERSION}
    After=network.target
    
    [Service]
    Type=forking
    PIDFile=/run/tomcat/tomcat.pid
    User=tomcat
    Group=tomcat
    ExecStart=/opt/tomcat/current/bin/startup.sh
    ExecStop=/opt/tomcat/current/bin/shutdown.sh
    RuntimeDirectory=tomcat
    
    # TERM to main process only, KILL to all
    KillMode=mixed
    
    #restart: in case of failure, try starting every 30 seconds
    Restart=always
    RestartSec=30
    
    [Install]
    WantedBy=multi-user.target
    EOF
    ```
    
*   Deploy the IdP web-app into this Tomcat: create {{/opt/tomcat/context/idp.xml}} with:
    
    ```
    <Context docBase="/opt/shibboleth-idp/war/idp.war"
             unpackWAR="true"
             swallowOutput="true" />
    ```
    
*   Reload systemd config and start Tomcat:
    
    ```
    systemctl daemon-reload
    systemctl enable tomcat
    systemctl start tomcat
    ```
    

Only proceed further after upgrading the application container to a compatible version - Tomcat 8.5 or later or Jetty 9.4

  

## Shibboleth IdP versions

The upstream documentation states very clearly that 4.x upgrades are supported only from latest 3.x, running with no Deprecation warnings.  The IdP 3.x documentation provides the full list of [IdPv4 deprecations](https://wiki.shibboleth.net/confluence/display/IDP30/DeprecatedIdPV4), and this page also links to documentation covering how to deal with these deprecations.

So first, [upgrade to the latest 3.x](https://reannz.atlassian.net/wiki/spaces/Tuakiri/pages/3815539011/Upgrading+a+Shibboleth+3.x+IdP) (3.4.8 as of December 2020, but 3.4.7 will be sufficient - 3.4.8 did not introduce any new deprecation warnings).

After the upgrade, watch the logs while restarting the IdP and logging into the Attribute Validator ( [https://attributes.tuakiri.ac.nz/](https://attributes.tuakiri.ac.nz/) or [https://attributes.test.tuakiri.ac.nz/](https://attributes.test.tuakiri.ac.nz/) )

If the IdP is loading the Attribute Filter files generated by Tuakiri, you will get a Deprecation warning for the attribute filter.  We recommend switching to the new static attribute filter files ( `metadata-based-attribute-filter.xml`  and  `rns-attribute-filter.xml`  ), developed for eduGAIN connectivity.  Follow the instructions for configuring Attribute Release at [Configuring an IdP for eduGAIN](https://reannz.atlassian.net/wiki/spaces/Tuakiri/pages/3815539032/Configuring+an+IdP+for+eduGAIN#ConfiguringanIdPforeduGAIN-ConfiguringAttributeRelease).  This will get your IdP ready for eduGAIN (together with loading the eduGAIN metadata), and will provide the same functionality for existing Tuakiri SPs as the previously used Attribute Filter file.  (A very special exception is if your IdP relied on rules for releasing individual `eduPersonEntitlement` values with a regular expression filter - please contact us if this is the case for your IdP).

Please contact us if you need help removing any other Deprecation warnings.

Proceed further only after the IdP restart + Attribute Validator cycle comes through clean with no Deprecation warnings.

## SharedToken module

The IdP plugin module that generates the auEduPersonSharedToken attribute also has to be upgraded - the version released earlier for IdP 3.4.x is not compatible with IdP 4.x.  A new version, 2.0.x (2.0.3 as of February 3rd, 2021), has been released for IdP 4.x.

However, this new version, 2.0.x, is only compatible with 4.x but not 3.x - so the module will have to be upgraded as part of the IdP upgrade and not earlier.

The new version can be downloaded from [https://github.com/REANNZ/arcs-shibext/releases/download/2.0.3/arcs-shibext-2.0.3.jar](https://github.com/REANNZ/arcs-shibext/releases/download/2.0.3/arcs-shibext-2.0.3.jar)

The new version also introduces syntax changes in how it is configured - the single but significant change is replacing the custom `DatabaseConnection` element with a reference to a `DataSource` defined in other parts of the IdP configuration.

Please see the next section for detailed instructions.

## Migrating from JPAStorageService to JDBCStorageService

IdP 4.1.0 introduced support for IdP Plugins, which are versioned independently of the IdP itself.

IdP 4.3.0 marks built-in JPAStorageService as deprecated and offers JDBCStorageService as a suitable replacement.  The [Installing a Shibboleth 3.x IdP](https://reannz.atlassian.net/wiki/spaces/Tuakiri/pages/3815538813/Installing+a+Shibboleth+3.x+IdP) manual (that this guide recommends to upgrade from) gives instructions for configuring Database Storage based on the JPAStorageService.

After upgrading to 4.3.0, the IdP (if using JPAStorageService) will start emitting deprecation warnings (surprisingly, in Tomcat log, not IdP log): 

```
[ WARN] : DEPRECATED: Java class 'JPAStorageService': This will be removed in the next major version of this software; replacement is JDBCStorageService
```

This will have to be resolved before upgrading to next major version, so IdP 5.

The steps, as per in the [JDBCStorageService documentation](https://shibboleth.atlassian.net/wiki/spaces/IDPPLUGINS/pages/2989096970/JDBCStorageService), are:

*   Install the JDBCStorageService plugin with: 
    
    ```
    /opt/shiboleth-idp/bin/plugin.sh -I net.shibboleth.plugin.storage.jdbc
    ```
    
*   Make sure your database uses case sensitive primary key for the `StorageRecords` table.  On MySQL, when using `utf8` CHARACTER SET, it is crucial to set COLLATION to utf8\_bin.  
    This got added to the [Installing a Shibboleth 3.x IdP](https://reannz.atlassian.net/wiki/spaces/Tuakiri/pages/3815538813/Installing+a+Shibboleth+3.x+IdP) manual only in a later revision.  The SQL command to ensure the correct collation is: 
    
    ```
    ALTER TABLE StorageRecords CONVERT TO CHARACTER SET utf8 COLLATE utf8_bin;
    ```
    
*   Change the configuration in `/opt/shibboleth-idp/conf/global.xml` to:
    *   remove the shibboleth.JPAStorageService.EntityManagerFactory and shibboleth.JPAStorageService.JPAVendorAdapter beans
    *   change the definition of the StorageService bean from  
        `class="org.opensaml.storage.impl.JPAStorageService"`    
        to  
        `parent="shibboleth.JDBCStorageService"` 
    *   replace the constructor parameter  
        `c:factory-ref="shibboleth.JPAStorageService.EntityManagerFactory"`   
        with DataSource reference:  
        `p:dataSource-ref="shibboleth.JPAStorageService.DataSource"` 
*   After the change, the remaining database storage config should look like: 
    
    ```
        <bean id="shibboleth.JPAStorageService"
            parent="shibboleth.JDBCStorageService"
            p:cleanupInterval="%{idp.storage.cleanupInterval:PT10M}"
            p:dataSource-ref="shibboleth.JPAStorageService.DataSource" />
    
        <bean id="shibboleth.JPAStorageService.DataSource"
            ....
        />
    ```
    

# Upgrade

*   Get the new version:
    
    ```
    NEW_IDP_VERSION=4.0.1
    cd /root/inst
    wget http://shibboleth.net/downloads/identity-provider/${NEW_IDP_VERSION}/shibboleth-identity-provider-${NEW_IDP_VERSION}.tar.gz
    tar xzf shibboleth-identity-provider-${NEW_IDP_VERSION}.tar.gz
    cd shibboleth-identity-provider-${NEW_IDP_VERSION}
    ```
    
*   Start the upgrade: stop Tomcat:
    
    ```
    service tomcat stop
    ```
    
*   Run the installer (with the correct JDK) - and also fix permissions right after running the installer:
    
    ```
    JAVA_HOME=/usr/lib/jvm/java-11-openjdk ./bin/install.sh
    chown -R tomcat.tomcat /opt/shibboleth-idp/
    # and for SELinux:
    restorecon -R /opt/shibboleth-idp
    ```
    
*   Update LDAP connector definition.  Besides the settings deprecated in IdP 3.x and removed in IdP 4.x, there are also settings deprecated in IdP 4.x to be removed in a future version (possibly 5.x).  Now is a good time to make the changes and have the IdP running without any deprecation warnings.  In `/opt/shibboleth-idp/conf/attribute-resolver.xml` , make the following changes:
    *   Check the `ConnectionPool` element inside the `LDAPDirectory` `DataConnector` element - and remove any **failFastInitialize** attribute it might have.
    *   Check the `LDAPDirectory` `DataConnector` element for any `LDAPProperty` elements.  These elements have been deprecated without a generic replacement, but some specific properties can be mapped to `LDAPDirectory` element attributes.  See the [LDAPDirectory element documentation](https://wiki.shibboleth.net/confluence/display/IDP4/LDAPConnector) for further information.  
        *   Replace `java.naming.ldap.attributes.binary` property with the `BinaryAttributes` element, with the space-delimited list of LDAP attribute names as the text inside the element - e.g.:
            
            ```
            <BinaryAttributes>object GUID objectSid<BinaryAttributes/>
            ```
            
        *   Remove/comment-out `<LDAPProperty name="java.naming.referral" value="follow"/>` .  This property can be replaced with the boolean attribute `followReferrals` on the LDAPDirectory element.  However, this property was only needed with IdP software up to and including 2.x - and has been ignored by IdP 3.x and 4.x.  
            Actually turning the referral following on might have undesired side-effects (IdP attempting to connect to other trees in the AD forest, potentially failing - and reporting a warning) - unless it is needed for the IdP operation, we recommend leaving it off.
            
            ![](https://reannz.atlassian.net/wiki/images/icons/grey_arrow_down.png)Click here to expand the instructions to turn followReferrals on...
            
            ```
            <DataConnector id="myLDAP" xsi:type="LDAPDirectory"
                followReferrals="true"
                ...
            />
            ```
            
*   Upgrade SharedToken module:
    *   Download new version:
        
        ```
        wget -P /opt/shibboleth-idp/edit-webapp/WEB-INF/lib https://github.com/REANNZ/arcs-shibext/releases/download/2.0.3/arcs-shibext-2.0.3.jar
        ```
        
    *   Remove old version(s):
        
        ```
        rm /opt/shibboleth-idp/edit-webapp/WEB-INF/lib/arcs-shibext-1.*.jar
        ```
        
    *   Update definition: in `/opt/shibboleth-idp/conf/attribute-resolver.xml` , locate the `SharedToken` `DataConnector` and replace the `DatabaseConnection` element with a DataConnector attribute `databaseConnectionID` referening to a DataSource defined in `/opt/shibboleth-idp/conf/global.xml` (as part of the [IdP 3.x installation Database Storage setup](https://reannz.atlassian.net/wiki/spaces/Tuakiri/pages/3815538813/Installing+a+Shibboleth+3.x+IdP#InstallingaShibboleth3.xIdP-ConfigureDatabaseStorage)).  So assuming the DataSource is named `shibboleth.JPAStorageService.DataSource` , this would be:
        
        ```
        <DataConnector xsi:type="st:SharedToken" xmlns:st="urn:mace:arcs.org.au:shibboleth:2.0:resolver:dc"
            id="sharedToken"
            databaseConnectionID="shibboleth.JPAStorageService.DataSource"
            ...
        />
        ```
        
*   Rebuild IdP WAR file (with new sharedToken module version) and start Tomcat
    
    ```
    /opt/shibboleth-idp/bin/build.sh
    service tomcat start
    ```
    
*   The updated version of the IdP should be running
*   To properly record the change, edit `/etc/profile.d/shib.sh` and update `IDP_VERSION` to the new IdP version.