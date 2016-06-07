Simple App for verifying AWS and Okta as IdP using OIDC.
Sample app taken from [Setting up Amazon Cognito with OIDC and Salesforce as an Identity Provider](https://blogs.aws.amazon.com/security/post/Tx3LP54JOGBE0AY/Building-an-App-using-Amazon-Cognito-and-an-OpenID-Connect-Identity-Provider) and modified to work with okta. 

Setup is needed on the AWS side of things (just replace everything Salesforce-esque with Okta-esque).
**Note**: be sure to setup things in a Cognito enabled location in aws (us-east-1 for example)

## File changes
### index.html
**client_id** - client ID you get from Okta when registering the OIDC app

**url** - edit to point to your okta orga

### callback.html
**provider_url** - change to your okta org provider url (from .well-known)

**pool_id** - Change to the pool Id from Task 2, Congito Pool setup steps in the setup link.

**aws_region** - (optional) if you change it during your step

## Setup and Running
Runs with [http-server](https://github.com/indexzero/http-server)

1. Configure certs
`openssl req -newkey rsa:2048 -new -nodes -x509 -days 3650 -keyout key.pem -out cert.pem`

2. Instal/start server with SSL

`npm install http-server -g`

`jitsu install http-server`

`node bin/http-server -S -C cert.pem -o`

This will start it up on `https://localhost:8080`

You can then click to sign in with Okta and it should query your ProductCatalog table in AWS using OIDC!

