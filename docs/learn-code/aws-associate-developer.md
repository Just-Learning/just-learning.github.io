
# AWS Certified Developer - Associate

## Blueprint
[AWS_certified_developer_associate_blueprint.pdf](http://awstrainingandcertification.s3.amazonaws.com/production/AWS_certified_developer_associate_blueprint.pdf)

**10** Fundamentals
- EC2, Beanstalk
- RDS, S3, DynamoDB
- SWF, SQS, CloudFormation

**40** Designing and Developing
- AMI (Amazon Machine Image)
- APIs

**40** Deployment and Security
- Security Best Practices
- IAM (Identity and Access Management)
- VPC

**10** Debugging
- General troubleshooting
- Best Practices in debugging

## 1. EC2 ❗️

[EC2 FAQ](https://aws.amazon.com/ec2/faqs/)

### Types

[EC2 Instance Types](https://aws.amazon.com/ec2/instance-types/)

- General
  - T (cheap)
  - M
- CPU
  - C
- GPU
  - P compute
  - G graphics-intensive
  - F FPGA // Field-programmable gate array
- Memory
  - X (extreme)
  - R (optimized)
- Storage
  - H (HHD high throughput)
  - I (SSD IOPS) // input output per second
  - D (HDD cheap)

### Pricing
- On Demand
- Spot (up-to 90% discount, can be interrupted with two minutes of notification, e.g. test, analytics, stateless app, image rendering, etc)
- Reserved (you can pay upfront to get more discount)
- Dedicated Instances
- Dedicated Hosts (physical server)

### Security Group
- stateful


### EBS

### EFS

### Load Balancer

### Lambda (out of scope in 2018)

### CLI Commands
- [AWS CLI Command](https://docs.aws.amazon.com/cli/latest/index.html)
- describe-instances
- describe-image
- start-instances (start)
- run-instances (create)


##  2. S3 ❗️

[S3 FAQ](https://aws.amazon.com/s3/faqs/)

## 3. DynamoDB ❗️

[DynamoDB FAQ](https://aws.amazon.com/dynamodb/faqs/)

### Partition Key + Sort Key
- partition key, aka. hash key / sort key aka. range key
- partition key + sort key must be unique (sort key is optional)
- e.g. partition key = unique user id and sort key = time stamp


### Index

### Query vs Scan



## 4. SQS

> can be delivered in multiple times and in any order. **NOT FIFO**

- the **1st** service in AWS
- largest message can be **256kb**
- default visibility timeout: **30s**, max/**12hr**, can be extended by `ChangeMessageVisibility`
- short polling (wait time = 0)
- long polling (add a message interval)
  - wait until a message is available until timeout
  - max wait time **20s**
- messages -> SNS -> all subscribed  SQS queues

## 5. SNS


## 6. SWF

## 7. Beanstalk

## 8. CloudFormation

## 9. Shared Responsibility

## 10. Route53

## 11. DNS

## 12. VPC

## References

- http://awstrainingandcertification.s3.amazonaws.com/production/AWS_certified_developer_associate_blueprint.pdf
- https://aws.amazon.com/faqs/
- https://docs.aws.amazon.com/cli/latest/index.html
- https://aws.amazon.com/ec2/instance-types
- https://acloud.guru

