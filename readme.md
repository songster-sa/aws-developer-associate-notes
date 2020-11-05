# AWS Developer Associate Exam Notes 2019-2020

Based on Udemy course by aCloudGuru (Ryan), own research and practise tests.
The topics on which most questions are based are highlighted in bold.
Some references :
https://github.com/mransbro/aws-developer-notes

## IAM - Identity Access Manager
- Users, roles, groups, policy document (json), secret key and access key
- Roles - provide permissions to resources / services
- **ARN**
- policies
    - inline - embedded in user/role/group
    - managed - aws managed - can be reused - cannot be customised
    - customised - you create - only resides in your account
- IAM policy simulator - to test policies before production

## Amazon Cognito - web identity federation
- used for larger companies
- cognito is ID broker
- types
    - user pools - JWT
    - identity pool - STS (security token service) API
    - aws credentials - mapped to IAM role
- SMPS - system manager parameter store - to store credentials

## EC2 - Elastic Compute Cloud
- types - FIGHT DR MC PIX
- **Price options**
    - on demand
    - reserved
    - spot
    - dedicated hosts
- provisions via images (ami = amazon machine image)
- max of tags allowed 50
- IAAS - infrastructure as a service
- **access public IP via http://169.254.169.254/latest/meta-data**
- EFS - Elastic File System - file system on EC2
- EBS - block storage on EC2

## EBS - Elastic Block Storage
- **2 types**
    - General purpose SSD
    - Provisioned IOPS SSD ->10k IOPS -> high performance
- magnetic disks
    - throughput optimised HDD
    - Cold HDD (file save) -> where data is NOT accessed regularly
    - Standard -> bootable -> where data is NOT accessed regularly
    
## ELB - Elastic Load Balancer
- **types**
    - ALB - on OSI layer 7 - application aware
    - NLB - on OSI layer 4 - extreme performance - million requests / sec
    - Classic - both 4 and 7 - x-forward and sticky session - 504 error when app not responding
    
## Route 53 - DNS
- applicable to EC2, S3, LB
- limit to 50 domain names
- types
    - CNAME - canonical name
    - record (address record) - map to IP address
    - alias - best - chose this in exam over CNAME - free
- routing policies
    - simple - default - when there is only 1 instance
    - weighted
    - latency - to lowest latency region
    - failover - via health checks
    - geolocation

## RDS - Relational Database Service
- OLTP - online transaction processing
- aws supports - mysql, mssql, oracle, postgres, aurora,  mariaDB
- when created- you get DNS endpoint (good for DR)
- set endpoint in EC2 and whitelist the EC2 security group in RDS security group
- backups
    - automated - daily - to the second - saved in S3 - retention 1-35days - deleted when RDS deleted
    - manual - user intiated, user deleted
- restoring / encryption - via new instance
- if DB encrypted, backup encrypted too
- Multi AZ - Synchronous replica
- Read Replica - Asynchronous replica - only reads - performance - scaling out - max 5
- can be set up inside EC2 or outside (outside is adv)

## Data warehousing
- OLAP - online analytics processing
- business intelligence - query huge data for stats or reports
- RedShift

## Elastic Cache
- in memory cache
- **types**
    - mem cahche - simply - no persistence - scale out - 
    - Redis - leader boards - sorting - like RDS - multi az - persisted - stateful
    
## S3 - Simple Storage Service
- object bases (instead of block based)
- 99.9% availability
- **unlimited storage**
- **0 bytes to 5 TB**
- **globally unique DNS**
- **Read (immediately) after write for new PUTs objects**
- **Eventually consistent for modify / overwrites via PUTs and DELETEs**
- ** PUTs by default support max 5GB data**
- secure data by
    - access (IAM)
    - ACL
    - bucket policies
    - CORS
    - Transfer Acceleration - use Cloud Front to upload data
- **Tiers**
    - Regular S3 - high redundancy
    - S3-IA - infrequent acess - charged per retrieval
    - S3-one zone IA - stored in 1AZ - cost 20% less
    - Reduced redundancy - when data can be recreated
    - Glacier - archival use - very cheap -very infrequent - takes 3-5hr to retrieve
    - Intelligent tier - for unkown patterns (2 subtypes - frequent and infrequent)
- **data is private by default - public access is forbidden by default**
- **Encryption**
    - in transit - SSL / TLS applied on https requests
    - at rest
        - server side
            - SSE-S3 - AES256 keys - S3 managed
            - SSE-KMS - KMS / AWS managed - you get master key
            - SSE-C - customer managed - you manage the keys but AWS encrypts/decrypts
        - client side - you encrypt before upload
    - how? - add a parameter in PUT request and check for it in bucket policy
- **CORS - cross origin resource sharing - cross bucket data access**
- **for performance optimization**
    - if GET / read is more - use Cloud Front
    - if mixed - add hex hash as suffix to key names - so they are saved in diff partitions
- if expect more than 300 PUT/LIST/DELETE req/sec or more than 800 GET req/sec - raise support request with AWS
- SGS - Simple Gateway Service - file system on S3
- **how to access bucket - s3://bucket - region**

## Cloud Front - CDN - content delivery network
- **for fast reads and downloads**
- **first time read form root server, then store in EDGE LOCATIONS**
- eq sydney user trying to access ireland data
- **Can be used to upload data - S3 transfer acceleration**
- 2 kinds of distribution
    - real time messaging protocol
    - web distribution
- via Signed URL / Signed Cookies
- by default applies to all edge locations, but you can blacklist countries
- you can manually remove a file from the cache - but charged for it
- web application firewall VS network firewall
- **TTL** can be 0sec to 365 days - default is 24hr

## ECS / ECR - elastic containers service / elastic containers registry
- buildSpec.yaml file

