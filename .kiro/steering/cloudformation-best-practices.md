---
title: CloudFormation Best Practices
inclusion: fileMatch
fileMatchPattern: '**/*.template'
---

# CloudFormation Best Practices

## Template Organization
- Use descriptive template names: `CustomControlTower-<Purpose>.template`
- Group related resources logically
- Use nested stacks for complex deployments
- Keep templates under 51,200 bytes (use nested stacks if larger)

## Naming Conventions
- Resource logical IDs: PascalCase (e.g., `VPCFlowLogBucket`)
- Parameter names: PascalCase (e.g., `KmsKeyId`)
- Output names: PascalCase (e.g., `BucketName`)
- Tag keys: PascalCase (e.g., `Application`, `Environment`)

## Parameters
- Provide clear descriptions for all parameters
- Use appropriate parameter types (String, Number, List, AWS-specific)
- Set sensible default values when applicable
- Use AllowedValues for constrained inputs
- Document parameter constraints

```yaml
Parameters:
  KmsKeyId:
    Type: String
    Description: KMS key ID for encryption
    Default: alias/aws/ebs
  
  Environment:
    Type: String
    Description: Environment name
    AllowedValues:
      - Dev
      - PreProd
      - Prod
```

## Outputs
- Export values needed by other stacks or resources
- Use descriptive output names
- Include descriptions for all outputs
- Export to SSM Parameter Store for cross-account access

```yaml
Outputs:
  BucketName:
    Description: Name of the S3 bucket
    Value: !Ref LogBucket
    Export:
      Name: !Sub '${AWS::StackName}-BucketName'
```

## SSM Parameter Exports
- Use consistent parameter naming: `/org/member/<account>/<resource>`
- Store cross-account resource identifiers
- Use parameter values in dependent stacks
- Document parameter dependencies

```yaml
# In manifest.yaml
export_outputs:
  - name: /org/member/logging/bucket_name
    value: $[output_BucketName]
```

## Resource Configuration
- Use intrinsic functions (Ref, GetAtt, Sub, Join)
- Tag all resources with standard tags
- Enable encryption by default
- Configure logging and monitoring
- Use DeletionPolicy for critical resources

```yaml
Resources:
  LogBucket:
    Type: AWS::S3::Bucket
    DeletionPolicy: Retain
    Properties:
      BucketName: !Sub 'logs-${AWS::AccountId}-${AWS::Region}'
      BucketEncryption:
        ServerSideEncryptionConfiguration:
          - ServerSideEncryptionByDefault:
              SSEAlgorithm: AES256
      Tags:
        - Key: Application
          Value: Logging
        - Key: ManagedBy
          Value: CloudFormation
```

## Lambda Integration
- Reference Lambda code from S3
- Use consistent S3 key structure
- Include Lambda execution role in template
- Configure appropriate timeout and memory

```yaml
LambdaFunction:
  Type: AWS::Lambda::Function
  Properties:
    FunctionName: !Sub 'CustomControlTower-${Purpose}'
    Runtime: python3.12
    Handler: index.lambda_handler
    Code:
      S3Bucket: !Sub 'custom-control-tower-lambda-functions-${AWS::AccountId}-${AWS::Region}'
      S3Key: !Sub '${LambdaDirectory}/function.zip'
    Role: !GetAtt LambdaExecutionRole.Arn
    Timeout: 300
    MemorySize: 256
```

## Custom Resources
- Use Lambda-backed custom resources for unsupported operations
- Include ServiceToken pointing to Lambda ARN
- Pass necessary properties to Lambda
- Handle all lifecycle events (Create, Update, Delete)

```yaml
CustomResource:
  Type: Custom::EBSEncryption
  Properties:
    ServiceToken: !GetAtt EnableEBSEncryptionFunction.Arn
    KmsKeyId: !Ref KmsKey
```

## StackSets
- Use StackSets for multi-account deployments
- Target specific OUs or accounts
- Configure appropriate regions
- Handle StackSet operation failures gracefully
- Use parameters for environment-specific values

## IAM Resources
- Use managed policies when possible
- Follow least privilege principle
- Use policy conditions for additional security
- Include permission boundaries
- Document policy purposes

```yaml
LambdaExecutionRole:
  Type: AWS::IAM::Role
  Properties:
    AssumeRolePolicyDocument:
      Version: '2012-10-17'
      Statement:
        - Effect: Allow
          Principal:
            Service: lambda.amazonaws.com
          Action: sts:AssumeRole
    ManagedPolicyArns:
      - arn:aws:iam::aws:policy/service-role/AWSLambdaBasicExecutionRole
    Policies:
      - PolicyName: EC2EncryptionAccess
        PolicyDocument:
          Version: '2012-10-17'
          Statement:
            - Effect: Allow
              Action:
                - ec2:EnableEbsEncryptionByDefault
                - ec2:GetEbsDefaultKmsKeyId
                - ec2:ModifyEbsDefaultKmsKeyId
              Resource: '*'
```

## Security Best Practices
- Enable encryption for all data stores
- Use KMS customer managed keys
- Configure bucket policies for S3
- Enable versioning for critical buckets
- Use VPC endpoints where applicable
- Enable CloudTrail logging

## Validation
- Validate templates before deployment using `aws cloudformation validate-template`
- Test in non-production environments first
- Use CloudFormation drift detection
- Monitor stack events during deployment
- Review StackSet operation results

## Documentation
- Include template description
- Document all parameters and outputs
- Add comments for complex logic
- Maintain version history in comments
- Reference AWS documentation for resource types
