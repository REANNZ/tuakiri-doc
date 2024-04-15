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

Any new configuration files that do not exist in the `conf` directory yet are copied there.  New versions of existing files are stored in the same location with `.idpnew-<version>` added to the file name (e.g., `conf/idp.properties.idpnew-511`).  (This convention replaces the earlier approach where `dist/conf`  directory contained a pristine copy of each configuration file as of the release being upgraded to, and all configuration files from the previous version were kept in `/opt/shibboleth-idp/old-<timestamp>`.

However, an upgrade between major version (such as 4.x to 5.x) is more involved and may require some manual changes and may require different versions of Java and of the web application container - see below for details.

## In-place upgrade vs. new install

The [upstream upgrade documentation](https://shibboleth.atlassian.net/wiki/spaces/IDP5/pages/3199500925/Upgrading) strongly recommends doing an in-place upgrade to 5.x.  For Tuakiri IdPs installed initially with 3.x, significant changes to attribute configuration would be required in a new install and we follow the upstream recommendation and recommend Tuakiri members do the upgrade in-place.

Most of the changes to new installs already came in IdP 4.x.  IdP upgraded from a 3.x install retains the "3.x style" and is compatible with 3.x-style configuration files.

A fresh 4.x or 5.x install would not activate the 3.x style and just copying 3.x-style configuration files (such as `attribute-resolver.xml` ) into a 5.x install will not work correctly.

The differences between 3.x and 4.x include:

*   4.x uses a new configuration subsystem, the Attribute Registry, to define and configure attributes.  The Attribute Registry includes encoders - which 3.x defines in attribute-resolver.xml.  A clash between 3.x/4.x configuration styles here might result into having attributes duplicated in outgoing SAML assertions.
*   4.x uses a new default for encryption of SAML Assertions (GCM), which may cause issues with some SPs (not being able to decrypt).  Rolling out the change of encryption ciphersuite should be separate from the IdP upgrade.
*   4.x stores all secrets in a separate file, `credentials/secrets.properties` and other configuration files that need the secrets refer to the properties defined in this file.  (This includes LDAP connection credentials).

We will later provide also an installation manual for a clean 5.x install - but for now, we recommend proceeding with an upgrade from the existing installation.

## Key aspects of 5.x upgrade

As a new major release, 5.0.0 rolls out breaking changes that could not have been done in minor releases.

These include:
*   Dropping support for Java 11 and requiring Java 17.  As IdP 4.x (4.2.x+) supports Java 17, the Java upgrade can be done ahead of actual IdP upgrade.
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

We recommend OpenJDK 17, on RHEL-like systems available as: `java-17-openjdk-devel` .

Latest IdP 4.x also supports Java 17, so it is possible to upgrade Java first in an isolated step, reducing the amount of change in the actual IdP 5.x upgrade.

## Shibboleth IdP versions

The upstream documentation states very clearly that 5.x upgrades are supported only from latest 4.x, running with no Deprecation warnings (for both startup and regular operation).

So first, [upgrade to the latest 4.x](upgrading_a_shibboleth_3_x_idp) (4.3.2 as of April 2024, but 4.3.1 will be sufficient - 4.3.2 did not introduce any new deprecation warnings).

After the upgrade, watch the logs while restarting the IdP and logging into the Attribute Validator ( [https://attributes.tuakiri.ac.nz/](https://attributes.tuakiri.ac.nz/) or [https://attributes.test.tuakiri.ac.nz/](https://attributes.test.tuakiri.ac.nz/) )

Please contact us if you need help removing any other Deprecation warnings.

Proceed further only after the IdP restart + Attribute Validator cycle comes through clean with no Deprecation warnings (checking both the IdP log in `/opt/shibboleth-idp/logs/idp-process.log` and the servlet container log, likely either in `/var/log/tomcat9/catalina.out` or `/opt/tomcat/current/logs/catalina.out`).

Please note that one specific deprecation is the removal of JPAStorageService, replacing it with JDBCStorageService.  Please see the [relevant section in the 3.x-to-4.x upgrade instructions](upgrading_a_4_x_idp_to_5_x#migrating-from-jpastorageservice-to-jdbcstorageservice).

## Fix metadata configuration

Earlier versions of the [IdP Install Manual](installing_a_shibboleth_3_x_idp) used incorrect syntax in the configuration snippet for loading Tuakiri (and eduGAIN) metadata.  While the incorrect syntax worked with IdP 3.x and 4.x, it does not work with IdP 5.x and needs to be corrected before the upgrade.

In `/opt/shibboleth-idp/conf/metadata-providers.xml`, the `certificateFile` attribute of the `SignatureValidation` `MetadataFilter` incorrectly used the `$` character in Spring property reference, while the correct syntax is using the `%` character.

The change in our documentation was from `certificateFile="${idp.home}/credentials/tuakiri-metadata-cert.pem"` to `certificateFile="%{idp.home}/credentials/tuakiri-metadata-cert.pem"`.

Please check all occurrences of `certificateFile` and if they use `${idp.home}`, replace it with `%{idp.home}`.

# Upgrading to IdP 5.x

## Application Container

The Shibboleth IdP web application runs in an Application Container, typically Tomcat or Jetty.  IdP 5.x requires a newer version of the application container - Tomcat 8.5 and 9.0 which was frequently used for IdP 4.x deployments will not work with IdP 5.x; Tomcat 10.1 is required.  For Jetty, the required version is 11.

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
    chown -R tomcat.tomcat /opt/shibboleth-idp/
    # and for SELinux:
    restorecon -R /opt/shibboleth-idp
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
