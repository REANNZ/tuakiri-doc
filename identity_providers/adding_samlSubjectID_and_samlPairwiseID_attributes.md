# Adding samlSubjectID and samlPairwiseID attributes

The international R&E community has agreed on new attributes to identify users in SAML SSO: a general-purpose subject identifier, **samlSubjectID** and a pairwise (targeted) identifier, **samlPairwiseID** .

These attributes provide an alternative to the `eduPersonPrincipalName` and `eduPersonTargetedID` / SAML2 Persistent NameID attributes used within Tuakiri.  However, at the current stage, the new attributes will be provided alongside the existing attributes - and the process for services to transition to the new attributes will be over a longer period of time.

This document provides the instructions for Identity Providers to start supporting the attributes - and so far, this is where it stays.

/\*<!\[CDATA\[\*/ div.rbtoc1693282059487 {padding: 0px;} div.rbtoc1693282059487 ul {list-style: disc;margin-left: 0px;} div.rbtoc1693282059487 li {margin-left: 0px;padding-left: 0px;} /\*\]\]>\*/

*   1 [Background](#AddingsamlSubjectIDandsamlPairwiseIDattributes-Background)
*   2 [Rationale](#AddingsamlSubjectIDandsamlPairwiseIDattributes-Rationale)
*   3 [New samlSubjectID and samlPairwiseID Attributes](#AddingsamlSubjectIDandsamlPairwiseIDattributes-NewsamlSubjectIDandsamlPairwiseIDAttributes)
    *   3.1 [Syntax](#AddingsamlSubjectIDandsamlPairwiseIDattributes-Syntax)
    *   3.2 [Semantics](#AddingsamlSubjectIDandsamlPairwiseIDattributes-Semantics)
    *   3.3 [Release rules](#AddingsamlSubjectIDandsamlPairwiseIDattributes-Releaserules)
*   4 [Configuring an IdP to support samlSubjectID and samlPairwiseID](#AddingsamlSubjectIDandsamlPairwiseIDattributes-ConfiguringanIdPtosupportsamlSubjectIDandsamlPairwiseID)
    *   4.1 [Defining samlSubjectID](#AddingsamlSubjectIDandsamlPairwiseIDattributes-DefiningsamlSubjectID)
    *   4.2 [Defining samlPairwiseID](#AddingsamlSubjectIDandsamlPairwiseIDattributes-DefiningsamlPairwiseID)
    *   4.3 [Configuring Attribute Release](#AddingsamlSubjectIDandsamlPairwiseIDattributes-ConfiguringAttributeRelease)
    *   4.4 [Applying changes](#AddingsamlSubjectIDandsamlPairwiseIDattributes-Applyingchanges)
*   5 [Testing](#AddingsamlSubjectIDandsamlPairwiseIDattributes-Testing)
*   6 [Documentation](#AddingsamlSubjectIDandsamlPairwiseIDattributes-Documentation)
*   7 [References](#AddingsamlSubjectIDandsamlPairwiseIDattributes-References)

# Background

The global R&E community has gone through several iterations of attributes used to identify users.  Generally, there is need for two kinds of attribute, a targeted one (same user gets different value for different services) and general-use one (same user gets the same value for all services).

When Tuakiri started (~2010), the consensus within the global community was was to use eduPersonPrincipalName as the general-use attribute, and eduPersonTargetedID as the targeted attribute.

When Tuakiri was driving the upgrade to IdP 3.x, eduPersonTargetedID was already getting deprecated, and so the recommendation in the IdP 3.x upgrade was to switch to using SAML2 Persistent NameID as the targeted attribute - which at that point was, at least within some parts of the R&E community, considered the best practice.

However, using SAML2 Persistent NameID did not get much traction globally, and some [recommendations](https://kantarainitiative.github.io/SAMLprofiles/saml2int.html#_subject_identifiers_2) now mark it as to be avoided.

The international R&E community finally settled on the [SAML V2.0 Subject Identifier Attributes Profile Version 1.0](https://docs.oasis-open.org/security/saml-subject-id-attr/v1.0/cs01/saml-subject-id-attr-v1.0-cs01.html), defining new attributes **samlSubjectID**  (general use) and **samlPairwiseID** (targeted).

In the long term, this is how service providers should be identifying users - the first step is to support these on each IdP.

# Rationale

It may be worth explaining the rationale for the change.  The attributes currently in use have served us reasonably well, but suffer from the following issues:

*   **eduPersonPrincipalName** was NOT defined to be non-reassignable.  Even though when deploying it within Tuakiri, we were putting strong emphasis on not using source attributes that can potentially be re-assigned to different user (such as username `jsmith` ), this was not part of the original definition - and globally, there are deployments that have been using source attributes with reassigned values.  Consequently, the global community lost trust in this attribute.  
    If the source attribute is not reassigned in your deployment, the samlSubjectID attribute can be defined with the same source attribute as eduPersonPrincipalName - but the non-reassignment is a critical property.
*   **eduPersonTargetedID** and **SAML2 Persistent NameID** have a complex structued (XML) value that is hard to manipulate.  While Shibboleth SP implements a simple way of serializing into a single string value (using "`!`" as the separator), other SP implementations have been using shortcuts that used only the hash value, but not other scoping elements, increasing risk of collisions (both accidental and intentional/malicious). Also, due to the custom encoding used here, as the attribute is not a _scoped_ attribute (like `eduPersonPrincipalName`), it does not get the protection of  assigned namespace scope that would be enforced for scoped attributes.  
    Further, these attributes are case sensitive, but some applications interpret them as case insensitive, further increasing the risk of collisions.
*   **SAML2 Persistent NameID** also makes it technically challenging to make Service Providers request the right NameID Format.  (If a NameID Format is explicitly requested by the SP, but cannot be provided by an IdP, the IdP fails the login.  If left unspecified, the IdP chooses a default format, which may be Persistent NameID if supported by the SP -but this makes the whole setup unreliable, building upon implementation details outside of SAML specifications).  
    And also makes it tricky for applications to receive the value.  While Shibboleth SP receives both variants (**eduPersonTargetedID** and **SAML2 Persistent NameID**) into the same attribute `persistent-id` , other SAML SP implementations do not have this straightforward - and applications often needs to be customised to receive the userID attribute from the SAML2 NameID.

# New samlSubjectID and samlPairwiseID Attributes

The samlSubjectID and samlPairwiseID attributes are defined in the [SAML V2.0 Subject Identifier Attributes Profile Version 1.0](https://docs.oasis-open.org/security/saml-subject-id-attr/v1.0/cs01/saml-subject-id-attr-v1.0-cs01.html).

These two attributes have many properties in common - they are:

*   expressed as **scoped** attributes, preventing collisions (accidental or malicious) from different IdPs.
*   case insensitive, avoiding collisions due to case ignored in comparison
*   expressed using a limited character set (alphanumeric plus hyphen `'-'` and equals `'='` ).
*   limited in length for database storage (255 characters)

The values can be either transparent or opaque - though, in reality, this only applies to samlSubjectID.  As samlTargetedID needs to provide targeted values for different service providers, these will have to be produced via a hashing function, and will thus be opaque.

## Syntax

Both samlSubjectID and samlPairwiseID follow the same syntax rules.  The specification should be taken as authoritative, but the format is `<uniqueID> "@" <scope>`, where:

*   The unique ID consists of 1 to 127 ASCII characters, each of which is either an alphanumeric ASCII character, an equals sign (ASCII 61), or a hyphen (ASCII 45). The first character MUST be alphanumeric.
*   The scope consists of 1 to 127 ASCII characters, each of which is either an alphanumeric ASCII character, a hyphen (ASCII 45), or a period (ASCII 46). The first character MUST be alphanumeric.

## Semantics

The key property of the attributes is that values are not reassigned - that the same value is not assigned to different subjects:

*   **A value MUST NOT be assigned to more than a single subject over its lifetime of use under any circumstances.**

And the values must be compared in a case-insensitive way:

*   **Value comparison MUST be performed case-insensitively (that is, values that differ only by case are the same, and MUST refer to the same subject).** 

Additionally, the samlPairwiseID attribute has to provide targeted values that cannot be mapped back to the user at the service provider:

*    **The value MUST NOT be mappable by a relying party into a non-pairwise identifier for the subject through ordinary effort.**

## Release rules

The specification for samlSubjectID and samlPairwiseID provides a new mechanism for an SP to signal it desires one of these identifiers: via an EntityAttribute (embedded in the SP metadata) of name `urn:oasis:names:tc:SAML:profiles:subject-id:req` with possible values being:

*   `subject-id` : request samlSubjectID
*   `pairwise-id` : request samlPairwiseID
*   `none` : none of these identifiers is requested (... this will be the default if not specified)
*   `any` : requesting any of these identifiers - but requiring at least one.  (Default Shibboleth IdP configuration interprets this as releasing just samlPairwiseID)

This mechanism is completely different from the `RequestedAttribute` entries included in the SP's SAML metadata for other attributes - and that mechanism should not be used for samlSubjectID / samlPairwiseID.

# Configuring an IdP to support samlSubjectID and samlPairwiseID

Requires 4.1.0+ for database table name support.

The attributes however have strict requirements around their syntax and semantics - before configuring the attributes, please carefully consider whether you'd be meeting these requirements.

**The key to getting these attributes right is having a source attribute that is not reassigned.**

## Defining samlSubjectID

As stated earlier, the key property of these new attributes is that the values are not reassigned - and they can only be configured properly if a suitable source attribute is available at the IdP.  If the source attribute also meets the requirement on syntax - specifically:

*   is a sequence of alphanumeric characters ('-' and '=' are also permitted but not as the first character)
*   has no case-folding collisions (there will not be distinct values that differ only in case)

then the source attribute (username) can be used directly and its definition would be mimicking how eduPersonPrincipalName is defined.

Attribute definition to put into `/opt/shibboleth-idp/conf/attribute-resolver.xml` (assuming the source attribute is already visible in the resolver as `sAMAccountName` ) could be: 

```
    <AttributeDefinition id="samlSubjectID" xsi:type="Scoped" scope="%{idp.scope}">
        <InputAttributeDefinition ref="sAMAccountName" />
        <DisplayName xml:lang="en">Unique ID</DisplayName>
        <AttributeEncoder xsi:type="SAML2ScopedString" name="urn:oasis:names:tc:SAML:attribute:subject-id" friendlyName="samlSubjectID" />
    </AttributeDefinition>
```

## Defining samlPairwiseID

Assuming the IdP is already configured with Persistent NameID (or eduPersonTargetedID) that uses BASE64 encoding and is stored in a database (typically table `shibpid` ), the samlPairwiseID attribute will need a separate database table (following the same table structure as `shibpid` ).

*   Create the database table with: 
    
    ```
    CREATE TABLE pairwise_id (
        localEntity VARCHAR(255) NOT NULL,
        peerEntity VARCHAR(255) NOT NULL,
        persistentId VARCHAR(50) NOT NULL,
        principalName VARCHAR(50) NOT NULL,
        localId VARCHAR(50) NOT NULL,
        peerProvidedId VARCHAR(50) NULL,
        creationDate TIMESTAMP NOT NULL,
        deactivationDate TIMESTAMP NULL,
        PRIMARY KEY (localEntity, peerEntity, persistentId)
    );
    ```
    
*   If the database account used by the IdP for connecting to the database uses fine-grained permissions (i.e., not `ALL PRIVILEGES ON idp_db.*` ), grant the account `SELECT,INSERT,DELETE` permissions on this table: 
    
    ```
    GRANT SELECT,INSERT,DELETE ON idp_db.pairwise_id TO 'idp_admin'@'localhost';
    ```
    
*   Create the DataConnector and Attribute definition by adding the following to `/opt/shibboleth-idp/conf/attribute-resolver.xml` (again assuming the source attribute is already visible in the resolver as `sAMAccountName` ): 
    
    ```
        <DataConnector id="PairwiseIDConnector" xsi:type="StoredId"
                generatedAttributeID="PairwiseID"
                salt="%{idp.persistentId.salt}"
                tableName="pairwise_id"
                encoding="BASE32">
            <InputAttributeDefinition ref="sAMAccountName" />
            <BeanManagedConnection>%{idp.persistentId.dataSource}</BeanManagedConnection>
        </DataConnector>
    
        <AttributeDefinition id="samlPairwiseID" xsi:type="Scoped" scope="%{idp.scope}">
            <InputDataConnector ref="PairwiseIDConnector" attributeNames="PairwiseID" />
            <DisplayName xml:lang="en">Pairwise ID</DisplayName>
            <AttributeEncoder xsi:type="SAML2ScopedString" name="urn:oasis:names:tc:SAML:attribute:pairwise-id" friendlyName="samlPairwiseID" encodeType="false" />
        </AttributeDefinition>
    ```
    

## Configuring Attribute Release

New IdP installs automatically include rules to release samlSubjectID / samlPairwiseID as requested by the EntityAttribute (see Release Rules above) in `/opt/shibboleth-idp/conf/attribute-filter.xml`.  These rules were however only added in IdP 3.4.0.  IdP deployments where attribute-filter.xml is based on an older version may have to explicit add the rule into attribute-filter.xml.  The rule ( `subject-identifiers` `AttributeFilterPolicy` ) can be found in `/opt/shibboleth-idp/dist/conf/attribute-filter.xml` (the copy of the file included with the most recently installed version).

## Applying changes

While some files (IdP services) may be configured for automatic reload (like `attribute-resolver.xml` ), this is not configured on all IdPs, and even reloading the Attribute Resolver service will not update the part of the configuration from this file that is stored in the Attribute Registry Service.  For the above changes to take affect, either restart the IdP, or reload the Attribute Resolver, Attribute Registry and Attribute Filter services with: 

```
wget --no-check-certificate --quiet -O - 'https://localhost/idp/profile/admin/reload-service?id=shibboleth.AttributeFilterService'
wget --no-check-certificate --quiet -O - 'https://localhost/idp/profile/admin/reload-service?id=shibboleth.AttributeResolverService'
wget --no-check-certificate --quiet -O - 'https://localhost/idp/profile/admin/reload-service?id=shibboleth.AttributeRegistryService'
```

Note: the reload commands connect to the IdP via `localhost` to meet the access control rules - but as the certificate is not valid for the `localhost` hostname, the `--no-check-certificate` parameter is required (and legitimate in this case).

# Testing

The Tuakiri Attribute Validator has been already configured to request these attributes (as a special case, to use with testing, it requests BOTH of these attributes, even a typical SP deployment would request at most one).

Log into the appropriate instance of the Attribute Validator to confirm the attributes are being released by your IdP:

*   Tuakiri (Production): [https://attributes.tuakiri.ac.nz/](https://attributes.tuakiri.ac.nz/)
*   Tuakiri-TEST: [https://attributes.test.tuakiri.ac.nz/](https://attributes.test.tuakiri.ac.nz/)

# Documentation

This guide assumes the IdP was installed as per the [Tuakiri IdP Installation Manual](https://reannz.atlassian.net/wiki/spaces/Tuakiri/pages/3815538813/Installing+a+Shibboleth+3.x+IdP).

The following links into the upstream [Shibboleth IdP documentation](https://wiki.shibboleth.net/confluence/display/IDP4/Configuration) may be helpful:

*   [StoredIdConnector](https://wiki.shibboleth.net/confluence/display/IDP4/StoredIdConnector)
*   [AttributeResolverConfiguration](https://wiki.shibboleth.net/confluence/display/IDP4/AttributeResolverConfiguration)
*   [PersistentNameIDGenerationConfiguration](https://wiki.shibboleth.net/confluence/display/IDP4/PersistentNameIDGenerationConfiguration)

# References

\[samlSubjectID\]: SAML V2.0 Subject Identifier Attributes Profile Version 1.0: [https://docs.oasis-open.org/security/saml-subject-id-attr/v1.0/cs01/saml-subject-id-attr-v1.0-cs01.html](https://docs.oasis-open.org/security/saml-subject-id-attr/v1.0/cs01/saml-subject-id-attr-v1.0-cs01.html)

\[SAML2Int\]: SAML V2.0 Deployment Profile for Federation Interoperability: [https://kantarainitiative.github.io/SAMLprofiles/saml2int.html](https://kantarainitiative.github.io/SAMLprofiles/saml2int.html) (in particular sections [4.1.3 IdP Subject Identifiers](https://kantarainitiative.github.io/SAMLprofiles/saml2int.html#_subject_identifiers_2) and [3.1.3. Subject Identification](https://kantarainitiative.github.io/SAMLprofiles/saml2int.html#_subject_identification))

\[Tuakiri-samlSubjectID\]: Tuakiri samlSubjectID overview: [https://attributes.tuakiri.ac.nz/documentation/attributes/urn:oasis:names:tc:SAML:attribute:subject-id](https://attributes.tuakiri.ac.nz/documentation/attributes/urn:oasis:names:tc:SAML:attribute:subject-id)

\[Tuakiri-samlPairwiseID\]: Tuakiri samlPairwiseID overview: [https://attributes.tuakiri.ac.nz/documentation/attributes/urn:oasis:names:tc:SAML:attribute:pairwise-id](https://attributes.tuakiri.ac.nz/documentation/attributes/urn:oasis:names:tc:SAML:attribute:pairwise-id)

\[IdPv4\]: IdPv4 documentation: [https://wiki.shibboleth.net/confluence/display/IDP4/Configuration](https://wiki.shibboleth.net/confluence/display/IDP4/Configuration)