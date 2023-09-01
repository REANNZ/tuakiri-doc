
Shibboleth SP has two separate components (the `shibd` daemon and the `mod_shib` module running inside Apache), and they also have separate logging configuration and destinations.

*   The `shibd` daemon logs primarily into `/var/log/shibboleth/shibd.log` (with transaction details in `/var/log/shibboleth/transaction.log`)  
    *   Logging configuration is in `/etc/shibboleth/shibd.logger`
    *   Log files should be owned by `shibd` (the user account `shibd` daemon runs under)
*   The `mod_shib` Apache module logs into syslog (as facility `LOCAL0` ).  
    *   Logging configuration in `/etc/shibboleth/native.logger`
    *   In Shibboleth SP 2.x, `mid_shib` was logging into `/var/log/shibboleth-www/native.log` and `/var/log/shibboleth-www/native-warn.log`Log (and these files were owned by `apache`, the user account Apache httpd runs under)

