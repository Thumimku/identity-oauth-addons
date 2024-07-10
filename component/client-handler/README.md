## <p style="color: red; font-weight: bold;">⚠️ Deprecation Notice </p>

**This component has been moved to a new standalone repository.All the future development will be
continued in the new repository and this repository will no longer be maintained.**

**You can find the new repository here: [identity-oauth-privatekey-jwthandler](https://github.com/wso2-extensions/identity-oauth-privatekey-jwthandler)**


<hr style="border: 2px solid yellow;" />

## 01. Private Key JWT Client Authentication 

Pre-requisites:

- Maven 3.x
- Java 1.7 or above

Tested Platform:

- Linux
- WSO2 IS 5.3.0
- Java 1.7

Do the following:

Deploying and Configuring JWT client-handler artifacts:
1. Navigate to client-handler/org.wso2.carbon.identity.oauth2.grant.jwt and build.

2. Place target/
org.wso2.carbon.identity.oauth2.token.handler.clientauth.jwt-1.0.0-SNAPSHOT.jar in the <IS_HOME>/repository/component/dropins directory.

3. To register the JWT grant type, configure the <IS_HOME>/repository/conf/identity/identity.xml file by adding a new entry under the <OAuth><ClientAuthHandlers> element. Add a unique <ClientAuthHandler> identifier between as seen in the code block below.

        <ClientAuthHandler Class="org.wso2.carbon.identity.oauth2.token.handler.clientauth.jwt.PrivateKeyJWTClientAuthHandler">
            <Property Name="RejectBeforePeriodInMinutes">60</Property>
        </ClientAuthHandler>
            
4. Update <GrantTypeValidatorImplClass> for supported grant types as below in <IS_HOME>/repository/conf/identity/identity.xml, under <SupportedGrantTypes> tag

        <SupportedGrantType>
            <GrantTypeName>authorization_code</GrantTypeName>
            <GrantTypeHandlerImplClass>org.wso2.carbon.identity.oauth2.token.handlers.grant.AuthorizationCodeGrantHandler</GrantTypeHandlerImplClass>
            <GrantTypeValidatorImplClass>org.wso2.carbon.identity.oauth2.token.handler.clientauth.jwt.validator.grant.JWTAuthorizationCodeGrantValidator</GrantTypeValidatorImplClass>
        </SupportedGrantType>
        <SupportedGrantType>
            <GrantTypeName>client_credentials</GrantTypeName>
            <GrantTypeHandlerImplClass>org.wso2.carbon.identity.oauth2.token.handlers.grant.ClientCredentialsGrantHandler</GrantTypeHandlerImplClass>
            <GrantTypeValidatorImplClass>org.wso2.carbon.identity.oauth2.token.handler.clientauth.jwt.validator.grant.JWTClientCredentialGrantValidator</GrantTypeValidatorImplClass>
        </SupportedGrantType>

5. Create new table in Identity datasource configured in <IS_HOME>/repository/conf/identity/identity.xml
   - h2.sql
       ```CREATE TABLE IF NOT EXISTS IDN_JWT_PRIVATE_KEY (JWT_ID VARCHAR(255), EXP_TIME TIMESTAMP DEFAULT 0,
       TIME_CREATED TIMESTAMP DEFAULT 0, PRIMARY KEY (JWT_ID));```
   - mysql.sql, mysql-5.7.sql, postgres.sql
       ```CREATE TABLE IF NOT EXISTS IDN_JWT_PRIVATE_KEY (JWT_ID VARCHAR(255), EXP_TIME TIMESTAMP DEFAULT 0,
       TIME_CREATED TIMESTAMP DEFAULT 0, PRIMARY KEY (JWT_ID));```

   - db2.sql
      ```CREATE TABLE IDN_JWT_PRIVATE_KEY (JWT_ID VARCHAR(255), EXP_TIME TIMESTAMP,
       TIME_CREATED TIMESTAMP, PRIMARY KEY (JWT_ID))```
   - oracle.sql, oracle-rac.sql
       ```CREATE TABLE IDN_JWT_PRIVATE_KEY (JWT_ID VARCHAR(255), EXP_TIME TIMESTAMP,
       TIME_CREATED TIMESTAMP, PRIMARY KEY (JWT_ID))```

6. Add Cache-configuration entry in <IS_HOME>/repository/conf/identity/identity.xml as below

        <CacheConfig>
           <CacheManager name="IdentityApplicationManagementCacheManager">
              ...
   	            <Cache name="PrivateKeyJWT" enable="true" timeout="10" capacity="5000" isDistributed="false"/>
           </CacheManager>
       </CacheConfig>
       
7. Restart Server
8. Add service provider
    - Select Add under Service Providers menu in the Main menu.
    - Fill in the Service Provider Name and provide a brief Description of the service provider.
    - Expand the OAuth/OpenID Connect Configuration and click Configure.
    - Enter a callback url for example http://localhost:8080/playground2/oauth2client and click Add.
    - The OAuth Client Key and OAuth Client Secret will now be visible.

9. Import the public key of the private_key_jwt issuer.
    - Rename the public key certificate file of the private_key_jwt issuer, with the Client Key of the above auth app
    - Log in to the API Manager's management console (https://localhost:9443/carbon) using admin/admin credentials and select Keystores under Manage menu in the Main menu.
    - Import above cert in to the default key store defined in <IS_HOME>/repository/conf/carbon.xml. In a default pack, keystore name is wso2carbon.jks

10. The cURL command below can be used to retrieve access token and refresh token using a JWT.
    ```curl -v POST -H "Content-Type: application/x-www-form-urlencoded;charset=UTF-8" -k -d 'client_id=<clientid>&grant_type=authorization_code&code=$CODE&client_assertion_type=urn%3Aietf%3Aparams%3Aoauth%3Aclient-assertion-type%3Ajwt-bearer&client_assertion=<private_key_jwt>&redirect_uri=http://localhost:8080/playground2/oauth2client" https://localhost:9443/oauth2/token```
