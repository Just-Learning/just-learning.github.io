
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

IAM contains **users**, **groups**,  **roles**, **policies** in json
- centralized control of your AWS account
- fine-grained access control to AWS resources
- group: group users and assign policies), no nested groups
- New users have NO permissions when first created, can be assigned **Access Key ID** and **Secret Access Keys** (for AWS CLI or API)

Tips:
- `root account` first setup the aws account, full admin access
- should login as a user
- Use Mutli-Factor Authentication (MFA)
- Rotate credentials regularly
- Grant least privilege
- **CloudTrail** helps us track AWS API calls and transitions, **AWS Config** helps to understand what resources we have now, and **IAM Credential Reports** allows auditing credentials and logins.

Roles
- you can assign roles to an EC2 instance AFTER it has been provisioned.
- best practice: attach an IAM role with xxx access to the EC2 instance
- Always launch EC2 instance with IAM role instead of hardcoded credentials
- you CANNOT add an IAM role to an IAM group
- roles are easier to manage than user's Access Key ID and Secret Access Key
- roles are **global**

Temporary Security Credentials
- via Security Token Service (AWS STS)
- generated dynamically and provided to the user when requested
- **SAML**, integrates with existing **AD** account allowing single sign on (**SSO**), steps:
  - SSO on ADFS `https://signin.aws.amazon.com/saml`
  - get SAML from AD (Security Assertion Markup Language)
  - `AssumeRoleWithSAML`
- **Web Identity Federation**, integrated with **social media account** allowing temporary access, steps:
  - authenticates with facebook
  - get ID token
  - `AssumeRoleWithWebIdentity`
- e.g. App on EC2 to accessing object stored in S3
  - An IAM **trust policy** allows the EC2 instance to **assume** an EC2 instance role.
  - An IAM **access policy** allows the EC2 role to access S3 objects

## EC2

