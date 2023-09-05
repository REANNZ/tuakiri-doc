
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

