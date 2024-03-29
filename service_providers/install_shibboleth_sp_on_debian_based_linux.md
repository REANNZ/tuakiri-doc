---
redirect_from: Install+Shibboleth+SP+on+Debian+Based+linux
id: service_providers/install_shibboleth_sp_on_debian_based_linux
---
# Install Shibboleth SP on Debian Based linux
{:.no_toc}

# Introduction

{% include service_providers/shibsp-excerpt-intro-part1.md %}

This documentation is written based on and tested on Ubuntu 20.04 Server x86\_64, but should work on other Debian-based distributions as well.

{% include service_providers/shibsp-excerpt-intro-part2.md %}

Please note that SP 3.x has only been released for Ubuntu 20.04 and above and has not been released for older versions.  On hosts running older versions of Ubuntu, following this manual will install Shibboleth 2.6.x.

1. TOC
{:toc}

# Prerequsites

## Firewall settings

{% include service_providers/shibsp-excerpt-prereq-firewall.md %}

## Dependencies

Before starting to build and configure the Shibboleth Service Provider, be sure that the Apache2 package is installed, and required modules (socache\_shmcb and ssl) are enabled:

```
sudo apt-get install -y apache2
sudo a2enmod ssl
sudo a2ensite default-ssl
```

## Time synchronization

{% include service_providers/shibsp-excerpt-prereq-timesync.md %}

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

{% include federation_management/adding_an_sp_to_tuakiri-excerpt.md %}

## Configuration

{% include service_providers/shibsp-excerpt-configuration.md %}

> **Note**  
> Log file race condition
>
> With earlier versions of Shibboleth SP (2.x), it was necessary to work around issues with rotation of logs generated by the `mod_shib`  module running inside Apache.  In Shibboleth SP 3.x, this module logs via syslog and this is no longer an issue. 
>
> On hosts that already have SP 3.x (like Ubuntu 20.04), this part can be skipped.
>
> If deploying a 2.x installation (e.g., installing on a distribution that SP 3.x has not been released for yet, like Ubuntu 18.04), or explicitly logging to file, follow these instructions.
>
> <details markdown="1">
> <summary>Click here to expand...</summary>
>
> *   To work around issues with rotation with logs generated by the `mod_shib`  module running inside Apache, it is necessary to move the log rotation from the module to logrotate.
>     *   There is a race condition in the log rotation.  This has been reported upstream as [SSPCPP-757](https://issues.shibboleth.net/jira/browse/SSPCPP-757) - and we recommend to move log rotation out of `mod_shib` to `logrotate`.  
>           
>         
>     *   Edit `/etc/shibboleth/native.logger`  and:
>         *   replace `RollingFileAppender` with `FileAppender`
>         *   comment out log rotation-specific options: `maxFileSize` and `maxBackupIndex`
>         *   or just replace the file with our copy with exactly these customizations: [native.logger](https://github.com/REANNZ/Tuakiri-public/raw/master/shibboleth-sp/native.logger)
>     *   Install a new file into `/etc/logrotate.d/shibboleth-www` to rotate these files via `logrotate` (and reload Apache post-rotate): [shibboleth-www](https://github.com/REANNZ/Tuakiri-public/raw/master/shibboleth-sp/logrotate-debian/shibboleth-www) containing:
>         
>           
>         
>         ```
>         /var/log/shibboleth-www/*.log {
>             missingok
>             daily
>             rotate 10
>             nodateext
>             size 1000000
>             sharedscripts
>             postrotate
>                 /usr/sbin/service apache2 reload > /dev/null 2>/dev/null || true
>             endscript
>         }
>         ```
>         
>     *   These can be both installed with:
>         
>         ```
>         wget -O /etc/shibboleth/native.logger https://github.com/REANNZ/Tuakiri-public/raw/master/shibboleth-sp/native.logger
>         wget -O /etc/logrotate.d/shibboleth-www https://github.com/REANNZ/Tuakiri-public/raw/master/shibboleth-sp/logrotate-debian/shibboleth-www
>         ```
>         
> </details>
  

# Special considerations

## HTTP/HTTPS access

{% include service_providers/shibsp-excerpt-special-http-https.md %}

## ECP

{% include service_providers/shibsp-excerpt-special-ecp.md %}

# Logging

{% include service_providers/shibsp-excerpt-logging.md %}

# Protecting a Resource

{% include service_providers/shibsp-excerpt-protect-resource.md %}

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

{% include service_providers/shibsp-excerpt-testing.md %}
