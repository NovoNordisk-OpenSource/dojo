Novo Nordisk Microsoft Entra ID Training - Code Kata #1
======================================

This training exercise is a **beginner-level** course on [Microsoft Entra ID](https://learn.microsoft.com/en-us/entra/identity/) at Novo Nordisk and serves as a starting point for developers looking to onboard [Heimdall](https://medium.com/@ZaradarTR/wtf-is-heimdall-646843ec18d0).

## Getting started
These instructions will help you prepare for the kata and ensure that your training machine has the tools installed you will need to complete the assignment(s). If you find yourself in a situation where one or more tools might not be available for your training environment please reach out to your instructor for assistance on how to proceed, post an [issue in our repository](https://github.com/NovoNordisk-OpenSource/dojo/issues) or fix it yourself and update the kata via a [pull request](https://github.com/NovoNordisk-OpenSource/dojo/pulls).

### Prerequisites
* [Azure](https://portal.azure.com/)
* [Azure CLI](https://learn.microsoft.com/en-us/cli/azure/install-azure-cli#install)
* [Git](https://git-scm.com/book/en/v2/Getting-Started-Installing-Git)

## Exercise
This instructional exercise is meticulously designed to serve as a comprehensive guide, taking you step-by-step through the landscape of [Microsoft Entra ID](https://learn.microsoft.com/en-us/entra/identity/) application registrations. Its purpose is not solely limited to getting you started on the [IAM](https://www.microsoft.com/en-gb/security/business/security-101/what-is-identity-access-management-iam) training; it also aims to provide a detailed walkthrough that acquaints you with the nuanced functionalities of application manifests to ensure we have a broad shared understanding of the taxonomy associated with this particulare subject.

### 1. Login to Azure portal
To get started we need to login to Azure to ensure we have permissions to interact with our [Microsoft Entra ID](https://learn.microsoft.com/en-us/entra/identity/) tenant:

```bash
az login -u johndoe@contoso.com -p VerySecret
```

***Note*** <br/>
By default, this command logs in with a user account and will pick up on the active windows user if no values are supplied. [Azure CLI](https://learn.microsoft.com/en-us/cli/azure) will try to launch a web browser to log in interactively. If a web browser is not available, CLI will fall back to device code login. To login with a service principal, specify `--service-principal`.

### 2. Create a kata directory
Once we have our access permissions sorted we can setup a directory for our exercise files. It's pretty straight forward:

```bash
mkdir kata1
cd kata1
```

### 3. Create application manifest
Next we can proceed to creating a `manifest.json` file in our `kata1` directory that contains the desired state of the application registration we want to provisiong with our Microsoft Entra ID tenant:

```json
{
  "id": "f7f9acfc-ae0c-4d6c-b489-0a81dc1652dd", # Unique identifier for the app in the directory. This ID is not used to identify the app in any protocol transaction. It's used for referencing the object in directory queries.
  "acceptMappedClaims": false, # Do not set acceptMappedClaims property to true for multi-tenant apps. This can allow malicious actors to create claims-mapping policies for your app.
  "accessTokenAcceptedVersion": 2, # Possible values for accesstokenAcceptedVersion are 1, 2, or null. If the value is null, this parameter defaults to 1, which corresponds to the v1.0 endpoint.
  "allowPublicClient": false, # If this value is set to true the fallback application type is public client. The default value is false which means the fallback application type is confidential client.
  "appRoles": ["MyAppRole"], # Specifies the collection of roles that an app may declare. These roles can be assigned to users, groups, or service principals. 
  "groupMembershipClaims": "None", # Configures the groups claim issued in a user or OAuth 2.0 access token that the app expects. Valid values are: None, SecurityGroup , ApplicationGroup, DirectoryRole, All
  "optionalClaims": { # The optional claims returned in tokens issued by the security token service for this specific app registration.
    "idToken": [
        {
            "name": "auth_time",
            "essential": false
        }
    ],
    "accessToken": [
        {
            "name": "ipaddr",
            "essential": false
        }
    ],
    "saml2Token": [
        {
            "name": "upn",
            "essential": false
        },
        {
            "name": "extension_ab603c56068041afb2f6832e2a17e237_skypeId",
            "source": "user",
            "essential": false
        }
    ]
  },
  "identifierUris": ["api://f7f9acfc-ae0c-4d6c-b489-0a81dc1652dd"], # User-defined URI(s) that uniquely identify a web app within its Microsoft Entra tenant or verified customer owned domain.
  "knownClientApplications": ["f7f9acfc-ae0c-4d6c-b489-0a81dc1652dd"], # Used for bundling consent if you have a solution that contains multiple application registrations
  "oauth2AllowImplicitFlow": false, # Never allow implicit flow. Its dead @ https://developer.okta.com/blog/2019/05/01/is-the-oauth-implicit-flow-dead
  "oauth2Permissions": [ # Specifies the collection of OAuth 2.0 permission scopes that the web API (resource) app exposes to client apps. 
      {
          "adminConsentDescription": "Allow the app to access resources on behalf of the signed-in user.",
          "adminConsentDisplayName": "Access resource1",
          "id": "<guid>",
          "isEnabled": true,
          "type": "User",
          "userConsentDescription": "Allow the app to access resource1 on your behalf.",
          "userConsentDisplayName": "Access resources",
          "value": "user_impersonation"
      }
  ],
  "oauth2RequirePostResponse": false, # Specifies whether, as part of OAuth 2.0 token requests, Microsoft Entra ID will allow POST requests, as opposed to GET requests. 
  "replyUrlsWithType": [ # Registered redirect_uri values that Microsoft Entra ID will accept as destinations when returning tokens. For more info: https://learn.microsoft.com/en-us/entra/identity-platform/reply-url
     {
        "url": "https://localhost:4400/services/office365/redirectTarget.html",
        "type": "InstalledClient"
     },
     {
        "url": "https://someconfidentialclient/someauthhandler",
        "type": "Web"
     },
     {
        "url": "https://somepublicclient/someauthhandler",
        "type": "Spa"
     }
  ],
  "requiredResourceAccess": [ # Specifies that your application requires access to the resource application (identified by RESOURCE_APP_ID) and requests the specified appRole (identified by GUID_OF_APP_ROLE) defined in that resource application
    {
      "resourceAppId": "RESOURCE_APP_ID",
      "resourceAccess": [
        {
          "id": "GUID_OF_APP_ROLE",
          "type": "Role"
        }
      ]
    }
  ],
}
```

***Note*** <br/>
Its worth noting that due to limitations in the `AZ CLI` we are only allowed to infer certain values from our application manifest file, however for the sake of completness we have opted to present a more verbose version to ensure that people understand all the key settings that are required for them to succesfully setup an application registration that is compatible with our Heimdall platform.

Furthermore the application manifest schema is quiet extensive and our working example only covers the essentials, thus it is worth taking time to [investigate the various configuration options at our disposal](https://learn.microsoft.com/en-us/entra/identity-platform/reference-app-manifest).

### 4. Create application registration
With the manifest in hand we can now use the `AZ CLI` to create a new application registration:

```bash
# Replace <display-name> with the display name of your Azure App Registration
display_name="<display-name>"

# Create app registration with desired display name
az ad app create --display-name $display_name

# Get the ID of the Azure App Registration
app_id=$(az ad app list --display-name "$display_name" --query '[].{AppId: appId}' --output tsv)

# Update the settings of our new app registration based on our manifest.json
az ad app update --id $app_id  --app-roles ./manifest.json --optional-claims ./manifest.json --required-resource-accesses ./manifest.json
```

***Note*** <br/>
We cannot directly create an application registration from a `manifest.json` file using the `az ad app create` command. The `az ad app create` command creates a new Azure AD application registration with the specified parameters, but it doesn't accept a manifest file as input.

To create an application registration with specific configurations defined in a `manifest.json` file, you would typically create the application first using `az ad app create`, and then update its properties using the `az ad app update` command as shown above. Alternatively you can create the application registration by hand via the Azure portal and paste in the full `manifest.json`.

## Want to help make our training material better?
 * Want to **log an issue** or **request a new kata**? Feel free to visit our [GitHub site](https://github.com/NovoNordisk-OpenSource/dojo/issues).
