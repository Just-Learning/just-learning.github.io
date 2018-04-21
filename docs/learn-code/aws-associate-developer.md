
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
- Blocked Storage > virtual disk

Volumes | Snapshots
--------|----------
EBS, virtual disk | S3
take a snapshot of a volume | snapshot is a point-time copy of volume
block storage | incremental



### EFS

### Load Balancer

- alias records
- can't resolved by A records

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

- store in SSD
- replicates data across three facilities in an AWS Region

> pricing by **read** throughput and **write** throughput

### Primary Keys

- Partition Key: hash key (like the unique key in relational database)
- Sort Ke: range key

Single primary key (**partition key**)
- partition key (unique)

Composite primary key (**Partition Key + Sort Key**)
- partition key + sort key (unique)
- 2 items cam have same partition key but must have a different sort key
- all items w same partition key store in one partition, and sorted by sort key
- e.g. partition key = unique user id and sort key = time stamp

### Secondary Indexes

local secondary index (LSI)
- same partition key + different sort key
- can be create at table creation

global secondary index (GSI)
- different partition key + different sort key
- can be create at table creation or LATER


### Streams

capture modifications of DynamoDB
- new: entire item
- update: before and after
- delete: entire deleted item
- data stored for **24hr**

### Query vs Scan

### Provisioned Throughput

Read Capacity Units (RCU): (round up to 4K)
- Strongly Consistent Reads: 1/s
- Eventually Consistent Reads: 2/s

Write Capacity Units (WCU):


Strong Consistency vs Eventual Consistency


### Index

### Query vs Scan

## 4. SQS

!> can be delivered in multiple times and in any order. **NOT FIFO**

- **PULL**
- the **1st** service in AWS
- largest message can be **256kb**
- default visibility timeout: **30s**, max/**12hr**, can be extended by `ChangeMessageVisibility`
- short polling (wait time = 0)
- long polling (add a message interval)
  - wait until a message is available until timeout
  - max wait time **20s**
- messages -> SNS -> all subscribed  SQS queues

## 5. SNS

- **PUSH**
- `publish/subscribe`, publish message and immediately deliver to subscribers
- subscriber needs confirm the subscription
- to SMS, email, http/s endpoint, SQS, App
- **Topic**
- pay as you go
- send email json: TopicArn,
- TTL (undelivered messages will expire)

SQS vs SNS
- both messages services
- SNS is PUSH
- SQS is PULL

> message can be customized for each protocol

## 6. SWF

- Workers to get task, process, return result
- Decider to control the coordination of tasks
- Domain as container
- Max workflow can be **1yr**

!> SWF maintains the execution state. It ensures a task is **assigned only once** and is **never duplicated**

SWF vs SQS
- SWF: task-oriented vs  SQS: message-oriented
- SWF: task assigned only once and never duplicated vs message delivered in multiple times and in any order
- SWF: track execution state vs SQS doesn't

## 7. Beanstalk

- multiple application versions (zip)
- split in to tiers (web/app/db)
- update app or config: 1 instance / % instance / immutable
- **free** but pay for the resources it uses
- Beanstalk manages RDS instance it created
- supports
  - Tomcat for Java
  - HTTP Server for PHP
  - Nginx or HTTP Server for Node.js
  - IIS for .Net
  - Java SE
  - Docker
  - Go

## 8. CloudFormation

- *automatic rollback on error*, charge for errors
- *WaitCondition* wait for resources to be provisioned
- *FN::GetAtt* to output data
- *R53* supported, hosted zones / records creation
- *IAM* role creation
- default format is *json* / template in json
- **free** but pay for the resources it provisions
- error occurs: rollback all resources created onn failure

## 9. Shared Responsibility

Infrastructure Services

- aws
  - foundation: compute, storage, db, networking
  - infrastructure: region, availability zone, edge location
  - e.g. hyper-v: aws patch and reboot
- customers
  - IAM
  - customer data, platform, app management, os, network, firewall config
  - encryption: client side / server-side, network traffic protection: http/https

Container Services
- aws takes care of os, platform and app management, patch os
- e.g. RDS, EMR

Abstracted Services
- customer data, client-side encryption
- e.g. DynamoDB, Lambda


## 10. Route53 ❗️

> R53 is global like IAM

Simple Policies
- example.com / the DNS name of the ELB (alias)
- 50 50 switch between EC2 instances

Weighted
- split traffic to multiple resources

Latency
- route based on latency (cross region)
- route traffic to the resource that provides the best latency.

Failover
- health check the London ELB endpoint
- 2 records: primary and secondary,

Geolocation
- route based on geo location (cross region)
- shift traffic from resources in one location to resources in another.

## 11. DNS

search a public ip by a  domain name

top Level Domains: .com .gov .edu, second level domain name: .com.au, .edu.cn

- SOA Records ()
- NS Records (**name** server by top level domain servers)
- A Records (**domain name / ip address**)
- TTL (cache, time to live)
- CNAMES (canonical **domain name / domain name**)
- Alias Records (AWS created, easy way to map naked domain name (apex) to **resource record / ELB, CF distribution, S3 bucket**)


Tips:
- ELB do not have pre-defined IPv4 addresses, to resolve using a DNS name
- Alias vs CNAME
- choose Alias over CNAME

## 12. VPC ❗️

## References

- https://aws.amazon.com/certification/certified-developer-associate/
- http://awstrainingandcertification.s3.amazonaws.com/production/AWS_certified_developer_associate_blueprint.pdf
- https://aws.amazon.com/whitepapers/aws-security-best-practices/
- https://aws.amazon.com/faqs/
- https://docs.aws.amazon.com/cli/latest/index.html
- https://aws.amazon.com/ec2/instance-types
- https://acloud.guru

