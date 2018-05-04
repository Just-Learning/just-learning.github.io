
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

## IAM

!> IAM is global

IAM contains **users**, **groups** (group users and assign policies), **roles**, **policies** in json

IAM
- centralized control of your AWS account
- integrates with existing AD account allowing single sign on (**SSO**)
  - SSO on ADFS `https://signin.aws.amazon.com/saml`
  - get SAML from AD (Security Assertion Markup Language)
  - `AssumeRoleWithSAML`
- integrated with social media account allowing temporary access (**Web Identity Federation**)
  - authenticates with facebook
  - get ID token
  - `AssumeRoleWithWebIdentity`
- fine-grained access control to AWS resources

Tips:
- `root account` first setup the aws account, full admin access
- should login as a user
- user assigned **Access Key ID** and **Secret Access Keys** (for AWS CLI or API)
- always setup multifactor on your root account
- password rotation policies

Roles

- you can assign rules to an EC2 instance AFTER it has been provisioned.
- roles are easier to manage than user's Access Key ID and Secret Access Key
- roles are **global**


## EC2

?> [EC2 FAQ](https://aws.amazon.com/ec2/faqs/)

### Types

[EC2 Instance Types](https://aws.amazon.com/ec2/instance-types/)

\ | Type | Specialty | Use case
-----|------|-----------|----------
General | T | **lowest** cost | web servers/small DBs
        | M | general purpose | App servers
CPU | C | compute optimized | CPU intensive Apps/DBs
GPU | P | **GPU compute** | machine learning, bit coin mining
    | G | **graphics** intensive | video encoding, 3D app
    | F | field programmable gate array | hardware acceleration
Memory | X | RAM **extreme** optimized | SAP HANA/Apache Spark
       | R | RAM optimized | memory intensive Apps/DBs
Storage | H | **HHD** high throughput |
        | I | **SSD** IOPS |
        | D | dense storage | file server, data warehouse, hadoop

### Pricing

Type | Use case
-----|---------
On Demand | spike access on Friday and go down
Spot | can be interrupted, for testing, analytics
Reserved | very stable and fix web app, pay upfront to get more discount
Dedicated Instances |  fully owned EC2 instances
Dedicated Hosts | physical server

?> If you terminate the the spot instance, you pay for the hour. If AWS terminates it, you get the hour for free.

### EC2 Tips
- termination protection is turned off by default
- on EBS-backed instance, root EBS volume to be deleted when instance is terminated by default
- support for encrypted boot volumes
- get instance info, e.g. public ip `curl http://169.254.169.254/latest/meta-data/`

### Security Group
- stateful


### EBS

Blocked Storage, like virtual disk
!> cannot mount 1 EBS volume on multiple EC2 instances, instead use EFS

- SSD, General Purpose
- SSD, Provisioned IOPS
- HDD, Throughout Optimized, **frequent access**, data warehousing, logging
- HDD, Code, **less frequent access**, file servers
- HDD, Magnetic, cheap

Volumes | Snapshots
--------|----------
EBS, virtual disk | S3
take a snapshot of a volume | snapshot is a point-time copy of volume
block storage | incremental
encrypted >>> | <<< encrypted

Cross-Account Copying: You can share the encrypted EBS snapshot

?> To create a snapshot for an Amazon EBS volume that serves as a root device, you should stop the instance before taking the snapshot.

[Instance Store vs EBS](https://aws.amazon.com/premiumsupport/knowledge-center/instance-store-vs-ebs/)

Instance store-backed instances | Amazon EBS-backed instances
--------------------------------|-----------------------------
The root device is temporary. If you stop the instance, the data on the root device vanishes and cannot be recovered. | The root device is an Amazon EBS volume. If you stop the instance, the Amazon EBS volume persists. If you restart the instance, the volume is automatically remounted, restoring the instance state and any stored data. You can also mount the volume on a different instance.

[Take snapshot of a RAID Array](https://aws.amazon.com/premiumsupport/knowledge-center/snapshot-ebs-raid-array/)


To create an "application-consistent" snapshot of your RAID array, stop applications from writing to the RAID array, and flush all caches to disk. Then ensure that the associated EC2 instance is no longer writing to the RAID array by taking steps such as **freezing** the file system, **unmounting** the RAID array, or **shutting down** the associated EC2 instance.

### AMI (Amazon Machine Images)

**regional** must copy to other regions then lunch instance in that region.


### CloudWatch

- standard monitoring **5m**
- detailed monitoring **1m**
- dashboards for visual
- alarms for notification

?> CloudWatch is performance monitoring whereas CloudTrail is for auditing

### EFS

- Network File System (NFS)
- pay as you use, no provisioning required
- scale up PB
- K of concurrent NFS connections
- across multiple AZs with in a region
- **Read After Write** consistency

### Load Balancer ($chargeable$)

- Application vs Classic
- Health Check: timeout, interval, threshold
- InService / OutService
- ELB uses **DNS name** because public IP address might change
- EC2 instance can get public IP
- SO in R53, must use **alias** records cause can't resolved by **A** records

?> Application Load Balancers now support multiple SSL certificates and Smart Certificate Selection using Server Name Indication (SNI)

### Lambda and Serverless

- event driven compute service
- response on HTTP with API Gateway

### CLI Commands
- [AWS CLI Command](https://docs.aws.amazon.com/cli/latest/index.html)
- describe-instances
- describe-image
- start-instances (start)
- run-instances (create)

### SDK

Available SDK
- Android, iOS, Browser
- Java
- .Net
- Node.js
- PHP
- Python
- Go
- C++

Default Regions = US-EAST-1

## S3

?> [S3 FAQ](https://aws.amazon.com/s3/faqs/)

S3 Types
- S3: immediately available, frequent access
- S3 IA: infrequent access
- S3 RRS: reduced redundancy storage, reproducible data, e.g. thumb nails
- Glacier: archived data

Features
- **Unlimited** object-based storage whereas EBS is block-based, EFS is Network File System, you can install OS or Apps on S3
- file size **0 ~ 5TB**
- largest file using PUT  **5G**
- Bucket name must be **globally unique** and lower-case
- bucket arn: arn:aws:s3:::bucketname
- static web hosting: http://**bucketname-website**.s3-website-ap-southeast-2.amazonaws.com
-  bucket url: http://s3-**regionname**.amazonaws.com/**bucketname**
- multi part upload (**stop and restart**)
- HTTP 200 for a successful write
- **pre-signed** URL with expiration date and time to download private data,

Consistency
- **Read After Write Consistency** for CREATE an object
- **Eventual Consistency** for UPDATE and DELETE

Object contains *Key, Value, Version ID, Metadata, Access Control List*

Versioning
- all version including delete an object
- charge on each version
- easy to backup
- can be enabled, suspended, can't be disabled
- lifecycle rules
- MFA on DELETE
- cross region replication, requires versioning enabled on the source bucket

Lifecycle Management, work with current version and previous version
- scope: whole or prefix
- transition: after X days transfer previous version to IA/Glacier
- expiration: permanently delete after X days or clean up rules

Bucket Security
- by default PRIVATE
- modify access permission via Access Control List, Bucket Policy, CORS Config
- bucket access log

CORS
- Cross Origin Resource Sharing, e.g. web in one bucket, image in another bucket
- enable CORS on the bucket where the assets are stored. in **xml** format

Encryption
- In Transit: SSL/TLS
- Server Side Encryption
  - S3 Managed Keys: **SSE-S3** (AES-256)
  - AWS Key Management Service, **SSE-KMS**
  - with customer provided keys: **SSE-C**
- Client Side Encryption: encrypt before uploading to S3

Storage Gateway
- File Gateway: store flat files directly on  S3
- Volume Gateway
  - In cached mode, store your primary data in **S3** and retain your frequently accessed data locally in **cache**
  - In stored mode, store your entire data set locally, while making an asynchronous S3 and point-in-time EBS snapshots.
- Tape Gateway (VTL): for backup

Snowball, Snowball Edge (with compute in the box), Snowmobile (truck) - can **Import** to S3 and **Export** from S3

S3 Transfer Acceleration: use CloudFront’s globally distributed edge locations, data -> Edge Location -> S3 Bucket

S3 Static Website: cheap, scales automatically, **static** site only

### CloudFront

- Edge Location where content will be cached, separate to Region/AZ, **Read**, **Write**
- Origin: the origin of the files, **S3 Bucket, EC2, LB, R53**
- Distribution: can have **multiple** origin. Type: Web for websites, RTMP for video streaming
- TTL **24hr** by default
- invalidation: refresh all your cached data

## DynamoDB

### General

?> [DynamoDB FAQ](https://aws.amazon.com/dynamodb/faqs/) **MUST READ**

- fully managed, can't SSH or RDP
- store in **SSD**
- replicates data across 3 geographically distributed replicas
- **schema-less**
- secondary index on **top-level** json element only
- query / scan nest json object `ProjectionExpression`
- can update **sub-element** of a nest json data
- **no storage limitation and no throughput limitation**
- **400KB** limitation per item
- seamless scalability: automatically scales up and scales down
- Auto Scaling Policy (**free**): based on CloudWatch, can only be set to a **single table** or a global secondary indexes within a **single region**

> pricing by **read** throughput and **write** throughput

### Consistency model of READ

Eventual Consistency (default)
- consistency across all copies within **1s**
- best read performance

Strong Consistency

!> **1 strongly consistent read or 2 eventual consistent read  requires 1 read capacity unit**

Terms
- Table
- Item
- Attribute


### Primary Keys

- Partition Key: hash key (like the unique key in relational database)
- Sort Ke: range key

Single-attribute **partition key**
- partition key (unique)

Composite **partition-sort Key**
- partition key + sort key (unique)
- 2 items cam have same partition key but must have a different sort key
- all items w same partition key store in one partition, and sorted by sort key
- e.g. partition key = unique (user id and time stamp)

### Secondary Indexes

- independent throughput management
- a table CRUD, related GSI consume capacity unit
- a query on GSI consume the read capacity unit
- charge by hour - provision throughput, data storage, external transfer
- query/scan - no order or sort by sort key
- GSI is available after index creation process
- reduce the write capacity once index creation process is complete (mins to hours, SNS notification, cannot be cancelled)
- `DescribeTable`


global secondary index (GSI) - eventual consistent
- **different** partition key + **different** sort key
- create at table creation or **LATER**
- a GSI key can be defined on **non-unique** attributes
- **max 5 GSI**

local secondary index (LSI) - strong consistent
- in one partition, has to be created at table creation
- **same** partition key + **different** sort key

> To create one or more local secondary indexes on a table, use the LocalSecondaryIndexes parameter of the CreateTable operation. Local secondary indexes on a table are created when the table is created. When you delete a table, any local secondary indexes on that table are also deleted. **You can only create one secondary index at a time.**

### Streams

capture modifications of DynamoDB
- new: entire item
- update: before and after
- delete: entire deleted item
- data stored for **24hr**

### Query vs Scan

Query | Scan
------|------
using partition key | scan all items or index
fast | slow
x |  **1MB** `LastEvaluatedKey`
x | a scan with `ConsistentRead` consumes twice the read capacity as a scan with eventual consistent read


### Provisioned Throughput

**Read Capacity Units (RCU)**: (round up to 4K)
- Strongly Consistent Reads: 1/s
- Eventually Consistent Reads: 2/s

?> Q: 5 items of 10 KB per second using eventual consistency, calculate the read throughput?
1. how many unit per item? 10KB round up to 4KB / 4KB = 3 unit /s
2. 5 * 3 = 15 unit /s
3. read throughput
  - eventual consistency ceil(15 / 2) = 8
  - strong consistency 15

?> Q: read 80 items per second from a table. The items are 3 KB in size, strong consistency
1. 80 * 1 = 80 units
2. 80 units for strong consistency
3. 40 units for eventual consistency

**Write Capacity Units (WCU)**: (round up to 1KB)
- 1/s

?> Q: write 100 items per second to your table, and that the items are 512 bytes in size
1. each write requires one provisioned write capacity unit.
2. 100 * 1KB = 100 units

?> Q: write 3 items per second to your table, and that the items are 3KB bytes in size
1. 3 * 3 = 9 units

**Capacity Unit**

- The UpdateTable API call does not use capacity. It is used to change provisioned throughput capacity.



`ProvisionedThroughputExceededException` HTTP error 400

Authenticate
1. authenticates with ID provider (fb, g)
2. pass a token by ID provider
3. your app call `AssumeRoleWithWebIdentity` with token, arn for the IAM role
4. your app can now access DynamoDB **15m ~ 1hr (default)**

Conditional Write
- by default (PutItem, UpdateItem, DeleteItem) are unconditional
- concurrency safe
- idempotent (幂等):send same conditional write request multiple times w/o further effect
- e.g. conditional expression: `ATTRIBUTE_EXIST CONTAINS BEGIN_WITH` `=, <>, <, >,<=, >=, BETWEEN, IN` `NOT AND OR`

Atomic Increment and Decrement
- global incremental counter for visits
- concurrency safe

Batch Operations
- `BatchGetItem` from one or more tables
- 16 MB of data, which can contain as many as 100 items
- `ValidationException` if more than 100 items

Common usage: write: `PutItem` `BatchWriteItem` read: `GetItem` `BatchGetItem`

### DynamoDB vs Other

DynamoDB | RDS
---------|----
key-value and document | relational DB (multiple engines)
weak query | complex query
fast | x
predictable performance | x


DynamoDB | SimpleDB
---------|----
No SQL | No SQL
no limit | 10 GB
scalability | scaling limitations
large | smaller workload

DynamoDB | S3
---------|----
1 byte ~ 400 KB | up to 5TB
small and fast | large and infrequently access


## SQS

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
- e.g. exceed maximum allowable size, send a reference to a S3 object
- `MessageRetentionPeriod` attribute to set the message retention period from 60 seconds (**1m**) to 1,209,600 (**14d**). The default is **4d**

## SNS

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

## SWF

- Workers to get task, process, return result
- Decider to control the coordination of tasks
- Domain as container
- Max workflow can be **1yr**

!> SWF maintains the execution state. It ensures a task is **assigned only once** and is **never duplicated**

SWF vs SQS
- SWF: task-oriented vs  SQS: message-oriented
- SWF: task assigned only once and never duplicated vs message delivered in multiple times and in any order
- SWF: track execution state vs SQS doesn't

## Beanstalk

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

## CloudFormation

?> turn infrastructure into configuration

- *automatic rollback on error*, charge for errors
- *WaitCondition* wait for resources to be provisioned
- use `Fn::GetAtt` to output data
- *R53* supported, hosted zones / records creation
- *IAM* role creation
- default format is *json* / template in json
- **free** but pay for the resources it provisions, **full root access**
- error occurs: rollback all resources created on failure
- can't nested templates

### Template

-  architecture infrastructure diagram
- `json` or `yaml`

> `Recourses` List of recourses and associated configuration values, **Mandatory**

```yml
AWSTemplateFormatVersion: 2010-09-09
Parameters: # input values - name, password, key name, etc,  max 60
Mappings: # instance type -> arch, arch -> AMI
Resources: # resources to be created
Outputs: # output values, e.g. PublicIp, ELB address, can use `Fn::GetAtt`
```

### Stack
- the resources that created

##  Shared Responsibility

#### Infrastructure Services

- aws
  - foundation: compute, storage, db, networking
  - infrastructure: region, availability zone, edge location
  - e.g. hyper-v: aws patch and reboot
- customers
  - IAM
  - customer data, platform, app management, os, network, firewall config
  - encryption: client side / server-side, network traffic protection: http/https

#### Container Services
- aws takes care of os, platform and app management, patch os
- e.g. RDS, EMR

#### Abstracted Services
- customer data, client-side encryption
- e.g. DynamoDB, Lambda


##  Route53

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

## DNS

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

## VPC

## Memorize Matrix

## References

- https://aws.amazon.com/certification/certified-developer-associate/
- http://awstrainingandcertification.s3.amazonaws.com/production/AWS_certified_developer_associate_blueprint.pdf
- https://aws.amazon.com/whitepapers/aws-security-best-practices/
- https://aws.amazon.com/blogs/aws/new-cross-account-copying-of-encrypted-ebs-snapshots/
- https://aws.amazon.com/about-aws/whats-new/2015/12/amazon-ebs-announces-support-for-encrypted-boot-volumes/
- https://aws.amazon.com/faqs/
- https://docs.aws.amazon.com/cli/latest/index.html
- https://aws.amazon.com/ec2/instance-types
- https://acloud.guru