?> [EC2 FAQ](https://aws.amazon.com/ec2/faqs/)

### Types

[EC2 Instance Types](https://aws.amazon.com/ec2/instance-types/)

| \       | Type                          | Specialty                           | Use case                          |
| ------- | ----------------------------- | ----------------------------------- | --------------------------------- |
| General | T                             | **lowest** cost                     | web servers/small DBs             |
| M       | general purpose               | App servers                         |
| CPU     | C                             | compute optimized                   | CPU intensive Apps/DBs            |
| GPU     | P                             | **GPU compute**                     | machine learning, bit coin mining |
| G       | **graphics** intensive        | video encoding, 3D app              |
| F       | field programmable gate array | hardware acceleration               |
| Memory  | X                             | RAM **extreme** optimized           | SAP HANA/Apache Spark             |
| R       | RAM optimized                 | memory intensive Apps/DBs           |
| Storage | H                             | **HHD** high throughput             |
| I       | **SSD** IOPS                  |
| D       | dense storage                 | file server, data warehouse, hadoop |

### Pricing

| Type                | Use case                                                      |
| ------------------- | ------------------------------------------------------------- |
| On Demand           | spike access on Friday and go down                            |
| Spot                | can be interrupted, for testing, analytics                    |
| Reserved            | very stable and fix web app, pay upfront to get more discount |
| Dedicated Instances | fully owned EC2 instances                                     |
| Dedicated Hosts     | physical server                                               |

?> If you terminate the the spot instance, you pay for the hour. If AWS terminates it, you get the hour for free.

- When an EC2 instance is terminated, we do charge for the storage for any Amazon EBS volumes
- If I have two instances in different availability zones, data transfer is charged at *Data Transfer Out from EC2 to Another AWS Region*  for the first instance and at *Data Transfer In from Another AWS Region* for the second instance

### EC2 Tips
- termination protection is turned off by default
- on EBS-backed instance, root EBS volume to be deleted when instance is terminated by default
- support for encrypted boot volumes
- get instance info, e.g. public ip `curl http://169.254.169.254/latest/meta-data/`
- RDP on Windows TCP port 3389
- Elastic IP address is a static IPv4
- For Linux, no password, use key pair to ssh log in. For Windows, use key pair to obtain the administrator password and then log in using RDP
- if the private key is lost, there is no way to recover the same. have to generate a new one

> To view all categories of instance metadata from within a running instance, use the following URI: http://169.254.169.254/latest/meta-data/

### Security Group
- stateful, if you create an inbound rule allowing traffic in, that traffic is automatically allowed back out again
- You cannot block specific IP addresses using Security Groups, instead use Network Access Control lists
- You can allow rules, but cannot deny rules
- You must assign each server to at least 1 security group

### EBS

Blocked Storage, like virtual disk, `AttachVolume ` to attach EBS volume to EC2 instance

!> cannot mount 1 EBS volume on multiple EC2 instances, instead use EFS

>  you can now encrypt all your EBS storage, including boot volume

- SSD, General Purpose
- SSD, Provisioned IOPS
- HDD, Throughout Optimized, **frequent access**, data warehousing, logging
- HDD, Code, **less frequent access**, file servers
- HDD, Magnetic, cheap

| Volumes                     | Snapshots                               |
| --------------------------- | --------------------------------------- |
| EBS, virtual disk           | S3                                      |
| take a snapshot of a volume | snapshot is a point-time copy of volume |
| block storage               | incremental                             |
| encrypted >>>               | <<< encrypted                           |

Cross-Account Copying: You can share the encrypted EBS snapshot

?> To create a snapshot for an Amazon EBS volume that serves as a root device, you should stop the instance before taking the snapshot.

[Instance Store vs EBS](https://aws.amazon.com/premiumsupport/knowledge-center/instance-store-vs-ebs/)

| Instance-backed instances                                                                                             | EBS-backed instances                                                                                                                                                                                                                                                            |
| --------------------------------------------------------------------------------------------------------------------- | ------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| The root device is temporary. If you stop the instance, the data on the root device vanishes and cannot be recovered. | The root device is an Amazon EBS volume. If you stop the instance, the Amazon EBS volume persists. If you restart the instance, the volume is automatically remounted, restoring the instance state and any stored data. You can also mount the volume on a different instance. |

!> EBS-backed instances can be stopped and restarted without losing the data on the volume. In other word, you can't restart a instance-backed instance.

[Take snapshot of a RAID Array](https://aws.amazon.com/premiumsupport/knowledge-center/snapshot-ebs-raid-array/)

To create an "application-consistent" snapshot of your RAID array, stop applications from writing to the RAID array, and flush all caches to disk. Then ensure that the associated EC2 instance is no longer writing to the RAID array by taking steps such as **freezing** the file system, **unmounting** the RAID array, or **shutting down** the associated EC2 instance.


[Changing the Encryption State of Your Data](https://docs.aws.amazon.com/AWSEC2/latest/UserGuide/EBSEncryption.html)

There is no direct way to encrypt an existing unencrypted volume, or to remove encryption from an encrypted volume. However, you can migrate data between encrypted and unencrypted volumes. You can also apply a new encryption status while copying a snapshot:
- While copying an unencrypted snapshot of an unencrypted volume, you can encrypt the copy. Volumes restored from this encrypted copy are also encrypted.
- While copying an encrypted snapshot of an encrypted volume, you can associate the copy with a different CMK. Volumes restored from the encrypted copy are only accessible using the newly applied CMK.

You cannot remove encryption from an encrypted snapshot.

### AMI (Amazon Machine Images)

**regional** must copy to other regions then lunch instance in that region.

`DescribeImages` will list the AMIs available in the current region.

### CloudWatch

- standard monitoring **5m**
- detailed monitoring **1m**
- dashboards for visual
- alarms for notification
- events of state changes

?> **CloudWatch** is performance monitoring whereas **CloudTrail** is for auditing

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
- ELB's IP address keeps changing. You should instead use the DNS name provided to you.

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

### http status code

1. 1xx Informational responses
2. 2xx Success
3. 3xx Redirection
4. 4xx Client errors
5. 5xx Server errors

## Auto Scaling

> **configure** automatic scaling for the scalable AWS resources.
> create one scaling plan per application source (a CloudFormation stack or a set of tags).

Scalable resources
- Aurora DB clusters
- Auto Scaling groups
- DynamoDB global secondary indexes
- DynamoDB tables
- ECS services
- Spot Fleet requests

### What is EC2 Auto Scaling
- **Auto Scaling groups**: create collections of EC2 instances
- **Launch configuration** as a template for its EC2 instances: AMI, keypair, security group, etc. any new instances launched use the new configuration parameters, but the existing instances are not affected.
- **Auto Scaling Plan**: minimum/desired/maximum capacity
- Availability Zones or Subnets
- Metrics & Health Checks
- Auto Scaling can **launch or terminate** instances as demand on your application increases or decreases.

![EC2 Autl Scaling](http://tinyimg.io/i/C7ULNnU.png)

### Auto Scaling + Load Balancer

As the Auto Scaling group adds and removes EC2 instances, you must ensure that the traffic for your application is distributed across all of your EC2 instances. The Elastic Load Balancing service automatically routes incoming web traffic across such a dynamically changing number of EC2 instances. Your load balancer acts as a single point of contact for all incoming traffic to the instances in your Auto Scaling group

Instance Status in Load Balancer
- `Adding` -> register all instances to Load Balance -> `Added`
- `InService` at least one registered instance passes the **health checks**
- Auto Scaling can **terminate and replace** any instances that are reported as **unhealthy**
- `Removing` state while **deregistering** the instances, **deregistered instances remain running. Manually delete it**
- `OutOfService` will trigger a replacement

What is an unhealthy instance?
- the instance is `NOT running`
- the system status is `impaired`
- ELB reports the instance as `OutOfService`

ELB health check
- ping instance and check the timeout
- interval, unhealthy threshold, healthy threshold

Manual scaling
- **Changing the desired capacity** limit of the Auto Scaling group
- **Attaching/Detaching instance**s to the Auto Scaling group

Scheduled scaling
- scale your application in response to predictable load changes, e.g. Black Friday, Financial year

Dynamic scaling
- scale out/in based on CPU usage, OR base on CloudWatch Alarms
- **CloudWatch** monitors the specified metrics
- metrics breaches the **threshold**
- CloudWatch sends to either the **scale-in** policy or the **scale-out** policy
- Auto Scaling performs the scaling activity

Scaling Cooldown
- waiting period until the newly created instance takes effect, default `300` seconds

Termination Policy
- Controlling Which Auto Scaling Instances Terminate During Scale In
- `OldestInstance`, `NewestInstance`, `OldestLaunchConfiguration`, `ClosestToNextInstanceHour`, `Default`

Instance Protection
- To control whether an Auto Scaling group can terminate a particular instance when scaling in
- `Protect From Scale In`

To terminate an instance in an Auto Scaling group
- This example terminates the specified instance from the specified Auto Scaling group without updating the size of the group:
- `aws autoscaling terminate-instance-in-auto-scaling-group --instance-id i-93633f9b --no-should-decrement-desired-capacity`
- Auto Scaling launches a replacement instance after the specified instance terminates.

`terminate-instance-in-auto-scaling-group`



### Limits
- Auto Scaling groups could be in **multi AZ but one region**
- default scaling cooldown period is `300 seconds`

## ELB

> allows the incoming traffic to be distributed automatically across multiple healthy EC2 instances.

- handle the client requests
- is itself a distributed system that is fault tolerant and actively monitored
- integration with Auto Scaling
- **scale up or scale out**
- works best with **gradual increase** in traffic
- are engineered to not be a single point of failure
- work across AZs within a region
- ELB is scaled automatically handled by AWS
- Only one subnet per AZ can be attached to the ELB - client facing ELB should be attached to a `Public Subnet`

### Type

Application Load Balancer
- at the request level (layer 7), routing traffic to targets - EC2 instances, containers and IP addresses based on the content of the request.
- You can load balance HTTP/HTTPS applications and use layer 7-specific features, such as X-Forwarded-For headers.
- request level

Network Load Balancer
- at the connection level (Layer 4), routing connections to targets - Amazon EC2 instances, containers and IP addresses based on IP protocol data.
- connection level

Classic Load Balancer
- basic load balancing across multiple Amazon EC2 instances and operates at both the request level and connection level
- both request and connection level

### Cross Zone

**w/o Cross-Zone**: By default, the load balancer distributes incoming requests evenly across its enabled Availability Zones for e.g. If AZ-a has 5 instances and AZ-b has 2 instances, the load will still be distributed 50% across each of the AZs

![w/o Cross-Zone](http://tinyimg.io/i/xgD36Hp.png)

**w/ Cross-Zone**: Enabling Cross-Zone load balancing allows the ELB to distribute incoming requests evenly across all the back-end instances, regardless of the AZ

![w/ Cross-Zone](http://tinyimg.io/i/QXOuf9u.png)


### Health Checks
- Load balancer performs health checks on all registered instances, whether the instance is in a healthy state or an unhealthy state.
- Health check is `InService` for status of healthy instances and `OutOfService` for unhealthy ones
- Load balancer only sends requests to the healthy EC2 instances and stops routing requests to the unhealthy instances

## S3

?> [S3 FAQ](https://aws.amazon.com/s3/faqs/)

### S3 Types
- S3: immediately available, frequent access
- S3 IA: infrequent access, within 30 days will be charged for a full **30 days**
- S3 RRS: reduced redundancy storage, reproducible data, e.g. thumb nails
- Glacier: archived data, have a minimum of **90 day**s of storage, and objects deleted before 90 days incur a pro-rated charge equal to the storage charge for the remaining.

### Features
- **Unlimited** object-based storage whereas EBS is block-based, EFS is Network File System
- Object contains *Key(name), Value (data), Version ID, Metadata, Access Control List*
- bucket arn `arn:arn:aws:s3:::bucketname`
- bucket url: `http://s3-regionname.amazonaws.com/bucketname`
- static web hosting: `http://bucketname.s3-website-ap-southeast-2.amazonaws.com`, cheap, scales automatically, **static** site only
- S3 used to store data in alphabetical order
- HTTP 400 for `MissingSecurityHeader`, `IncompleteBody`, `InvalidBucketName`, `InvalidDigest`
- HTTP 404 for `NoSuchBucketPolicy`
- HTTP 409 for conflict issue
- HTTP 200 for a successful write
- **pre-signed** URL with expiration date and time to download private data using SDK in code
- `x-amz-delete-marker`, `x-amz-id-2`, and `x-amz-request-id` are all common S3 response headers

### Limits
- single file could be **0 bytes to a maximum of 5TB**
- **a single PUT is 5GB**
- **Multipart Upload** preferred if > 100MB, must if > 5GB
- **globally unique**,  **lower-case**, 3 and 63 characters long
- CANNOT install OS or Apps on S3
- max **100 S3 buckets** per account
-
### Price
- Storage – cost is per GB/month
- Requests – per request cost varies depending on the request type GET, PUT
- Data Transfer
  - data transfer in is free
  - data transfer out is charged per GB/month (except in the same region or to Amazon CloudFront)

### Consistency
- provides eventual consistency for read-after-write.
- **Read After Write Consistency** for CREATE an object
- **Eventual Consistency** for UPDATE and DELETE

###  Management

Lifecycle
- work with current version and previous version
- scope: whole or prefix
- transition: after X days transfer previous version to IA/Glacier
- expiration: permanently delete after X days or clean up rules

Bucket Policy
- by default PRIVATE, contained objects are also private
- modify access permission via Access Control List, Bucket Policy (json), CORS Config (xml)
- bucket access log

CORS Policy
- Cross Origin Resource Sharing, e.g. web in one bucket, image in another bucket
- enable CORS on the bucket where the assets are stored. in **xml** format

Replication
- Cross-region Replication by default copies only updated or newly added objects. To copy existing content, we use command line:

### Properties

Versioning
- stores all versions including delete an object
- charge on each version
- easy to backup
- can be enabled, suspended, can't be disabled
- lifecycle rules
- MFA on DELETE
- cross region replication, requires versioning enabled on the source bucket

Encryption
- In Transit: SSL/TLS
- Server Side Encryption
  - S3 Managed Keys: **SSE-S3** (AES-256), HTTP header `x-amz-server-side-encryption`
  - AWS Key Management Service, **SSE-KMS**
  - with customer provided keys: **SSE-C**
- Client Side Encryption: encrypt before uploading to S3

Multi part upload
- **stop and restart**
-  **recommended for files greater than 100MB, and is required for files larger than 5GB**
- The largest object that can be uploaded in a single PUT is **5GB**
- This operation completes a multipart upload by assembling the previously uploaded parts, and does not begin until invoked. The processing of a `CompleteMultipartUpload` request can take several minutes to complete, and your applications should be prepared to retry any failed requests.

Multi-Object Delete
- send multiple object keys in a single request to speed up your deletes

### Storage Gateway
- File Gateway: store flat files directly on  S3
- Volume Gateway
  - In cached mode, store your primary data in **S3** and retain your frequently accessed data locally in **cache**. `(cache partial data on local)`
  - In stored mode, store your entire data set locally, while making an asynchronous S3 and point-in-time EBS snapshots. `(full copy on local)`
- Tape Gateway (VTL): for backup

### Snowball
- Snowball
- Snowball Edge (with compute in the box)
- Snowmobile (truck)
- **Import** to S3 and **Export** from S3


### CloudFront

- Edge Location where content will be cached, separate to Region/AZ, **Read**, **Write**
- S3 Transfer Acceleration: use CloudFront’s globally distributed edge locations, data -> Edge Location -> S3 Bucket
- Origin: the origin of the files, **S3 Bucket, EC2, LB, R53**
- Distribution: can have **multiple** origin. Type: Web for websites, RTMP for video streaming
- TTL **24hr** by default
- invalidation: refresh all your cached data


## RDS
- Provisioned IOPS and DB instance
- Multi DB engines

### RDS vs EC2 hosted

RDS
- database as a service
- w/o installing software
- Amazon takes care of infrastructure, backups, and updates on the DB instance
- the shell access to the underlying operating system is disabled.
- log in as privilege user
- cut off some features
- **Multi-AZ deployments**

EC2 hosted
- full control of you DB
- various EC2 type to fit your requirement
- manage multi-AZ DB Cluster yourself

### RDS Multi-AZ (high availability)

!> Multi-AZ is a **High Availability** feature is not a scaling solution for read-only scenarios; standby replica can’t be used to serve read traffic. To service read-only traffic, use a Read Replica.

!> Multi-AZ Standby instance cannot be used as a scaling solution

- synchronous **standby replica**
- independent infrastructure in a physically separate location
- may **increase latency** because of sync data replication comparing to Single-AZ
- minimize **latency spikes** and **I/O freezes** during **system backups**

Creation
- modify a Single-AZ to a Multi-AZ
1. take snapshot
2. restore into another AZ
3. enable sync replication

Failover
- automatically **failover** support w/ **Multi-AZ**
  - automatically **switches to a standby replica in another AZ**
  - automatically **changes the DNS record** of the DB instance to point to the standby DB instance.
  - database operations can be resumed **ASAP** w/o administrative intervention **OUTAGE**
- Multi-AZ failover trigged by
  - **AZ outage**
  - **Primary DB instance fails**
  - DB instance’s **server type is changed**
  - OS of the DB instance is undergoing software **patching**
  - A manual **Forced Failover**

### Read Replica (scale out)

!> Read Replica is used for scale out

- create a **Read Replica** from a source DB instance
- data are **asynchronously** copied to the Read Replica
- **Read traffic** can be rout to Read Replica, scale out
- **uss case: read-heavy, data warehouse, serving while source DB n/a**
- only allow **read-only connections**
- secure communications
- a read replica can be modified to a new source DB, won't impact other replicas

Read Replica Creation
  1. enable **automatic backups**
  2. modify existing DB to be the **source**
  3. take snapshot
  4. create a **read-only** instance
  5. enable **async replication**
  6. **I/O suspension** on the source DB instance as the DB **snapshot** occurs, multiple RR created in parallel from the same source DB instance, only one snapshot is taken at the start of the first create action.
  7. **I/O suspension** cab be avoided if the source DB instance is a Multi-AZ deployment (in the case of Multi-AZ deployments, DB snapshots are taken from the **standby**).
- Delete replica explicitly, source DB deletion won't trigger **replica deleting**, replica will run as a standalone

### Limits

- Up tp `5` Read Replica per source DB
- Read Replica does not support **circular replication**
- Read Replica support **cross region replication** for some DB engine
- Read Replica chain < `4`

## DynamoDB

### General

?> [DynamoDB FAQ](https://aws.amazon.com/dynamodb/faqs/) **MUST READ**

- store in **SSD**, both **document** and **key-value** data models
- fully managed, can't SSH or RDP
- replicates data across **3** geographically distributed replicas
- **schema-less**
- secondary index on **top-level** json element only
- query / scan nest json object `ProjectionExpression`
- can update **sub-element** of a nest json data
- **no storage limitation and no throughput limitation**
- **400KB** limitation per item
- seamless scalability: automatically scales up and scales down
- Auto Scaling Policy (**free**): based on CloudWatch, can only be set to a **single table** or a global secondary indexes within a **single region**

### Pricing
- pricing by **read** throughput and **write** throughput
- storage

### Consistency model of READ

Eventual Consistency (default)
- consistency across all copies within **1s**
- best read performance

Strong Consistency

!> **1 strongly consistent read or 2 eventual consistent read  requires 1 read capacity unit**

Terms
- Table
- Item **1 byte ~ 400 KB**
- Attribute **NO LIMIT**

### Primary Keys

- Partition Key: hash key (like the unique key in relational database)
- Sort Key: range key

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

local secondary index (LSI) - eventually consistent or strongly consistent
- in one partition, has to be created at table creation
- **same** partition key + **different** sort key
- **max 5 LSI**

> To create one or more local secondary indexes on a table, use the LocalSecondaryIndexes parameter of the CreateTable operation. Local secondary indexes on a table are created when the table is created. When you delete a table, any local secondary indexes on that table are also deleted. **You can only create one secondary index at a time.**

### Streams

capture modifications of DynamoDB
- new: **entire item**
- update: **before and after**
- delete: **entire deleted item**
- data stored for **24hr**

### Query vs Scan

| Query               | Scan                                                                                                               |
| ------------------- | ------------------------------------------------------------------------------------------------------------------ |
| by partition key    | scan all items or index, use `ProjectionExpression` parameter so that the Scan only returns some of the attributes |
| fast                | slow                                                                                                               |
| x                   | up to **1MB** `LastEvaluatedKey`, **Paginating the Results**                                                       |
| x                   | a scan with `ConsistentRead` consumes twice the read capacity as a scan with eventual consistent read              |

> A Scan operation always scans the entire table, then filters out values to provide the desired result

### Provisioned Throughput

Auto Scaling or manually

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

!> DynamoDB distributes capacity **evenly across all available partition**s. If a given partition is consuming more than its share of throughput, `ProvisionedThroughputExceededException` will be raised, HTTP error 400

Authenticate
1. authenticates with ID provider (facebook, google)
2. pass a token by ID provider
3. your app call `AssumeRoleWithWebIdentity` with token, arn for the IAM role
4. your app can now access DynamoDB **15m ~ 1hr (default)**

**Conditional Write**
- by default (PutItem, UpdateItem, DeleteItem) are unconditional
- concurrency safe
- idempotent (幂等):send same conditional write request multiple times w/o further effect
- e.g. conditional expression: `ATTRIBUTE_EXIST CONTAINS BEGIN_WITH` `=, <>, <, >,<=, >=, BETWEEN, IN` `NOT AND OR`

**Atomic Counter**: Increment and Decrement
- on **scalar values**
- global incremental counter for visits
- concurrency safe

Batch Operations
- `BatchGetItem` from one or more tables
- **16 MB** of data, which can contain as many as **100 items**
- `ValidationException` if more than 100 items

Common usage: write: `PutItem` `BatchWriteItem` read: `GetItem` `BatchGetItem`

> To achieve high uptime and durability, Amazon DynamoDB synchronously replicates data across three facilities within an AWS Region.


### DynamoDB vs Other

| DynamoDB                | RDS                              |
| ----------------------- | -------------------------------- |
| key-value and document  | relational DB (multiple engines) |
| weak query              | complex query                    |
| fast                    | x                                |
| predictable performance | x                                |


| DynamoDB    | SimpleDB            |
| ----------- | ------------------- |
| No SQL      | No SQL              |
| no limit    | 10 GB               |
| scalability | scaling limitations |
| large       | smaller workload    |

| DynamoDB        | S3                            |
| --------------- | ----------------------------- |
| 1 byte ~ 400 KB | up to 5TB                     |
| small and fast  | large and infrequently access |

### DynamoDB Cross-region Replication
- **replicas** of a  master table to be maintained in one or more AWS regions
- automatically **propagated** to all replicas
- **1 master table** and one or **n replica tables**
- Read replicas are updated asynchronously as DynamoDB acknowledges a write operation as successful once it has been accepted by the master table. The write will then be propagated to each replica with a slight delay.
- Efficient **disaster recovery**, in case a data center failure occurs.
- Faster reads - delivering data faster by reading a DynamoDB table from the closest AWS data center.
- distribute the read workload across tables and thereby consume less read capacity in the master table.
- Easy regional migration, by promoting a read replica to master
- Live data migration, to replicate data and when the tables are in sync, switch the application to write to the destination region
- Reading data from DynamoDB Streams to keep the tables in sync.

### DynamoDB Best Practices
- Keep item size small
- **Store metadata in DynamoDB and large BLOBs in Amazon S3**
- Use **conditional** or Optimistic Concurrency Control (**OCC**) updates
- **Avoid hot keys and hot partitions**, distribute partition evenly
- **Deleting an entire table** is significantly more efficient than removing items one-by-one
- **Maximum operations in a single request** — You can specify a total of up to 25 put or delete operations; however, the total request size cannot exceed 1 MB (the HTTP payload).

## SQS

- the **1st** service in AWS
- highly available distributed queue system
- helps build distributed application with decoupled components
- supports HTTP over SSL (**HTTPS**) and Transport Layer Security (**TLS**)
- SQS – Fanning Out: Create an SNS topic. Then create and subscribe multiple SQS queues to the SNS topic

### FIFO Queue

- provide **exactly-once** processing
- have a limited number of transactions per second (TPS)
- must end with the `.fifo` suffix
- support up to 300 messages per second

### Standard Queue

- Message Ordering, can be delivered in **multiple times and in any order**
- **At-Least-Once** Delivery

### Polling
- `WaitTimeSeconds` parameter of a `ReceiveMessage`, between **1 and 20**
- **Short Polling** by default (set `WaitTimeSeconds` to 0)
- **Long Polling** helps reduce the cost (add a message interval), set `WaitTimeSeconds` from **0 ~ 20**

### Limits

- Message attributes: A message can contain up t **10 metadata attributes**.
- Message batch: A single message batch request can include a maximum of 10 messages
- Message retention: By default, a message is retained for **4 days**  **1 min~14 day**, `MessageRetentionPeriod`
- Message size: **256 KB**, **contains a reference to a message payload in Amazon S3**
- Message visibility timeout: default: **30 seconds**. The maximum is **12 hours**, `ChangeMessageVisibility`. The visibility timeout is the time during which the message is invisible to workers. If this interval is set to "0", the message will be immediately available for processing.

## SNS

### Goals

- Publishers communicate **asynchronously** with subscribers
- **decouple** message publishers from subscribers
- **fan-out** messages to multiple recipients at once
- **eliminate polling** in your applications

### Features

- `publish/subscribe`, **PUSH** notification to a **Topic**
- publish to **HTTP, HTTPS, Email, Email-JSON, Amazon SQS, Application, AWS Lambda, SMS**
- publish **in order**, could result in **out of order** in subscriber side
- subscriber: needs **confirm** the subscription or **unsubscribe** from a topic
- pricing: pay as you go
- TTL (undelivered messages will expire)
- **retry**: attempts a retry after a failed delivery attempt, **total retries / intervals**
- default retry:  **3 times** in the backoff phase, w/ **20 seconds** delay between each retry
- limits: **up to 100 retries** and **max lifetime is 1hr**

### Fanout

```
Publisher -> SNS Topic -> SQS Queue -> EC2 Instances
                    |  -> SQS Queue -> EC2 Instances
                    |  -> SQS Queue -> EC2 Instances
```

For parallel asynchronous processing
- e.g. new order, one queue for processing while the other sending to data warehouse
- e.g. feed production data back to development env for testing on real data


### Publish from VPC

### SQS vs SNS
- both messages services
- SNS is PUSH
- SQS is PULL

> message can be customized for each protocol

## SWF

> !! **coordinate** !! work across **distributed** components

**Task = Activity Task** and **Worker = Activity Worker**

### Workflow

> SWF uses deciders and workers to complete tasks

- **Domain** as container, you **register** an activity in Amazon SWF, you provide a **domain**
- **Actors** can be workflow **starters**, **deciders**, or activity **workers**
  - **Starter** user submit an order in the website
  - **Workers** to get task, process and return result
  - **Decider** to control the coordination of tasks
- **Tasks** SWF interacts with activity workers and deciders by providing them with work assignments known as tasks. **Data Exchange Between Actors**
- **SWF** stores tasks and assigns them to workers when they are ready, tracks their progress, and maintains their state, including details on their completion

Workflow
- the automation of a business process
- a set of activities that carry out some objective, together with logic that coordinates the activities.

Workflow Execution
- a running instance of a workflow

Workflow History
- the state and progress of each workflow execution in its Workflow History
- your app is stateless as all state is stored in workflow history
- **Markers** can be used to record information in the workflow history of a workflow execution

> **Workers** and **Deciders** are both stateless

> **long polling**: requests will be held open for up to **60 seconds** if necessary, **to reduce network traffic and unnecessary processing**

### Workflow Implementation & Execution
1. Implement Activity **workers** with the processing steps in the Workflow.
2. Implement **Decider** with the coordination logic of the Workflow.
3. **Register** the Activities and workflow with SWF.
4. Start the Activity workers and Decider. Once started, the decider and activity workers should start **polling** Amazon SWF for **tasks**.
5. Start one or more **executions** of the Workflow. Each execution runs **independently** and can be provided with its own set of input data.
6. When an execution is started, SWF schedules the initial decision task. In response, the decider begins generating decisions which initiate activity tasks. Execution continues until your decider makes a decision to close the execution.
7. View and Track workflow executions

### Limits

- Max workflow can be **1yr**

!> SWF maintains the execution state. It ensures a task is **assigned only once** and is **never duplicated**

SWF vs SQS
- SWF: task-oriented vs  SQS: message-oriented
- SWF: task assigned only once and never duplicated vs message delivered in multiple times and in any order
- SWF: track execution state vs SQS doesn't

## Beanstalk

automated infrastructure management and code deployment, by simply uploading, for applications and includes
- Application platform management
- Capacity provisioning
- Load Balancing
- Auto scaling
- Code deployment
- Health Monitoring

AWS resources launched by Elastic Beanstalk are fully accessible i.e. you can ssh into the EC2 instances

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

Q: What AWS products and features can be deployed by Elastic Beanstalk
A:
- Auto scaling groups
- Elastic Load Balancers
- RDS Instances

### Environment tier

Web server environment
- An environment tier whose web application processes web requests is known as a web server tier.
- Every Environment has a **CNAME** url pointing to the ELB, aliased in Route 53 to **ELB** url
- Each EC2 server instance that runs the application uses a **container** type, which defines the infrastructure topology and software stack

Worker environment:
- **Daemon** is responsible for pulling requests from an **SQS** queue and then sending the data to the web application running in the worker environment tier that will process those messages
- An environment tier whose web application runs **background** jobs is known as a worker tier
- Worker tiers **pull** jobs from SQS

One environment cannot support two different environment tiers because each requires its own set of resources; a worker environment tier and a web server environment tier each require an Auto Scaling group, but Elastic Beanstalk supports only one Auto Scaling group per environment.

Q: You want to have multiple versions of your application running at the same time, with all versions launched via AWS Elastic Beanstalk. Is this possible
A: AWS Elastic Beanstalk is designed to support a number of multiple running environments

### SQS Dead Letter Queue
- useful for debugging your application or messaging system
- creating a queue and configuring a dead-letter queue
- messages can’t be processed, the message with the order request is sent to a dead-letter queue

### Beanstalk + logs
- are stored locally on individual instance
- CloudWatch: stream logs to Amazon **CloudWatch** Logs in real time.
- On-instance: **Tail and bundle logs** are removed from Amazon S3 15 minutes after they are created.
- S3:  **persist logs**, you can configure your environment to publish logs to Amazon S3 automatically after they are rotated.

### Beanstalk + RDS
- It is recommended to launch a database instance outside of the environment and configure the application to connect to it outside of the functionality provided by Elastic Beanstalk.
- Create your RDS instance separately and pass its DNS name to your app’s DB connection string as an environment variable. Create a security group for client machines and add it as a valid source for DB traffic to the security group of the RDS instance itself.

### Beanstalk + S3
- Elastic Beanstalk creates an S3 bucket named elasticbeanstalk-region-account-id for each region in which environments is created.
- Elastic Beanstalk uses the bucket to store application versions, logs, and other supporting files.
- It applies a bucket policy to buckets it creates to allow environments to write to the bucket and prevent accidental deletion

## CloudFormation

?> turn infrastructure into configuration

- *automatic rollback on error*, charge for errors
- syntax error failed at template validation. Since the stack will never start creating, there is nothing to roll back.
- *WaitCondition* wait for resources to be provisioned
- use `Fn::GetAtt` to output data, called **Intrinsic Function**
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
Conditions: # a production or test environment
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

### Hosted Zones

Simple Policies
- **link the domain to the web server**
- example.com / the DNS name of the ELB (alias)

Weighted
- 50 50 switch between EC2 instances
- split traffic to multiple resources, 50% to vCurrent, 50% to beta

Latency
- route based on latency (cross region)
- route traffic to the resource that provides the best latency.

Failover
- **active-passive failover**
- health check the London ELB endpoint
- 2 records: primary and secondary
- e.g. set your primary in your region, and a **DR** (disaster recovery) site in another region

Geolocation
- **by continent or country** (e.g. all Europe traffic go to EU London)
- route based on geo location (cross region)
- shift traffic from resources in one location to resources in another.

### Traffic Flow
- a visual editor to create a traffic policy
- complex configuration

### DNS

search a public ip by a  domain name

top Level Domains: .com .gov .edu, second level domain name: .com.au, .edu.cn

- SOA Records ()
- NS Records (**name** server by top level domain servers)
- A Records (**domain name to ip address**)
- AAAA Records for IPv6
- TTL (cache, time to live)
- CNAMES (canonical **domain name to domain name**)
- Alias Records (AWS created, easy way to map naked domain name (apex) to **resource record / ELB, CF distribution, S3 bucket**)

```
NAME                    TYPE   VALUE
--------------------------------------------------
bar.example.com.        CNAME  foo.example.com.
foo.example.com.        A      192.0.2.23
```

Tips:
- **ELB do not have pre-defined IPv4 addresses, to resolve using a DNS name**
- Alias vs CNAME
- choose Alias record over CNAME

## VPC

### General

- Virtual Private Cloud
- select **IP address range**, create **subnets**, configure **route tables**, **network gateway**, **security settings**
- IP address in **CIDR** `10.0.0.0/16`, allows `65536` IP address to be available, `http://cidr.xyz/`
- allows customers to expand their existing VPCs by adding secondary CIDRs after they have create the VPC with the primary CIDR block
-
### CIDR - (Classless Inter-Domain Routing)
- `8 bits . 8 bits . 8 bits . 8 bits / 8 bits` -> / the routing prefix, turn into a netmask
- `10.0.0.0/16` count 65535, `255.255.0.0` e.g. large block for a VPC
- `10.0.0.0/20` count 4096, `255.255.240.0`
- `10.0.0.0/24` count 256, `255.255.255.0` e.g. small block for a subnet
- `10.0.0.0/28` count 16, `255.255.255.240`
- recommended CIDR blocks
    - `10.0.0.0` – `10.255.255.255` (10/8 prefix)
    - `172.16.0.0` – `172.31.255.255` (172.16/12 prefix)
    - `192.168.0.0` – `192.168.255.255` (192.168/16 prefix)

### Connections

- VPC is separated from any other VPC
- VPC peering:
  - communications between my VPCs or others VPC using private IP addresses
  - in **same region**
  - **no overlapping** CIDR blocks, e.g. VPC A: `10.0.0.0/16` can peer to VPC B: `10.1.0.0/16`
  - up to **50** peering connections per VPC
  - `Dev-Test and Test-Prod != Dev-Prod`, if you want to push code from Dev to Prod, you have to peer Dev and Prod
- VPC to Corp/Home

### Address

- Private IP: used for communication within VPC
- Private IP: assigned to all, within the IP address range of the subnet
- Public IP: auto assigned: subnet setting or enable in EC2 creation
- Public IP: will be recycled
- Elastic IP: static and persistent
- Elastic IP: can be move fron one instance to another, same or different VPC within the same account
- Elastic IP: $$ for non usage

### Route Table

- Rules: network traffic from the subnet would be routed, for **IGW**, **VPC Peering**, **NAT Device**
- Each VPC: 1 **default** main route table & n custom route tables
- **subnet / route table**: `n` subnet <-----> `1` route table
- Each route table contains a **local route** enables communication within a VPC, cannot be modified / deleted
- matching **the most specific route** in the route table that matches the traffic

### Subnet
- belongs to 1 AZ
- All Subnets in default VPC have a route out to the internet

### IGT (Internet Gateway)
- EC2 instances to access internet
- managed service, horizontally scaled, no bandwidth constraints on the network traffic
- attached to 1 VPC

1. attach **IGW** to VPC
2. subnet route tables should **route to the IGW**
3. assign **Public IP or Elastic IP **to the instance
4. **security group and Network ACL** associated with the instance

### NAT
- in **public subnet**, w/ **Elastic IP** address, to enable `instances --> internet`, but not the other round
- private instances need NAT Gateway to perform software updates
- **NAT Instance**: **disable source/destination check**
- **NAT Gateway**: fully managed service, scale automatically, no patch, no security group
- allows **outbound** communication but doesn’t allow machines on the internet to initiate a connection to the privately addressed instances

### Security Group (EC2 instances)
- firewall for associated EC2 instances, **inbound / outbound** traffic at the **instance level**
- specify only **Allow rules, but not deny rules**
- `stateful`, return traffic is automatically allowed

### Network ACL (subnets)
- firewall for associated subnets, **inbound / outbound** traffic at the **subnet level**, applicable to all the instances in the subnet
- **Default** and **Newly created** ACL **allows all inbound and outbound traffic**
- **Allow Rules and Deny Rules**
- a subnet must be associated with **one** Network ACL, if not the subnet is automatically associated with the default network ACL
- You can associate a network ACL with multiple subnets; however, a subnet can be associated with only one network ACL at a time. `Subnet n----1 ACL`
- A network ACL contains a numbered list of rules that is evaluated in order, starting with the lowest numbered rule
- A network ACL has **separate inbound and outbound rules**, and each rule can either **allow or deny** traffic
- `stateless`, return traffic must be explicitly allowed by rules

### NAT Vs Bastion
- A NAT instance is used to provide internet traffic to EC2 instances in private subnets
- A Bastion is used to securely administer EC2 instances (using SSH or RDP) in private subnets.

### Limits
- reserved 5 IPs address (first 4 and last 1 IP address) in each Subnet. e.g. for a Subnet with a CIDR block 10.0.0.0/24 the following five IPs are reserved
  - 10.0.0.0: Network address
  - 10.0.0.1: Reserved by AWS for the VPC router
  - 10.0.0.2: Reserved by AWS for mapping to Amazon-provided DNS
  - 10.0.0.3: Reserved by AWS for future use
  - 10.0.0.255: Network broadcast address. AWS does not support broadcast in a VPC, therefore the address is reserved.
- **VPC per region** = `5`
- **subnets per VPC** = `200`
- IPv4 CIDR blocks per VPC = `5`
- **Elastic IP per region** = `5`
- IGW per region = `5`
- NAT Gateways per AZ = `5`
- Virtual private gateways per region = `5`
- **Rules per Network ACL** = `20`
- **Inbound or Outbound Rules per Security Group** = `50`
- Route table per VPC = `200`
- VPC peering per VPC = `50`


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

