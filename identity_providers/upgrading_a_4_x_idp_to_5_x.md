---
id: identity_providers/upgrading_a_4_x_idp_to_5_x
---
# Upgrading a 4.x IdP to 5.x
{:.no_toc}

IdP 5.0.0 was released in September 2023 and IdP 4.x will be declared End-Of-Life on September 1st, 2024.   All production IdPs thus have to be upgraded to 5.x.

The [upstream upgrade documentation](https://shibboleth.atlassian.net/wiki/spaces/IDP5/pages/3199500925/Upgrading) strongly recommends doing an in-place upgrade to 5.x - and given the upgrade is mostly smooth (while a fresh install would require significant changes to attribute configuration), we follow this recommendation and recommend Tuakiri members do the upgrade in-place.

Already in 3.x and 4.x, the process for in-place upgrades has been well tuned - and the upgrade to 5.x should be seamless as well, as long as prior to the 5.x upgrade, the IdP runs the latest 4.x (4.3.2 or 4.3.1) with no **Deprecation warnings**.

1. TOC
{:toc}

# Background

IdP 3.x introduced a simplified upgrade process for in-place upgrades - where essentially, just running the installer from the new version distribution directory is sufficient.  The installer keeps the existing local configuration (primarily the `conf` directory, but also e.g. `credentials`  and `metadata` ), while updating the system files.  This highlights the fact that any local configuration should be done in the `conf` directory only, not touching files under the `system` directory - which is actually removed in 5.x.  This also makes the upgrades very easy to run.

Any new configuration files that do not exist in the `conf` directory yet are copied there.  New versions of existing files are stored in the same location with `.idpnew-<version>` added to the file name (e.g., `conf/idp.properties.idpnew-511`).  (This convention replaces the earlier approach where the `dist/conf`  directory contained a pristine copy of each configuration file as of the release being upgraded to, and all configuration files from the previous version were kept in `/opt/shibboleth-idp/old-<timestamp>`.

However, an upgrade between major versions (such as 4.x to 5.x) is more involved and may require some manual changes and may require different versions of Java and of the web application container - see below for details.

## In-place upgrade vs. new install

The [upstream upgrade documentation](https://shibboleth.atlassian.net/wiki/spaces/IDP5/pages/3199500925/Upgrading) strongly recommends doing an in-place upgrade to 5.x.  For Tuakiri IdPs installed initially with 3.x, significant changes to attribute configuration would be required in a new install and we follow the upstream recommendation and recommend Tuakiri members do the upgrade in-place.

Most of the changes to new installs already came in IdP 4.x.  IdP upgraded from a 3.x install retains the "3.x style" and is compatible with 3.x-style configuration files.

A fresh 4.x or 5.x install would not activate the 3.x style and just copying 3.x-style configuration files (such as `attribute-resolver.xml` ) into a 5.x install will not work correctly.

The differences between 3.x and 4.x include:

*   4.x uses a new configuration subsystem, the Attribute Registry, to define and configure attributes.  The Attribute Registry includes encoders - which 3.x defines in attribute-resolver.xml.  A clash between 3.x/4.x configuration styles here might result into having attributes duplicated in outgoing SAML assertions.
*   4.x uses a new default for encryption of SAML Assertions (GCM), which may cause issues with some SPs (not being able to decrypt).  Rolling out the change of the encryption ciphersuite should be separate from the IdP upgrade.
*   4.x stores all secrets in a separate file, `credentials/secrets.properties` and other configuration files that need the secrets refer to the properties defined in this file.  (This includes LDAP connection credentials).

We will later also provide an installation manual for a clean 5.x install - but for now, we recommend proceeding with an upgrade from the existing installation.

## Key aspects of 5.x upgrade

As a new major release, 5.0.0 rolls out breaking changes that could not have been done in minor releases.

These include:
*   Dropping support for Java 11 and requiring Java 17.  As IdP 4.x (4.2.x+) supports Java 17, the Java upgrade can be done ahead of actual IdP upgrade.
    * Java 17 however drops support for the Nashorn scripting engine used to construct values of scripted attributes.
    * The Shibboleth IdP project provides an IdP plugin that provides this functionality (`net.shibboleth.idp.plugin.nashorn`), however, this plugin will have to be explicitly installed (see below).
*   Switching from Java Servlet API 4.0 to 5.0.  As these are mutually incompatible, each servlet container implementation supports either 4.0 or 5.0, but not both.  This means the servlet container will have to be upgraded at the same time as the IdP itself.  In case of Tomcat, this means upgrading from 8.5 or 9.0 to 10.1.
*   The system directory (`/opt/shibboleth-idp/system`) is no longer supported and all of its contents has been migrated to jars that are included in the IdP WAR file.  Before running the IdP installer to upgrade to 5.x, the system directory will have to be removed.
*   Because of the above removal of the `system` directory (and also other changes done in 5.x), the web.xml file (`/opt/shibboleth-idp/edit-webapp/WEB-INF/web.xml)`), if present,  will have to be updated - or removed.  Unless there is a need to have customisations applied to this file, the best approach is to remove this file altogether (if present).  If customisations are required, we still recommend making a new copy of `/opt/shibboleth-idp/dist/webapp/WEB-INF/web.xml` into `/opt/shibboleth-idp/edit-webapp/WEB-INF/web.xml` and reapplying the customisations there.

There are further changes required, but the above key points should provide sufficient basis for planning the upgrade.

## Upgrade planning

While the above recommendation is to perform an in-place upgrade of the IdP application, this may not be directly acceptable because:
* this would cause an outage on a production system
* the operating system of the host is due for a refresh (migrating to next major version of the OS)

While the exact upgrade sequence will depend on specifics of the local deployment, our recommendation in case the OS refresh is needed is to:
* create a new VM with preferred OS
* install dependencies / pre-requisites of the IdP application (incl. Tomcat as per below)
* copy over `/opt/shibboleth-idp` (and local database if used) to the new system
* upgrade the IdP to 5.x.
* confirm new IdP operates correctly
* cut-over to replace original IdP (at either IP or DNS level)

If no OS upgrade is needed, we recommend making a clone of the existing VM instead of creating a new VM and proceeding with the above sequence.

# Preparing for upgrade

## Java

While Shibboleth IdP 4.x was primarily targetting Java 11, Shibboleth IdP 5.x requires Java 17.

We recommend OpenJDK 17, on RHEL-like systems available as: `java-17-openjdk-devel`.

Latest IdP 4.x also supports Java 17, so it is possible to upgrade Java first in an isolated step, reducing the amount of change in the actual IdP 5.x upgrade.

Note that as Java 17 drops support for the Nashorn scripting engine, if using scripted attribute definitions in `attribute-resolver.xml` (very likely yes, e.g. for `eduPersonAffiliation`), it is necessary to install the [`net.shibboleth.idp.plugin.nashorn` IdP plugin](https://shibboleth.atlassian.net/wiki/spaces/IDPPLUGINS/pages/1374027996/Nashorn) that provides a standalone deployment of the Nashorn scripting engine.

Note also that if your deployment is using a workaround to suppress the deprecation warning that Java 11 Nashorn emits on startup (passing `-Dnashorn.args=--no-deprecation-warning` to Java), this workaround would prevent startup of the Nashorn engine introduced by the IdP plugin.

Therefore, the recommended sequence of steps is:
1. Remove the workaround suppressing the Nashorn deprecation warning (`-Dnashorn.args=--no-deprecation-warning`) if present (e.g., from `JAVA_OPTS` in `/etc/sysconfig/tomcat`)
2. Install the Nashorn plugin:
   
   ```
   cd /opt/shibboleth-idp
   ./bin/plugin.sh -I net.shibboleth.idp.plugin.nashorn
   ```
   
3. Upgrade to Java 17
   * Install `java-17-openjdk-devel`
   * Make web application container use this Java version (selected e.g. by `JAVA_HOME` in `/etc/sysconfig/tomcat`)
   * Restart web application container (`service tomcat restart`)
   * Confirm the IdP initialises correctly (`/opt/shibboleth-idp/logs/idp-process.log`, web application container log - e.g. `catalina.out`)
   * Confirm the IdP releases correct values for the scripted attributes.

## Shibboleth IdP versions

The upstream documentation states very clearly that 5.x upgrades are supported only from latest 4.x, running with no deprecation warnings (for both startup and regular operation).

So first, [upgrade to the latest 4.x](upgrading_a_shibboleth_3_x_idp) (4.3.2 as of April 2024, but 4.3.1 will be sufficient - 4.3.2 did not introduce any new deprecation warnings).

After the upgrade, watch the logs while restarting the IdP and logging into the Attribute Validator ( [https://attributes.tuakiri.ac.nz/](https://attributes.tuakiri.ac.nz/) or [https://attributes.test.tuakiri.ac.nz/](https://attributes.test.tuakiri.ac.nz/) )

Please contact us if you need help removing any other deprecation warnings.

Proceed further only after the IdP restart + Attribute Validator cycle comes through clean with no deprecation warnings (checking both the IdP log in `/opt/shibboleth-idp/logs/idp-process.log` and the servlet container log, likely either in `/var/log/tomcat9/catalina.out` or `/opt/tomcat/current/logs/catalina.out`).

Please note that one specific deprecation is the removal of JPAStorageService, replacing it with JDBCStorageService.  Please see the [relevant section in the 3.x-to-4.x upgrade instructions](upgrading_a_3_x_idp_to_4_x#migrating-from-jpastorageservice-to-jdbcstorageservice).

## Fix metadata configuration

Earlier versions of the [IdP Install Manual](installing_a_shibboleth_3_x_idp) used incorrect syntax in the configuration snippet for loading Tuakiri (and eduGAIN) metadata.  While the incorrect syntax worked with IdP 3.x and 4.x, it does not work with IdP 5.x and needs to be corrected before the upgrade.

In `/opt/shibboleth-idp/conf/metadata-providers.xml`, the `certificateFile` attribute of the `SignatureValidation` `MetadataFilter` incorrectly used the `$` character in Spring property reference, while the correct syntax is using the `%` character.

The change in our documentation was from `certificateFile="${idp.home}/credentials/tuakiri-metadata-cert.pem"` to `certificateFile="%{idp.home}/credentials/tuakiri-metadata-cert.pem"`.

Please check all occurrences of `certificateFile` and if they use `${idp.home}`, replace it with `%{idp.home}`.

# Upgrading to IdP 5.x

## Application Container

The Shibboleth IdP web application runs in an application container, typically Tomcat or Jetty.  IdP 5.x requires a newer version of the application container - Tomcat 8.5 and 9.0 which was frequently used for IdP 4.x deployments will not work with IdP 5.x; Tomcat 10.1 is required.  For Jetty, the required version is 11.

Unfortunately, even on RHEL 9, only Tomcat 9 is available in the base OS repository.  Neither do any community-run repositories provide RPM packages for up-to-date releases of neither Tomcat nor Jetty.

There are many blog posts providing instructions for manual one-off installs.   We provide a set of instructions here as convenience - with the aim to make future updates to newer Tomcat version easier (using separate per-version directories for Tomcat binary distribution, but a shared directory for application context-descriptor files).  Feel free to use these - but also to install Tomcat or Jetty via other means.

<details markdown="1">
<summary>Click here to expand instructions to install newer version of Tomcat on RedHat-based systems.</summary>

{% include infra_tasks/tomcat10_1.md %}
</details>

> **Warning**  
> Only proceed further after upgrading the application container to a compatible version - Tomcat 10.1 (or later) or Jetty 11 (or later).

  

## JSTL API module

The [IdP Install Manual](installing_a_shibboleth_3_x_idp) instructs to install the JSTL API module - which is needed for rendering the IdP status page.  As Tomcat 10.1 switches to Servlet API 5.0, it also needs a newer version of the JSTL API module, 3.0.0.   The Tomcat 10.1 installation instructions above include installing the correct JSTL API module.  Note that in this version, only the API module has to be installed (and the second jar file is no longer needed).

## SharedToken module

The IdP plugin module that generates the auEduPersonSharedToken attribute also has to be upgraded - the version released for IdP 4.x is not compatible with IdP 5.x.  A new version, 2.1.x (2.1.0 as of April 2024), has been released for IdP 5.x.

However, this new version, 2.1.0, is only compatible with 5.x but not 4.x - so the module will have to be upgraded as part of the IdP upgrade and not earlier.

The new version can be downloaded from [https://github.com/REANNZ/arcs-shibext/releases/download/2.1.0/arcs-shibext-2.1.0.jar](https://github.com/REANNZ/arcs-shibext/releases/download/2.1.0/arcs-shibext-2.1.0.jar)

Please see the next section for detailed instructions.

## Upgrading IdP itself

*   Get the new version:
    
    ```
    NEW_IDP_VERSION=5.1.1
    cd /root/inst
    wget http://shibboleth.net/downloads/identity-provider/${NEW_IDP_VERSION}/shibboleth-identity-provider-${NEW_IDP_VERSION}.tar.gz
    tar xzf shibboleth-identity-provider-${NEW_IDP_VERSION}.tar.gz
    cd shibboleth-identity-provider-${NEW_IDP_VERSION}
    ```
    
*   Remove IdP `system` directory - and also the web.xml file (see above for options):
    
    ```
    cd /opt/shibboleth-idp
    rm -rf system/ edit-webapp/WEB-INF/web.xml
    ```
    
*   Run the installer (with the correct JDK) - and also fix permissions right after running the installer:
    
    ```
    JAVA_HOME=/usr/lib/jvm/java-17-openjdk ./bin/install.sh
    chown -R tomcat:tomcat /opt/shibboleth-idp/
    # and for SELinux:
    restorecon -R /opt/shibboleth-idp
    ```
    
*   Update IdP plugins (likely Nashorn and JDBCStorageService)
    
    ```
    # Get list of installed plugins and their status
    JAVA_HOME=/usr/lib/jvm/java-17-openjdk ./bin/plugin.sh -l
    # Update each plugin - e.g.:
    JAVA_HOME=/usr/lib/jvm/java-17-openjdk ./bin/plugin.sh --update net.shibboleth.idp.plugin.nashorn
    JAVA_HOME=/usr/lib/jvm/java-17-openjdk ./bin/plugin.sh --update net.shibboleth.plugin.storage.jdbc
    ```
    
*   Upgrade SharedToken module:
    *   Download new version:
        
        ```
        wget -P /opt/shibboleth-idp/edit-webapp/WEB-INF/lib https://github.com/REANNZ/arcs-shibext/releases/download/2.1.0/arcs-shibext-2.1.0.jar
        ```
        
    *   Remove old version(s):
        
        ```
        rm /opt/shibboleth-idp/edit-webapp/WEB-INF/lib/arcs-shibext-2.0.*.jar
        ```
        
*   Rebuild IdP WAR file (with new sharedToken module version) and start Tomcat
    
    ```
    /opt/shibboleth-idp/bin/build.sh
    service tomcat start
    ```
    
*   The updated version of the IdP should be running
*   To properly record the change, edit `/etc/profile.d/shib.sh` and update `IDP_VERSION` to the new IdP version.

# Post-upgrade changes

## New configuration options

* The IdP now detects if a new version is available: on start up, it contacts a central repository to get list of IdP versions and their statuses - and logs the result. If the local version is no longer current and an update is recommended, this is logged as a WARNING.  If this behaviour (IdP "calling home") is not desired, it can be disabled with the `idp.updateCheck.enable` - to disable the check add to `/opt/shibboleth-idp/conf/idp.properties`:
    
    ```
    idp.updateCheck.enable = false
    ```

* IdP configuration is driven by property files.
  * At startup, the IdP first loads `/opt/shibboleth-idp/conf/idp.properties`, and from there (in 3.x-style deployment) loads additional property files listed in property `idp.additionalProperties`.
  * This was changed in 4.x to searching for all `.properties` files under `/opt/shibboleth-idp/conf` - and we recommend switching to this style.
  * New deployments created under 4.x have one property file stored separately, `/opt/shibboleth-idp/credentials/secrets.properties` (holding sensitive values).
  * To switch to the new way of loading property files, make the following changes to `/opt/shibboleth-idp/conf/idp.properties`::
    * Enable loading all property files under `/opt/shibboleth-idp/conf` by adding:
      ```
      idp.searchForProperties=true
      ```
    * Comment out existing `idp.additionalProperties` property.
    * Only if you have a `/opt/shibboleth-idp/credentials/secrets.properties` properties file (unlikely on an install originating from 3.x): add `idp.additionalProperties` pointing to this file only:
      ```
      idp.additionalProperties=/credentials/secrets.properties
      ```

## JDBCStorageService

Earlier recommendations for configuring the JDBCStorageService (and its predecessor, the JPAStorageService) might have used the Tomcat JDBC Pooling Driver, `org.apache.tomcat.jdbc.pool.DataSource`.

As of October 2024, this driver is no longer supported by the JDBCStorageService plugin (as per the [plugin documentation](https://shibboleth.atlassian.net/wiki/spaces/IDPPLUGINS/pages/2989096970/JDBCStorageService)).

We recommend switching to the Apache Commons DBCP2 driver, `org.apache.commons.dbcp2.BasicDataSource`, which is included in the IdP 5.x distribution.

Edit `/opt/shibboleth-idp/conf/global.xml` and in the `shibboleth.JPAStorageService.DataSource` bean definition, change the driver class name from `org.apache.tomcat.jdbc.pool.DataSource` to `org.apache.commons.dbcp2.BasicDataSource`.

The basic configuration parameters (bean properties) of the two drivers are the same, though for specialised configuration, please consult the documentation for the [Tomcat JDBC Pooling Driver](https://tomcat.apache.org/tomcat-10.0-doc/jdbc-pool.html) and [Apache Commons DBCP2 Driver](https://commons.apache.org/proper/commons-dbcp/configuration.html).

## Dealing with new deprecations

IdP 5.x deprecates some legacy config and after upgrading to 5.x, there may be new deprecation messages - for features that will be removed in 6.x.

To make future upgrades easier and to reduce pollution of the logs, we recommend dealing with these now.

The following covers commonly encountered deprecation messages (on an IdP upgraded from an initial 3.x install).

* Duo: Shibboleth IdP stopped supporting "legacy" Duo - however, the configuration files, installed by default in IdP 3.x, are still present - and trigger a warning:
  ```
  WARN [DEPRECATED:113] - property 'idp.authn.Duo.supportedPrincipals' is no longer supported
  ```
  To remove the legacy Duo configuration and resolve this warning:
  * Comment out duo-related properties (namely `idp.authn.Duo.supportedPrincipals`) in `/opt/shibboleth-idp/conf/authn/authn.properties`.
    Note this is a multi-line property and all parts of it need to be commented out:
    ```
    #idp.authn.Duo.supportedPrincipals = \
    #    saml2/http://example.org/ac/classes/mfa, \
    #    saml1/http://example.org/ac/classes/mfa
    ```
  * Remove legacy Duo configuration files (note: it is important to first complete the above changes to how property files are loaded - in case `authn/duo.properties` is listed in `idp.additionalProperties`):
    ```
    rm /opt/shibboleth-idp/conf/authn/duo-authn-config.xml /opt/shibboleth-idp/conf/authn/duo.properties{,.idpnew*}
    ```
  
* Liberty profile, generating warning:
  ```
  WARN [DEPRECATED:123] - Spring bean 'Liberty.SSOS or Liberty.SSOS.MDDriven', (relying-party.xml): This will be removed in the next major version of this software; replacement is (none)
  ```
  This is an unused SSO profile.  Edit `/opt/shibboleth-idp/conf/relying-party.xml` and remove all beans (profile instances) named `Liberty.SSOS` or `Liberty.SSOS.MDDriven`

* HttpClient configuration.  The IdP has changed how it creates HttpClient instances (used for fetching data from remote locations).
  An IdP with 3.x config may produce a warning:
  ```
  WARN [DEPRECATED:113] - property 'idp.httpclient.filecaching.cacheDirectory' is no longer supported
  ```
  Just remove the `idp.httpclient.filecaching.cacheDirectory` property from `/opt/shibboleth-idp/conf/services.properties`.  It is unused, just sits there inherited from the 3.x generated config.

* Changes to how X509 credentials are loaded in the IdP: the IdP may log (multiple times):
  ```
  WARN [DEPRECATED:130] - Java class 'net.shibboleth.idp.profile.spring.factory.BasicX509CredentialFactoryBean': This will be removed in the next major version of this software; replacement is Parent bean 'shibboleth.BasicX509CredentialFactoryBean'
  ```
  Edit `/opt/shibboleth-idp/conf/credentials.xml` and replace all (two used and one commented out) occurrences of `class="net.shibboleth.idp.profile.spring.factory.BasicX509CredentialFactoryBean"` with `parent="shibboleth.BasicX509CredentialFactoryBean"`
  (In the Spring bean configuration, instead of specifying the class name directly, use a parent bean - that sets the correct class name internally).

* Changes to login and logout templates: the IdP may generate the following warning - only after a user visits either the login or logout page:
  ```
  WARN [DEPRECATED:130] - Java class 'net.shibboleth.idp.profile.context.RelyingPartyContext': This will be removed in the next major version of this software; replacement is net.shibboleth.profile.context.RelyingPartyContext
  ```
  The templates for login and logout pages (`/opt/shibboleth-idp/views/login.vm`, `/opt/shibboleth-idp/views/logout.vm`) inherited from 3.x config refer to the RelyingPartyContext under a legacy name.
  Edit `/opt/shibboleth-idp/views/login.vm` and `/opt/shibboleth-idp/views/logout.vm` and in both, change
  ```
  #set ($rpContext = $profileRequestContext.getSubcontext("net.shibboleth.idp.profile.context.RelyingPartyContext"))
  ```
  to
  ```
  #set ($rpContext = $profileRequestContext.getSubcontext("net.shibboleth.profile.context.RelyingPartyContext"))
  ```
