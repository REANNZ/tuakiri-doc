
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
    
    <details markdown="1">
    <summary>Click here to expand...</summary>
    
    In earlier versions (Shibboleth SP 2.x), we were recommending to configure the TLS protocols and cipher-suites acceptable on the back-channel - the [default settings](https://wiki.shibboleth.net/confluence/display/SHIB2/NativeSPApplication#NativeSPApplication-RelyingPartyAttributes) were overly permissive and insecure.
    
    Shibboleth 3.x now sets a new default, identical to our recommendation in terms of actual ciphers permitted.  So, this step is no longer needed on Shibboleth SP 3.x
    
    On Shibboleth SP 2.x, add the following XML attribute to the `<ApplicationDefaults>` element:
    
    ```
    cipherSuites="DEFAULT:!EXP:!SSLv2:!DES:!IDEA:!SEED:!RC4:!3DES:!kRSA:!SSLv3:!TLSv1:!TLSv1.1"
    ```
    
    This sets the protocols to TLSv1.2 only (banning SSLv2, SSLv3, TLSv1.0, TLSv1.1) and blocks all ciphers deemed insecure (as of October 2017).

    </details>
    
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

