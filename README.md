# Litware Groceries Demo

This repository includes all the relevant source code and policies for the demo site: <https://aka.ms/CIAMDemo>

You can configure the demo here:  <https://aka.ms/CIAMDemoConfig>

The project shows [Azure AD Custom policies](https://docs.microsoft.com/en-us/azure/active-directory-b2c/custom-policy-overview) capabilites in user journeys for moder apps.

## Project Strucutre

This project has three main packages : [Ploicies](https://github.com/azure-ad-b2c/woodgrove-groceries-demo/tree/Release/Policies) witch contains all Azure AD custom polices, [Scripts](https://github.com/azure-ad-b2c/woodgrove-groceries-demo/tree/Release/Scripts) witch contains scripts to build the project and [Src](https://github.com/azure-ad-b2c/woodgrove-groceries-demo/tree/Release/src/WoodGroveGroceriesWebApplication)) witch contains source code.


## Build the project

To build this project you need to configure

- ### Dev environment
  - Install [VS-Code](https://code.visualstudio.com/)
  - Add [Azure AD B2C extensions](https://marketplace.visualstudio.com/items?itemName=AzureADB2CTools.aadb2c)
  - Install [PowerShell](https://code.visualstudio.com/docs/languages/powershell)
  - Install [dotnet 3.1](https://dotnet.microsoft.com/en-us/download/dotnet/3.1) or higher
  - Install [npm](https://docs.npmjs.com/cli/v7/configuring-npm/install/)


 - ### Cloud environment
    - [Create](https://docs.microsoft.com/en-us/azure/active-directory-b2c/tutorial-create-tenant#:~:text=Create%20Azure%20AD%20B2C%20Tenant%201%20Sign%20in,Create%20.%20The%20domain%20...%20.%20See%20More) an Azure Active Directory (Azure AD) B2C tenant.
    - [Register](https://docs.microsoft.com/en-us/azure/active-directory-b2c/tutorial-register-applications?tabs=app-reg-ga) your app on the tenant you created previously.  Add your app redirect URIs. By default, the app run [https://localhost:5001](https://localhost:5001) in development mode. The authentication callbacks are [https://localhost:5001/b2b-signin-callback](https://localhost:5001/b2b-signin-callback), [https://localhost:5001/betaaccess-callback](https://localhost:5001/betaaccess-callback) and [https://localhost:5001/partner-signin-callback](https://localhost:5001/partner-signin-callback). You may also need to add [https://yourappdomain.azurewebsites.net/yourtenant.onmicrosoft.com/oauth2/authresp](https://yourappdomain.azurewebsites.net/yourtenant.onmicrosoft.com/oauth2/authresp) redirect URI after deploying your app.
    - Add [Identity providers](https://docs.microsoft.com/en-us/azure/active-directory-b2c/add-identity-provider) according to your needs
    - [Create](https://docs.microsoft.com/en-us/azure/active-directory-b2c/tutorial-create-user-flows?pivots=b2c-custom-policy) user flows and ploicy keys

## Project updates
  - ### **Plocies**
    - [B2C_1A_base.xml](Policies/B2C_1A_base.xml)
      - replace all **yourtenant** by your Azure AD B2C tenant name
      - Update the template_id value with the ID of your SendGrid template and form.email value by your sender email address 
        ```xml
          <InputParameters>
          <!-- Update the template_id value with the ID of your SendGrid template. -->
             <InputParameter Id="template_id" DataType="string" Value="sendgrid-templateid-invitation"/>
             <InputParameter Id="from.email" DataType="string" Value="reply@address-here"/>
          </InputParameters>
        ```
      - Find and replace all **B2C_1A_TokenSigningKeyContainer** by the policy key name you created for token sign in and **B2C_1A_TokenEncryptionKeyContainer** by the one you created  for token encryption
      - If you have added Facebook as an idententiy provider, add an item for client_id in metadata like below
        ```xml
                 <TechnicalProfile Id="Facebook-OAUTH">
          <!-- The text in the following DisplayName element is shown to the user on the claims provider 
               selection screen. -->
          <DisplayName>Facebook</DisplayName>
          <Protocol Name="OAuth2" />
          <Metadata>
            <Item Key="ProviderName">facebook</Item>
            <Item Key="authorization_endpoint">https://www.facebook.com/dialog/oauth</Item>
            <Item Key="AccessTokenEndpoint">https://graph.facebook.com/oauth/access_token</Item>
            <Item Key="HttpBinding">GET</Item>
            <Item Key="UsePolicyInRedirectUri">0</Item>
            <Item Key="client_id">your facebook client id</Item>
            <Item Key="scope">email public_profile</Item>
            <Item Key="ClaimsEndpoint">https://graph.facebook.com/me?fields=id,first_name,last_name,name,email</Item>
            <!-- The Facebook required HTTP GET method, but the access token response is in JSON format from 3/27/2017 -->
            <Item Key="AccessTokenResponseFormat">json</Item>
          </Metadata>
          <CryptographicKeys>
            <Key Id="client_secret" StorageReferenceId="B2C_1A_Facebook" />
          </CryptographicKeys>
          <InputClaims />
          <OutputClaims>
            <OutputClaim ClaimTypeReferenceId="userId" PartnerClaimType="id" />
            <!-- <OutputClaim ClaimTypeReferenceId="givenName" PartnerClaimType="first_name" />
            <OutputClaim ClaimTypeReferenceId="surname" PartnerClaimType="last_name" /> -->
            <OutputClaim ClaimTypeReferenceId="displayName" PartnerClaimType="name" />
            <OutputClaim ClaimTypeReferenceId="email" PartnerClaimType="email" />
            <OutputClaim ClaimTypeReferenceId="identityProvider" DefaultValue="facebook.com" />
            <OutputClaim ClaimTypeReferenceId="authenticationSource" DefaultValue="socialIdpAuthentication" />
            <OutputClaim ClaimTypeReferenceId="identityProviderAccessToken" PartnerClaimType="{oauth2:access_token}" />
          </OutputClaims>
          <OutputClaimsTransformations>
            <OutputClaimsTransformation ReferenceId="CreateAlternativeSecurityId" />
            <OutputClaimsTransformation ReferenceId="CreateExternalUserName" />
            <OutputClaimsTransformation ReferenceId="CreateExternalUserPrincipalName" />
          </OutputClaimsTransformations>
          <UseTechnicalProfileForSessionManagement ReferenceId="SSOSession-ExternalLogin" />
        </TechnicalProfile>
        ```
        and replace B2C_1A_Facebook by the policy key name you created for Facebook

      - Do the same process if you have added Google as an IdP
      - Replace {tenant} by your tenant name in these two lines
        ```xml
                  <Item Key="authorization_endpoint">https://login.microsoftonline.com/{tenant}/oauth2/token</Item>
                
                  <Item Key="METADATA">https://login.microsoftonline.com/{tenant}/.well-known/openid-configuration</Item>
        ```
      - If you have Microsoft as an  IdP, add client_id in the meta data, and its value is the Copy app ID you created on your tenant. 
        ```xml
          <Metadata>
            <Item Key="METADATA">https://login.windows.net/consumers/v2.0/.well-known/openid-configuration</Item>
            <Item Key="ProviderName">https://login.microsoftonline.com/9188040d-6c67-4c5b-b112-36a304b66dad/v2.0</Item>
            <Item Key="response_types">id_token</Item>
            <Item Key="client_id">your app ID (client ID)</Item>
            <Item Key="scope">openid profile email</Item>
            <Item Key="UsePolicyInRedirectUri">false</Item>
          </Metadata>
        ```
        Replace the **B2C_1A_MSA** by the policy key you created for Microsoft IdP
      - Find **32e26e5a-8a00-4d1a-8ff3-e90a5615be38** by your app Id in commonaad ClaimsProvider and **B2C_1A_AADSecret** by the policy key you created for Azure AD
      - Replcae **B2C_1A_SendGrid** by the policy key you created for **SendGrid**
      - Upload the **B2C_1A_base** policy on your Identity Experience Framework
    - [B2C_1A_ext_Localization.xml](Policies/B2C_1A_ext_Localization.xml)
        - Open the locatization xml file  
        - Replace all **yourtenant** by your tenant name
        - If you have deployed your app on azure, replace all **-- deployed app location url --** by your domain. You can also also replace by your own API endpoint
        - Upload the policy on your Identity Experience Framework
    - [B2C_1A_ext.xml](Policies/B2C_1A_ext.xml)  
        - Open the extesnions file
        - Replace all **yourtenant** by your tenant name
        - Replcae **d8817253-63f7-42a7-9420-dd62f59dd3f3** by your b2c-extension-app object ID and **2ec06a55-5091-472e-9c01-8526946b848f** by its application ID
        - Replace **-- App Insights Instrumentation Key --** by your application insights key if you have one
        - Replace all **ProxyIdentityExperienceFrameworkAppId** by your ProxyIdentityExperienceFramework app ID and all **IdentityExperienceFrameworkAppId** by your IdentityExperienceFramework app ID
        - Replace **google_clientid** by google client id and **facebook_clientid** by your Facebook client ID
        - Replace all **msa_clientid** by the app client id you created on the tenant
        - Upload the policy on your Identity Experience Framework
    - **Profiles Policies**
      - In the all other ploices replace **yourtenant** by your tenant name
      - Upload all of them on your Identity Experience Framework
  - ### **Source code**
    - Open [New-B2CPolicy.ps1](Scripts/New-B2CPolicy/New-B2CPolicy.ps1) and [New-B2CTenant.ps1](Scripts/New-B2CTenant/New-B2CTenant.ps1) and replace all **ClientId** values by your app client ID and **00000002-0000-0000-c000-000000000000** by your app directory (tenant) ID
    - In the [appsettings.json](src/WoodGroveGroceriesWebApplication/appsettings.Development.json)
      - Replace all **yourtenant** by your tenant name
      - Replace **ClientId** value by your app client Id
      - Replace **PolicyPrefix** value by your own prefix. e.g **B2C_1A**
      - Replace **TenantId** value by your tenant ID
      - Back to your client app on Azure, navigate to **Certificates & secrets**, add a new client secret. Copie the Value and paste in your appsettings **ClientSecret** values
      - Replace the value of **ApiKey** by your **SendGrid** API key

    - Navigate to [Src/WoodGroveGroceriesWebApplication](Src/WoodGroveGroceriesWebApplication/) directory with command line interface. And execute :
      - <pre> dotnet restore </pre>
      - <pre> dotnet dev-certs https --clean <br/> dotnet dev-certs https --trust </pre>
      - And finally 
        - <pre> dotnet run </pre>
      - Navigate to your [https://localhost:5001](https://localhost:5001)
