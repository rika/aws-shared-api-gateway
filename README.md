# aws-shared-api-gateway
AWS Shared API Gateway with custom domain

This API Gateway can be used by many different services, so your lambdas can share a single domain.


## What you need

You will need AWS CLI, NodeJS, the [`Serverles framework`](https://serverless.com/) and this service utilizes the [`serverless-domain-manager`](https://github.com/amplify-education/serverless-domain-manager) plugin.

## Config & Deploy

In this service `serverless.yml`, configure the following:

- REGION (ex: us-east-1)
- CERTIFICATE_ARN (ex: arn:aws:acm:us-east-1:xxxx:certificate/aaaa-bbbb-cccc)
- STAGING_DOMAIN (ex: dev.my.domain.com)
- PRODUCTION_DOMAIN (ex: my.domain.com)

Create your custom domain by executing:
``` 
serverless create_domain
```
(as described in the [`serverless-domain-manager`](https://github.com/amplify-education/serverless-domain-manager) plugin page)


Deploy the API Gateway with:
```
serverless deploy --stage STAGE
```
(STAGE shoube be `prod` or `dev`)

That's it you have an API Gateway with a domain and ready to be used by other services.

## Usage in others services

In your others services, which you want to use the shared API Gateway, add the following in their `serverless.yml`:

```
provider:
  ...
  apiGateway:
    restApiId: ${cf:aws-shared-api-gateway-${opt:stage, self:provider.stage}.apiGatewayRestApiId}
    restApiRootResourceId: ${cf:aws-shared-api-gateway-${opt:stage, self:provider.stage}.apiGatewayRestApiRootResourceId}
  ...
```

In their functions add the service in the path to avoid conflicts:

```
functions:
  ...
  hello:
    handler: handler.hello
    events:
      - http:
          path: ${self:service}/hello
          method: get
  ...
```

Now your Lambda from other service will have an endpoint in this API Gateway. (ex: https://my.domain.com/hello will call my `hello` function)

## Known issues

The API Gateway service has an empty Lambda function on the root endpoint path `/` that returns 200 and an ok message.
This was made so that the `serverless-domain-manager` can be used.
An empty API Gateway with no endpoints raise some erros when creating custom the domain.
This endpoint can be changed and used to return something more useful.
For example, it could return a list of all your services/endpoints registered in this API Gateway.


## How it works

The serverless.yml in this repository creates an ApiGateway and exports Outputs of it's ApiGatewayRestApi and RestApiRootResourceId.
Thus, others services lambdas can utilize the same ApiGateway by declaring the provider apiGateway attribute as described above.

The `serverless-domain-manager` plugin is used to create the domain for this ApiGateway.

## Reference:
- https://serverless.com/framework/docs/providers/aws/events/apigateway/#share-api-gateway-and-api-resources
- https://serverless.com/blog/api-gateway-multiple-services/
- https://github.com/amplify-education/serverless-domain-manager
