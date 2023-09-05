
*   Configure the IdP to load the Federation Metadata in `/opt/shibboleth-idp/conf/metadata-providers.xml` by adding the following snippet into the `Chaining` `MetadataProvider`.
    
    ```
        <MetadataProvider id="TuakiriMetadata"
                          xsi:type="FileBackedHTTPMetadataProvider"
                      refreshDelayFactor="0.125"
                      maxRefreshDelay="PT2H"
                      backingFile="%{idp.home}/metadata/tuakiri-metadata.xml"
                      metadataURL="https://directory.tuakiri.ac.nz/metadata/tuakiri-metadata-signed.xml">
    
                <MetadataFilter xsi:type="SignatureValidation"
                        certificateFile="${idp.home}/credentials/tuakiri-metadata-cert.pem"
                        requireSignedRoot="true">
                </MetadataFilter>
                <MetadataFilter xsi:type="EntityRoleWhiteList">
                        <RetainedRole>md:SPSSODescriptor</RetainedRole>
                </MetadataFilter>
    
        </MetadataProvider>
    ```
    
      
    
    *   Note: validity checking is implicitly turned on, so it is not needed to explicitly add the `RequiredValidUntil` metadata filter, which would only be useful to reject metadata published with a validity longer then maxValidityInterval milliseconds.  We recommend to rely on signature validation.  The Tuakiri metadata are being generated with a validity of one week.
    *   Note: by default, metadata get refreshed only every 3 hours (0.75 factor out of 4 hours maximum refresh interval).   
        *   To make metadata changes propagate faster (reload every 15 minutes), set the maximum refresh interval to 2 hours and the factor to 0.125 as above.
        *   To avoid re-fetching the file even when not changed, turn on caching (memory caching is enough as we already do have a backing file)
    *   See the IDP30 [https://wiki.shibboleth.net/confluence/display/IDP30/MetadataConfiguration](https://wiki.shibboleth.net/confluence/display/IDP30/MetadataConfiguration) and [https://wiki.shibboleth.net/confluence/display/IDP30/FileBackedHTTPMetadataProvider](https://wiki.shibboleth.net/confluence/display/IDP30/FileBackedHTTPMetadataProvider) documentation for more information.
    *   Note: in IdP 3.0.0, the `RetainedRole` element was incorrectly using the namespace `samlmd` - as of 3.1.1, the namespace declared in metadata-providers.xml and used in the examples is `md`, consistent with other use.

*   This definition is referring to a certificate used to verify the signature - store the certificate in `/opt/shibboleth-idp/credentials`
    
    ```
    wget https://directory.tuakiri.ac.nz/metadata/tuakiri-metadata-cert.pem -O $IDP_HOME/credentials/tuakiri-metadata-cert.pem
    ```
    
    > **Note**  
    > Tuakiri-TEST specific
    >
    > When building a TEST IdP and registering into Tuakiri-TEST instead, please load instead the Tuakiri-TEST metadata with:
    >
    > ```
    >     <MetadataProvider id="TuakiriTESTMetadata"
    >                       xsi:type="FileBackedHTTPMetadataProvider"
    >                   refreshDelayFactor="0.125"
    >                   maxRefreshDelay="PT2H"
    >                   backingFile="%{idp.home}/metadata/tuakiri-test-metadata.xml"
    >                   metadataURL="https://directory.test.tuakiri.ac.nz/metadata/tuakiri-test-metadata-signed.xml">
    >
    >             <MetadataFilter xsi:type="SignatureValidation"
    >                     certificateFile="${idp.home}/credentials/tuakiri-test-metadata-cert.pem"
    >                     requireSignedRoot="true">
    >             </MetadataFilter>
    >             <MetadataFilter xsi:type="EntityRoleWhiteList">
    >                     <RetainedRole>md:SPSSODescriptor</RetainedRole>
    >             </MetadataFilter>
    >
    >     </MetadataProvider>
    > ```
    >
    > and fetch the Tuakiri-TEST metadata signing certificate instead:
    >
    > ```
    > wget https://directory.test.tuakiri.ac.nz/metadata/tuakiri-test-metadata-cert.pem -O $IDP_HOME/credentials/tuakiri-test-metadata-cert.pem
    > ```