## Serverless computing - Lambda and friends
- event driven, API driven
- node.js, python, java, c#, go
- scale out, not scale up
- lambda functions have to be independent
- **use X-ray to debug** - annotations or indexes in code/data/traces
- API vs API gateway
    - using API gateway we can configure which event triggers which serverless service
    - urls, http methods, SSL/TLS, targets
- API caching = API response caching
- browsers cache against same origin - so AWS uses CORS to change origin (to refresh cache)
- **Lambda versioning** - use alias
- lambda concurrent execution limit 1000
- API gateway errors 
    - **429 - too many requests** - throttle - exponential backoff etc
- **VPC** virtual private cloud
    - like a VPN for your account
    - private subnet id
    - security group id , route table, internet gateway, endpoint

## Dynamo DB
- noSQL - collection(table) - document(row) - key-value pairs
- **consistency model**
    - eventually consistent read - consistent in 1 sec
    - strongly consistent read
- **primary keys**
    - single attribute
    - composite = primary key + sort key
- primary key is unique ; sort key is a sequence eg date
- **indexes**
    - local - same primary key, diff sort key - created with table only
    - global - diff primary key, diff sort key - can be created later
- **Dynamo DB Streams = change log**
    - stores event data
    - can be used for event triggers
    - stores for 24hr only
    - usually used with lambda
    - 4 options - keys only, new image, old image, both
- **operations**
    - query - by partition key
    - scan - scan entire table - return all fields by default 
        - do many small scans for performance improvement
        - projection - when scan returns only few fields
- **query is more efficient in general**
- **BatchGetItem API for more efficient queries on large items**
- **Provisioned throughput**
    - RCU - read throughput - 4KB reads - 2 for eventually consistent
    - WCU - write throughput - 1KB write
    - error when throughput exceeds - 400 - ProvisionedThroughputExceededException
- **DAX - Dynamo Acceleration - like cache**
- **when overloaded**
    - client retry
    - exponential backoff
    - reduced request frequency

## KMS - Key Management Service
- encryption keys are regional
- keys
    - envelop key / data key
    - master key

## SQS - Simple Queue Service
- message queue - distributed - pull based
- **<= 256KB**
- **canNOT prioritise messages - better to set 2 queues**
- **types**
    - standard - best effort - may not be ordered - unlimited T/sec
    - FIFO - ordered - no duplicates - 300T/sec
- retention - 1min - 14days - default 4 days
- visibility timout - <=12hours - default 30 sec (ChangeMessageVisibility API ?)
- **polling**
    - short polling - response every time - whether empty or not
    - long polling - response only when data or timeout (20sec)
- not supported in all regions - only in 4
- 1st million requests are free - then 0.50 
- Delay Q - msg stays invisible form 0-900sec
- for msg >256KB to 2GB - use S3

## SWF -Simple Workflow Service
- task based queue (rather than msg based)
- task once assigned are never duplicated; tasks are tracked
- worker programs get tasks ; decider programs control the coordination

## SNS - Simple Notification Service
- push based
- mobile, email, website, sms, **SQS**, **lambda**
- Topic - group of recipients
- multi AZ
- publish subscribe
- Synchronous

## SES - Simple Email Service
- for marketing emails
- on receive reply - you can trigger lambda
- Asynchronous
- no subscription required

## Elastic BeanStalk
- PAAS - provisioning as a service
- **Deployment Policies**
    - All at once - app down - rollback
    - rolling - not down but performance hit - rollback
    - rolling with additional batches - no performance hit - no rollback
    - immutable - blue/green - no rollback
- .config file (yaml) inside .ebextensions (json)

## Cloud Formation
- script based provisioning - **yaml (preferred) or json**
- AWS Template Format Version - 2010-09-09
- resulting resources called Stack
- **Terminology**
    - parameter - what will be asked
    - conditions - tests run on parameters provided
    - mapping - to be available to stack eg region
    - transfer - include snippets sitting outside the template
    - resources - mandatory - resources you want to deploy
    - outputs - on console or input to another stack
- SAM - servless app model - for serverless provisionings
    - sam-package - input yaml output sam-template.yaml
    - sam-deploy
- **nexted stack** - template referenced in another template
    - in the resources section

## Kinesis - for streaming data
- 3 services
     - streams - in shards - retention 24h-7days
     - firehose - no retention - automated shards
     - analytics - run sql queries on data

## CI/CD Services
- code commit - git repo in aws - src control
    - **set notifications - via cloud watch or SNS**
- code build - compile build test
- code deploy - deploy on EC2, lambda etc
    - in place - rolling update
    - blue/green - immutable
- code pipeline - release management - orchestration
    - **if sees test failure - it will stop**
    - can watch S3 bucket for new zip/jar/war
- **Terminology**
    - deployment group
    - deployment config - **AppSec file - yaml(EC2) - yaml or json (lambda)**
    - revision = artifact
    - application = [ group + config + revision]
- **Hooks - defines actions to be done at specific points in the deployment lifecyle**
    - application start vs validate service
    - before allow traffic vs allow traffic vs after allow traffic

## Cloud Watch - monitoring and logs
- monitors performance of resources
- by default logs stored forever - even after termination
- in EC2 - by default monitors host level metrics
    - n/w, overall disk IO, CPU, status check
    - canNOT see how much space left
- default monitoring
    - in EC2 every 5 min
    - others every 1min
- GetMetricsStatistics API

## Cloud watch alarm - 1min/2min detailed monitoring

## Cloud Trail - API calls monitoring - resource provisioning

## AWS CLI Pagination
- default page size 1000
    - can give timeout
    - if result size is 2500 - it will make 3 calls internally but show the result altogether
- **use small page size** to avoid timeout
- default max items 100
    - still fetch all, show these many and return