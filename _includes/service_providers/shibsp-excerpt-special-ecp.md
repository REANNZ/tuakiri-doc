
If your SP should support [ECP]({{ 'ecp' | relative_url }}) (access via non-browser clients), then also:

1.  Edit the `<SSO>` element in `/etc/shibboleth/shibboleth2.xml` and add an `ECP="true"`attribute:
    
    ```
    <SSO ECP="true" ....>
    ```
    
2.  Add support for ECP in the metadata registered in the federation (as instructed above).

