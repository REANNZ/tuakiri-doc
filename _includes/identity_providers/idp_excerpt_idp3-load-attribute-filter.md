
*   Contact the [federation administrators](mailto:tuakiri@reannz.co.nz) (by emailing [tuakiri@reannz.co.nz](mailto:tuakiri@reannz.co.nz)) and request a URL for the Attribute Filter for your IdP.
    *   In the request, please include:
        *   The name (hostname or entityID) of your IdP
        *   An email address that should receive notifications whenever the attribute filter changes (these are notifications only, no action will be required).
    *   The attribute filter may have to be manually added to the list of attribute filters published. Once created, the URL will have the form of: `**[https://directory.tuakiri.ac.nz/attribute-filter/](http://directory.tuakiri.ac.nz/attribute-filter/)**``**<institution-domain>.xml**`

*   Edit `$IDP_HOME/conf/services.xml` and add the additional attribute filter as an additional resource in the `shibboleth.AttributeFilterResources` `util:list` bean, using the built-in FileBackedHTTPResource:
    
    ```
            <bean id="TuakiriAttributeFilterResource" class="net.shibboleth.ext.spring.resource.FileBackedHTTPResource"
                  c:client-ref="shibboleth.MemoryCachingHttpClient" 
                  c:url="https://directory.tuakiri.ac.nz/attribute-filter/institution.domain.ac.nz.xml"
                  c:backingFile="%{idp.home}/conf/tuakiri-attribute-filter.xml"/>
    ```
    
*   For Tuakiri-TEST, the configuration would be the same, just the URL would be different - please use the URL provided by the [federation administrators](mailto:tuakiri@reannz.co.nz).

