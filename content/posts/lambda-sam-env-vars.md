+++
date = "2018-08-17T14:16:20+00:00"
draft = false
title = "Dynamically passing in ENV vars to Lambda functions created by Cloudformation"
keywords = [lambda, environment variables, cloudformation]
tags = [infrastructure, configuration, lambda]
+++

# Background

I'm going to assume you have experience with Lambda, [AWS-SAM](https://github.com/awslabs/serverless-application-model), and Cloudformation. 
To start with you should already have a [template.yml](https://github.com/awslabs/serverless-application-model/blob/master/examples/2016-10-31/api_swagger_cors/template.yaml) for Cloudformation that looks alot like this:
```
---
AWSTemplateFormatVersion: '2010-09-09'
Transform: AWS::Serverless-2016-10-31
Description: AWS template
Resources:
  ApiGatewayApi:
    Type: AWS::Serverless::Api
    Properties:
      StageName: Prod

      # Allows www.example.com to call these APIs
      # SAM will automatically add AllowMethods with a list of methods for this API
      Cors: "'www.example.com'"

      DefinitionBody:
          'Fn::Transform':
            Name: 'AWS::Include'
            # Replace <bucket> with your bucket name
            Parameters:
              Location: s3://<bucket>/swagger.yaml

  LambdaFunction:
    Type: AWS::Serverless::Function
    Properties:
      # Replace <bucket> with your bucket name
      CodeUri: s3://<bucket>/api_swagger_cors.zip
      Handler: index.handler
      Runtime: nodejs6.10
      Events:
        ProxyApiRoot:
          Type: Api
          Properties:
            RestApiId: !Ref ApiGatewayApi
            Path: /
            Method: ANY
        ProxyApiGreedy:
          Type: Api
          Properties:
            RestApiId: !Ref ApiGatewayApi
            Path: /{proxy+}
            Method: ANY

Outputs:
  ApiUrl:
    Description: URL of your API endpoint
    Value: !Join
      - ''
      - - https://
        - !Ref ApiGatewayApi
        - '.execute-api.'
        - !Ref 'AWS::Region'
        - '.amazonaws.com/Prod'
```

There are two distinct approaches you can take to dynamically retrieve environment variables in Lambda. The approach you take is dictated by how frequently your variable changes. 

1. Your environment variable changes frequently.
    - I would recommend retrieving the variable from an external endpoint. We won't cover this use case in the blog post.
2. Your environment variables is fairly static, but you don't want to hardcode the variable or make it public in a code repository.
    - Retrieving from AWS SSM parameter store as a string is the best option in my opinion. 
    
--- 
## Let's walk through method #2.

1. To get started, go to the [parameter store](https://console.aws.amazon.com/systems-manager/parameters?region=us-east-1) and add the name of your environment variable, it's value and set the type to "String". 
    - Cloudformation doesn't support Secure String at the time of writing.
2. Adjust your Cloudformation's role to be able to access the newly created parameter 
3. Add the parameter to the Cloudformation template via the following:
  -   ```
        MyEnvVarParameterName:
          Type: 'Aws::SSM::Parameter::Value<String>
          Default: MyEnvParamaterNameSetInTheStore
      ```
4. Add the parameter to you lambda function as an environment variable with the following:
  -   ```
        MyLambdaEnvName:
          Ref: MyEnvVarParameterName
      ```
5. If you would like the environment variable to be hidden from view in the AWS Console then set NoEcho property to true.

--- 

So your final version would look something like this:
```
---
AWSTemplateFormatVersion: '2010-09-09'
Transform: AWS::Serverless-2016-10-31
Description: AWS Template
Parameters:
  MyEnvParameterName:
    Type: 'Aws::SSM::Parameter::Value<String>
    Default: MyEnvParamaterNameSetInTheStore
Resources:
  ApiGatewayApi:
    Type: AWS::Serverless::Api
    Properties:
      StageName: Prod

      # Allows www.example.com to call these APIs
      # SAM will automatically add AllowMethods with a list of methods for this API
      Cors: "'www.example.com'"

      DefinitionBody:
          'Fn::Transform':
            Name: 'AWS::Include'
            # Replace <bucket> with your bucket name
            Parameters:
              Location: s3://<bucket>/swagger.yaml

  LambdaFunction:
    Type: AWS::Serverless::Function
    Properties:
      # Replace <bucket> with your bucket name
      CodeUri: s3://<bucket>/api_swagger_cors.zip
      Handler: index.handler
      Runtime: nodejs6.10
      Environment:
        Variables:
          MyLambdaEnvName:
            Ref: MyEnvParameterName
      Events:
        ProxyApiRoot:
          Type: Api
          Properties:
            RestApiId: !Ref ApiGatewayApi
            Path: /
            Method: ANY
        ProxyApiGreedy:
          Type: Api
          Properties:
            RestApiId: !Ref ApiGatewayApi
            Path: /{proxy+}
            Method: ANY

Outputs:
  ApiUrl:
    Description: URL of your API endpoint
    Value: !Join
      - ''
      - - https://
        - !Ref ApiGatewayApi
        - '.execute-api.'
        - !Ref 'AWS::Region'
        - '.amazonaws.com/Prod'
```


