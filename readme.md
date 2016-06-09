Simple App for verifying AWS and Okta as IdP using OIDC.
Sample app taken from [Setting up Amazon Cognito with OIDC and Salesforce as an Identity Provider](https://blogs.aws.amazon.com/security/post/Tx3LP54JOGBE0AY/Building-an-App-using-Amazon-Cognito-and-an-OpenID-Connect-Identity-Provider) and modified to work with okta. 

Setup is needed on the AWS side of things (just replace everything Salesforce-esque with Okta-esque).
**Note**: be sure to setup things in a Cognito enabled location in aws (us-east-1 for example)

# Setup Steps in Okta and AWS

## Setup DynamoDb Table to fetch
Go to: [AWS DynoDb](https://console.aws.amazon.com/dynamodb/home?region=us-east-1)

**Note**: You can grab any data or make it more complex, just including the steps from the example.

### Create Table
[Create ProductCatalog table](http://docs.aws.amazon.com/amazondynamodb/latest/developerguide/SampleData.CreateTables.html)

### Load data into table
[Load some products into the table](http://docs.aws.amazon.com/amazondynamodb/latest/developerguide/SampleData.LoadData.html)

### Get table ARN
View Table overview and grab the Amazon Resource Name (ARN) 
ex: `arn:aws:dynamodb:us-east-1:1234567890:table/ProductCatalog`

# Create Client in Okta
**Note**: Currently in Beta. You can also create a client through the [API](http://developer.okta.com/docs/api/resources/oauth-clients)

1. Log in to Okta and [Create an Application with Application Wizard](https://support.okta.com/help/articles/Knowledge_Article/Using-the-App-Integration-Wizard) **Note**: Since the product is still not GA, you will need the OPENID_CONNECT flag enabled on your org

2. App Settings

 Platform: `Web`

 Sign on Method: `OpenIdConnect`

 Redirect URI: `https://localhost:8080/callback.html`

3. When finished, go to the app Settings and select 'Implicit' from **Allowed Grant Types**.

Also, here grab the `clientId` for future reference

# Create OIDC Identity Provider in AWS
[Create OIDC IdP in AWS documentation](http://docs.aws.amazon.com/IAM/latest/UserGuide/id_roles_providers_create_oidc.html#manage-oidc-provider-console) or [First part of Task 2 in the blog](https://blogs.aws.amazon.com/security/post/Tx3LP54JOGBE0AY/Building-an-App-using-Amazon-Cognito-and-an-OpenID-Connect-Identity-Provider)

# Setup an Identity Pool
Easiest to follow [here](https://blogs.aws.amazon.com/security/post/Tx3LP54JOGBE0AY/Building-an-App-using-Amazon-Cognito-and-an-OpenID-Connect-Identity-Provider) under the Task 2 section describing the setup. This will automatically create some roles for us to use in the next step.

More detailed setup guide from amazon [here](http://docs.aws.amazon.com/IAM/latest/UserGuide/id_roles_create_for-idp_oidc.html)
(will be under “Manage Federated Identities”)

Get the Identity Pool ID (under the pool information, Sample Code sidebar)

ex: `us-east-1:0as53gsd-fasd-4a135-asdt-gsd3236264`

# Setup the Role in AWS
Once again, best to continuing following the steps [here](https://blogs.aws.amazon.com/security/post/Tx3LP54JOGBE0AY/Building-an-App-using-Amazon-Cognito-and-an-OpenID-Connect-Identity-Provider).

Only difference is you should create the Policy ahead of time, as there is no longer an option to attach custom straight from the Role.

Go to Policies and create the custom one provided from the blog:


    {  
    "Version": "2012-10-17",
    "Statement":
    [{       

    "Effect": "Allow",
    "Action": "dynamodb:scan",
    "Resource": "arn:aws:dynamodb:us-east-1:1234567890:table/ProductCatalog"

    }] 
    }


# Web App Setup

## File changes
### index.html
**client_id** - client ID you get from Okta when registering the OIDC app

**url** - edit to point to your okta orga

### callback.html
**provider_url** - change to your okta org provider url (from .well-known)

**pool_id** - Change to the pool Id from Task 2, Congito Pool setup steps in the setup link.

**aws_region** - (optional) if you change it during your step

## Setup/Running Server
Runs with [http-server](https://github.com/indexzero/http-server)

1. Configure certs
`openssl req -newkey rsa:2048 -new -nodes -x509 -days 3650 -keyout key.pem -out cert.pem`

2. Instal/start server with SSL

`npm install http-server -g`

`jitsu install http-server`

`node /bin/http-server -S -C cert.pem -o`

This will start it up on `https://localhost:8080`

You can then click to sign in with Okta and it should query your ProductCatalog table in AWS using OIDC!

