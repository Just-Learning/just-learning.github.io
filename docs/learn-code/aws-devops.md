# AWS Certified DevOps - Professional

https://app.pluralsight.com/paths/certificate/aws-certified-devops-engineer

[Blueprint](https://d1.awsstatic.com/training-and-certification/docs-devops-pro/AWS_certified_devops_engineer_professional_blueprint.pdf)

Exam
- 170 minutes
- multi-response: most 2 or 3 most correct answers
- senario based question
- a lot reading

Read thru FAQ

## Continuous Delivery and Automation (55%)

### CI

```
Dev -> Repo <- CI Server
               - build
               - test

Ops -> Repo <- CI Server
               - build
               - test
```

### CD
```
Dev \
     -> Repo <- CI Server -> Dev/Prod
Ops /
```

### CI CD Pipeline

Source - Build - Staging - Production

### Infra as Code

### Configuration

### Lifecycle

AWS Services: CodeCommit, CodeDeploy, CodePipeline

AMI vs Bootstrapping
- partial custom AMI
- partial bootstrap for flexibility

Blue Green Deployment
- Low TTL on DNS record
- Backward compatible

### CloudFormation

- Infra as code
- Valid Template
- `Ref` function to define a reference
- Template store in `S3` - `TemplateURL`
- Resources `tag` for CloudFormation reference later

Resource

Parameters
- `KeyName` - `"Type": "AWS::EC2::KeyPair::KeyName"`
- `InstanceType` -  `"AllowValues": ["t2.micro", "t2.nano"]`
- `SSHLocation` - using regex valid the data, `AllowedPattern`
- `ImageId` same image has differnt image id in different region, `Fn::FindInMap` to get the image id dynamically
- `DBPassword` - set `NoEcho` to true

Mapping

Output
- log the url that deployed after launched `Fn::Join` `Fn::GetAtt`

Bootstrap
- `UserData` must be **Base64** encoded
- Install packages, config params, inject script using `Fn::Join`

Wait bootstrap finish
- WaitCondition
- WaitHandle
- `DependsOn` - Resources are created in parallel w/o `DependsOn`
- `Timeout` failure if expired

CreationPolicy
- `ResourdceSignal`
- `Timeout`

> You can turn off `rollback on failure` for troubleshooting, check log in `/var/log` or `c:\cfn\log`

Nested Templates
- Template 1 --output as input-- Template 2
- send output to another template as input params
- `Fn::GetAtt` VPCStack, Outputs.SubnetId

### Beanstalk


### Automating



## Monitoring, Metrics and Logging (20)

## Security, Governance and Validation (10%)

## High Availability and Elasticity (15%)

