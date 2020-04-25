# s3.getSignedUrl behaves differently in eu-west-1 vs eu-central-1

This works:

```
sam build
# Deploy to eu-central-1
sam deploy --guided 
curl --verbose -L https://{apiId}.execute-api.eu-central-1.amazonaws.com/Prod --upload-file README.md 
```

Observe: 
 - Curl follows the 307 redirect and uploads to S3 ending with an error 200 status code.
- Query parameters in redirect `Location` header populated with the result of `s3.getSignedUrl` include:
   - X-Amz-Expires
   - X-Amz-Signature
   - X-Amz-Security-Token
   - X-Amz-Algorithm
   - X-Amz-Credential
   - X-Amz-Date
   - X-Amz-SignedHeaders
   
This doesn't work:

```
sam build
# Deploy to eu-west-1
sam deploy --guided 
curl --verbose -L https://{apiId}.execute-api.eu-west-1.amazonaws.com/Prod --upload-file README.md 
```

Observe
 - Curl follows the 307 redirect and fails to upload to S3 ending with an error 403 status code.
- Query parameters in redirect `Location` header populated with the result of `s3.getSignedUrl` include:
   - Content-Type
   - Expires
   - Signature
   - x-amz-security-token
   - AWSAccessKeyId

So it would appear that the eu-central-1 and eu-west-1 regions create different signed urls and that the urls created
in eu-west-1 are not valid.
