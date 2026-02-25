
# CloudFormation Basics with AWS CLI (Default Profile)

This guide contains simple CloudFormation exercises executed entirely via the AWS CLI.

Prerequisites:
- AWS CLI installed
- Access Key ID
- Secret Access Key
- Usage of the default profile

---

# 0. AWS CLI Configuration (Default Profile)

Configure AWS CLI using your Access Key and Secret Access Key:
```
aws configure
```
Enter:
```
AWS Access Key ID: <YOUR_ACCESS_KEY>
AWS Secret Access Key: <YOUR_SECRET_KEY>
Default region name: us-east-1
Default output format: json
```
Test authentication:
```
aws sts get-caller-identity
```
If an Account ID is returned, authentication is successful.

---

# CloudFormation CLI Basics

Validate template:
```
aws cloudformation validate-template --template-body file://template.yml
```
Deploy stack (create or update):
```
aws cloudformation deploy \
  --stack-name <STACK_NAME> \
  --template-file template.yml \
  --parameter-overrides Key=Value
```
Show stack outputs:
```
aws cloudformation describe-stacks \
  --stack-name <STACK_NAME> \
  --query "Stacks[0].Outputs" \
  --output table
```
Delete stack:
```
aws cloudformation delete-stack --stack-name <STACK_NAME>
aws cloudformation wait stack-delete-complete --stack-name <STACK_NAME>
```
---

# Exercise 1 – Create an S3 Bucket

Create file: bucket.yml
```
AWSTemplateFormatVersion: '2010-09-09'
Description: Simple S3 Bucket

Parameters:
  BucketName:
    Type: String
    Description: Globally unique bucket name

Resources:
  MyBucket:
    Type: AWS::S3::Bucket
    Properties:
      BucketName: !Ref BucketName

Outputs:
  BucketName:
    Value: !Ref MyBucket
  BucketArn:
    Value: !GetAtt MyBucket.Arn
```
Validate:
```
aws cloudformation validate-template --template-body file://bucket.yml
```
Deploy:
```
STACK=bucket-demo
BUCKET=myname-demo-$(date +%s)

aws cloudformation deploy \
  --stack-name $STACK \
  --template-file bucket.yml \
  --parameter-overrides BucketName=$BUCKET
```
Verify:
```
aws s3 ls | grep $BUCKET
```
Delete:
```
aws cloudformation delete-stack --stack-name $STACK
aws cloudformation wait stack-delete-complete --stack-name $STACK
```
---

# Exercise 2 – Create a Security Group (HTTP)

Get default VPC ID:
```
aws ec2 describe-vpcs \
  --query "Vpcs[?IsDefault==\`true\`].VpcId" \
  --output text
```
Set:
```
VPC_ID=<YOUR_VPC_ID>
```
Create file: sg.yml
```
AWSTemplateFormatVersion: '2010-09-09'
Description: Security Group allowing HTTP

Parameters:
  VpcId:
    Type: AWS::EC2::VPC::Id

Resources:
  WebSecurityGroup:
    Type: AWS::EC2::SecurityGroup
    Properties:
      GroupDescription: Allow HTTP
      VpcId: !Ref VpcId
      SecurityGroupIngress:
        - IpProtocol: tcp
          FromPort: 80
          ToPort: 80
          CidrIp: 0.0.0.0/0

Outputs:
  SecurityGroupId:
    Value: !Ref WebSecurityGroup
```
Deploy:
```
STACK=sg-http
```
```
aws cloudformation deploy \
  --stack-name $STACK \
  --template-file sg.yml \
  --parameter-overrides VpcId=$VPC_ID
```
Delete:
```
aws cloudformation delete-stack --stack-name $STACK
aws cloudformation wait stack-delete-complete --stack-name $STACK
```
---

# Exercise 3 – Parameters and Environments

Create file: env.yml
```
AWSTemplateFormatVersion: '2010-09-09'
Description: Environment demo

Parameters:
  Environment:
    Type: String
    AllowedValues: [dev, prod]
    Default: dev

Resources:
  DemoBucket:
    Type: AWS::S3::Bucket
    Properties:
      BucketName: !Sub "demo-${Environment}-${AWS::AccountId}-${AWS::Region}"

Outputs:
  Environment:
    Value: !Ref Environment
  BucketName:
    Value: !Ref DemoBucket
```
Deploy:
```
aws cloudformation deploy \
  --stack-name demo-dev \
  --template-file env.yml \
  --parameter-overrides Environment=dev
```
```
aws cloudformation deploy \
  --stack-name demo-prod \
  --template-file env.yml \
  --parameter-overrides Environment=prod
```
Cleanup:
```
aws cloudformation delete-stack --stack-name demo-dev
aws cloudformation delete-stack --stack-name demo-prod
```
```
aws cloudformation wait stack-delete-complete --stack-name demo-dev
aws cloudformation wait stack-delete-complete --stack-name demo-prod
```
---

# Debugging

Show stack events:
```
aws cloudformation describe-stack-events \
  --stack-name <STACK_NAME> \
  --max-items 10
```
Show stack resources:
```
aws cloudformation describe-stack-resources \
  --stack-name <STACK_NAME>
```
---

# Learning Goals

After completing these exercises, you can:

- Validate CloudFormation templates
- Deploy stacks via CLI
- Use parameters
- Read outputs
- Delete stacks cleanly
- Operate CloudFormation entirely via AWS CLI
