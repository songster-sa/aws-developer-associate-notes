# AWS Developer Associate Exam Notes 2019-2020

Based on Udemy course by aCloudGuru (Ryan) and Stephane Marek's course, own research and practise tests.
The topics on which most questions are based are highlighted in bold.
Some references :
https://github.com/mransbro/aws-developer-notes

## IAM - Identity Access Manager
- Users, roles, groups, policy document (json), secret key and access key
- Roles - provide permissions to resources / services
- multi factor authentication
    - for root account SMS MFA is NOT supported (Virtual MFA, Hardware MFA and U2F are supported)
- **ARN**
- policies
    - inline - embedded in user/role/group
    - managed - aws managed - can be reused - cannot be customised
    - customised - you create - only resides in your account
- IAM policy simulator - to test policies before production
- IAM Policy
    - principal - person or application - whom the policy is applicable for
    - resource - objects that policy covers
    - variables - placeholders - help in single policy behaving differently for each user
    - condition - for when the policy should be effective
- **cross account access** can be configured to give user of 1 account access to stuff of another account
- not for code commit
- --dry-run - to check if you have permissions without making any request - UnauthorisedOperation

## Amazon Cognito - web identity federation
- used for larger companies
- cognito is ID broker - it provides seamless experience across all devices
- types
    - user pools - JWT - manage sign-in and sign-up
    - identity pool - STS (security token service) API 
        - create unique identities for users and authenticate them with web ID provider
        - only temporary access
    - aws credentials - mapped to IAM role
- SMPS - system manager parameter store - to store credentials
    - can store but cannot rotate
- **API call - STS assume-role-with-web-identity**
- **user authenticates with FB - FB gives ID token - then use this token to get temp creds via cognito**
- cognito provides features - 
    - signin signup to your apps - user pool
    - sync of user data across devices - **cognito Sync**

## EC2 - Elastic Compute Cloud
- types - FIGHT DR MC PIX
- **Price options**
    - on demand - office hours, peaks
    - reserved - steady, predictable, 3yr contract
        - capacity reserved
        - 3 types - standard, scheduled, convertible
        - by default - zonal instances - per available zone
        - you can buy regional reserved instances - they DONT reserve capacity
    - spot - high utilization, when data is not lost 
        - when interrupts - stop , terminate (default), hibernate - cannot reboot
    - dedicated hosts - secure, no multi-tenant virtualization
- provisions via images (ami = amazon machine image)
- provisioned per (by) availability zone
- max of tags allowed 50
- IAAS - infrastructure as a service
- **access public IP via http://169.254.169.254/latest/meta-data**
- EFS - Elastic File System - file system on EC2 - network file system
    - multi AZ - expensive but pay per use - need security group
    - shared file system across AZ 
    - only for linux
    - tiers - standard vs IA (infrequent access)
- EBS - block storage on EC2
- Instance Store - ephemeral - EC2 stopped, data is lost - physical (no network) - hence very high IOPs (millions)
- **Placement groups**
    - Spread Placement group - small num of critical instances - not share underlying hardware hence wont fail together
    - Cluster Placement group - 
- user data when provided to EC2 instances - for boot strap
    - runs only at boot time - when you first time launch the instance
    - scripts run with root privileges
- **EC2 Auto scaling** - new instances - always scales horizontally
    - when in VPC - are launched within subnet
    - can be in ANY AZ but within same region
    - works with both ALB and NLB
    - Scaling Policies
        - target tracking - on averages
        - simple/step scaling
        - scheduled actions
    - when set up with Management console - no detailed monitoring
    - cooldown periods
- first time ssh - key pair - permission denied - chmod 0400
- ENI - assign private ips to EC2 - ssh not changed on failover
- pricing - 60sec min - not when stopped
- for EC2 to do something (cloud watch, code build, xray)- you need agents - download and run - then your installed code will integrate

## EBS - Elastic Block Storage
- persistant - if EC2 stopped, no data loss here
- network drive - AZ locked - can be attached to only 1 EC2
- to share you need to take snapshot and then create using it
- can inc size over time - pay for provisioned capacity
- **2 types (both boot volumes)**
    - General purpose SSD - GP2 - 3k to 16k iops - ratio 3:1 - 3000 IOPs bursts
    - Provisioned IOPS SSD - IO 1 ->16k IOPS -> high performance -> 50:1 is max allowed ratio
- magnetic disks (HHD)
    - throughput optimised HDD (STI) - max 500 IOPs - for streaming workloads - fast throughput - 40:1
    - Cold HDD (SCI file save) -> where data is NOT accessed regularly - max 250 IOPs - 12:1
    - Standard magnetic-> bootable -> where data is NOT accessed regularly
- configure encryption while creating EBS volume
    - supports both in-flight and at-rest
    - the "default" option is region-specific - if enabled you cannot avoid for a volume
    
## ELB - Elastic Load Balancer
- **types**
    - ALB - on OSI layer 7 - application aware
    - NLB - on OSI layer 4 - extreme performance - million requests / sec
    - Classic - both 4 and 7 - x-forward and sticky session - 504 error when app not responding
        - legacy ones - mainly used for Classic EC2 instances
- target groups - for routing traffic by hostname / ip / lambda
- SNI - to support multiple SSL/TLSs certificates
- **ELB per AZ; region locked**
- logs to show public IP address - use x-forward-for header
- separate public and private traffic
- makes instances highly available (obviously)
- communicates to instances using private IP
- can connect with Auto Scale groups to provide horizontal scaling
- can target instances only within a single region
- **cross zone balancing (always on for ALB)** - charged for NLB
    - enabled - divide traffic across AZ
    - disabled - divide traffic only within own AZ
- ALB access logs - details abt the requests
    - time, IP, latencies, path, response etc
- ALB request tracing - 
- ALB listeners - like rules
    - for HTTPS listener - deploy certificate, 
        - SSL termination - node will decrypt request and then forward them down
        - SSL pass through - downstream nodes will need to decrypt requests
- ALB errors
    - 503 - service unavailable - when downstream targets are available / set
    - 504 - gateway timeout - targets are there but not connected/reachable/responding - security group
    - 403 - forbidden - WAF not allowing
    - 500 - internal server error - 
- ALB can authenticate users using Cognito
- connection draining - 300sec by default - when marked unhealthy and terminating
- NLB exposes public static IP whereas ALB and CLB exposes DNS URLs
- may distribute load based on EC2 instance types as well - low type less load
    
## Route 53 - DNS
- applicable to EC2, S3, LB
- limit to 50 domain names
- types
    - CNAME - canonical name - non-root hostname to ANY hostname - for non-root domain only
    - record (address record) - hostname to IP address (A , AAAA)
    - alias - best - ANY hostname to AWS resources - mydomain.com : xx.amazonaws.com - free - for both root and non-root domains
- routing policies
    - simple - default - can return more than 1 ips - browser can chose where to go - no health check
    - multi value - same as simple WITH health check
    - weighted
    - latency - to lowest latency region
    - failover - via health checks
    - geolocation
- provides load balancing and health checks(3 default; 30sec interval), TTLs
- DNS changes will start reflecting after TTL expired only - as browser wont query until then

## RDS - Relational Database Service
- OLTP - online transaction processing
- can scale both horizontally and vertically ??
- aws supports - mysql, mssql, oracle, postgres, aurora,  mariaDB
- when created- you get DNS endpoint (good for DR)
- set endpoint in EC2 and whitelist the EC2 security group in RDS security group
- backups
    - automated 
        - daily 
        - to the second - saved in S3
        - retention 1-35days (default 7)
        - deleted when RDS deleted 
        - transaction logs every 5min
    - manual - user initiated, user deleted
- restoring / encryption - via new instance
    - at rest(KMS) or in-flight(SSL certificate)
- if DB encrypted, backup encrypted too
- but we can encrypt an unencrypted snapshot by copying as encrypted
- Multi AZ - Synchronous replica
    - stand by DB - for fail over / DR
- Read Replica - Asynchronous replica 
    - only reads - performance - scaling out - max 5
    - can be within AZ, cross AZ, cross region
    - eventually consistent
    - can be made into own DB
    - can be set for failover / DR
- can be set up inside EC2 or outside (outside is adv)

## Aurora
- supports postgres and MySQL
- is like RDS - no flexible schema
- very cloud/high optimised
- storage automatically grows - auto scalling
- 15 read replicas
- failover is instantaneous
- 6 copies across 3 AZ
- self healing
- writer endpoint = master
- reader endpoints - connection load balancing
- aurora serverless
- global aurora 
    - cross region read replicas
    - global database 
        - 1 primary region for read/writes - 1 min DR to make another region primary
        - upto 5 secondary (read only) regions
        - upto 16 read replicas per secondary region

## Data warehousing
- OLAP - online analytics processing
- business intelligence - query huge data for stats or reports
- RedShift

## Elastic Cache
- in memory cache - RDS for caching
- read scaling, write scaling, multi az
- user session store - store session in cache
- for read-heavy AND compute-intensive
- **types**
    - mem cache - simple - high performance
        - no persistence - no backup
        - sharding
        - scale out/horizontally 
        - object caching
        - multi threading
    - Redis - leader boards 
        - sorting - like RDS 
        - multi az - auto failover
        - **read replicas**
        - **persisted**
        - stateful
        - encryption - at rest / in transit ( redis auth)
- strategies
    - lazy loading / cache aside / lazy population - add to cache when cache miss
        - data in cache may be stale - eventually consistent
        - read may be slow
    - write through - add to cache when db is updated
        - data not stale - data may be never read
        - write is slow
- cache eviction - LRU ot TTL
        
## S3 - Simple Storage Service
- object bases (instead of block based)
- 99.9% availability
- **unlimited storage**
- **0 bytes to 5 TB**
- **globally unique DNS**
- **Read (immediately) after write for new PUTs objects** - GET-PUT-GET may not work
- **Eventually consistent for modify / overwrites via PUTs and DELETEs**
- **PUTs by default support max 5GB data** - use multi-part upload for more 
- for file more than 100MB - multi part is any recommended
- bucket names are global but buckets are region resource
- no concept of directories - just key names with slashes
- secure data by
    - user based - access (IAM)
    - resource based - bucket policies - resource based policy - mention what resource/users have access
        - ACL - to control which principals in another account can access a resource
        - CORS
    - Transfer Acceleration - use Cloud Front to upload data
- **Tiers**
    - Regular S3 - high redundancy
    - S3-IA - infrequent access - charged per retrieval
    - S3-one zone IA - stored in 1AZ - cost 20% less
    - Reduced redundancy - when data can be recreated
    - Glacier - archival use - very cheap -very infrequent - takes 3-5hr to retrieve
    - Intelligent tier - for unkown patterns (2 subtypes - frequent and infrequent)
- S3 licycle rules
    - transition actions
    - expiration actions
- **data is private by default - public access is forbidden by default**
- **Encryption**
    - in transit - SSL / TLS applied on https requests
    - at rest
        - server side
            - SSE-S3 - AES256 keys - S3 managed
            - SSE-KMS - KMS / AWS managed - you get master key
            - SSE-C - customer managed - you manage the keys but AWS encrypts/decrypts - have to use HTTPS as u will send the key - can do this only via CLI
        - client side - you encrypt before upload - u can use a lib "amazon s3 encryption client"
    - how? - add a parameter in PUT request and check for it in bucket policy
- **CORS - cross origin resource sharing - cross bucket data access**
    - access-control-allow-origin header in requests - match with CORS setting
    - access-control-allow-methods header in requests - match with CORS setting
    - access-control-max-age
- **for performance optimization**
    - if GET / read is more - use Cloud Front
    - if mixed - add hex hash PREfix to key names - so they are saved in diff partitions
    - for uploads - use multi parts and transfer accelerations
    - for downloads - use byte-range fetches
- if expect more than 300 PUT/LIST/DELETE req/sec or more than 800 GET req/sec - raise support request with AWS
- SGS - Simple Gateway Service - file system on S3
- **how to access bucket - s3://bucket - region**
- can fetch a private object via cloud front - Signed URL / Signed Cookies / origin access identity
- versioning - enable-disable - (null-wont loose previous objects) - delete marker
- S2 access logs; MFA delete
- hosting static website - 403 error - as buckets are private
    - make bucket public
    - make bucket policy
- S3 object lock - write once read many - WORM - diff from "object locking" (when 2 PUTs come at the same time)
- S3 analytics - analyse storage access patterns
- Athena - Serverless data analysis service allowing you to query data in S3
- S3 Select / Glacier Select - select columns in csv
- S3 notifications - may miss if objects not versioned

## Cloud Front - CDN - content delivery network
- **for fast reads and downloads**
- **first time read form root server, then store in EDGE LOCATIONS**
- eq sydney user trying to access ireland data
- **Can be used to upload data - S3 transfer acceleration**
- 2 kinds of distribution
    - real time messaging protocol
    - web distribution
- via Signed URL / Signed Cookies / origin access identity
- by default applies to all edge locations, but you can blacklist countries
- you can manually remove a file from the cache - but charged for it
- web application firewall VS network firewall
- **TTL** can be 0sec to 365 days - default is 24hr
- cannot be used to cache DB data (its for files, pages, videos etc)
- CloudFront Key Pairs - can ONLY be created by root account
- HTTPS - can be done end-to-end -> frontend-cloud front-backend
    - Viewer Protocol Policy
    - Origin Protocol Policy

## ECS / ECR - elastic containers service / elastic containers registry
- buildSpec.yaml file
- ECR - to store, manage, and deploy Docker container images (uses S3 internally)
- ECS - to run, stop, and manage Docker containers on a cluster
- launch types
    - Fargate launch type - for serverless
    - EC2 launch types - for more control you can launch the tasks on EC2 too
    - EKS - for kubernetes
- task placement
    - binpack
    - random
    - spread
- single task vs separate tasks
    - when containers are launched on single tasks - they share memory
- to change clusted - update ECS_CLUSTER parameter in /etc/ecs/ecs.config

## Serverless computing - Lambda
- event driven, API driven
- node.js, python, java, c#, go
- inc RAM - when more than 1 CPU then use multi threading
- lambda functions have to be independent
- **Lambda versioning** - use alias
    - within single alias - you can divide traffic to multiple versions
- **lambda deployment**
    - zip upload to lambda console
    - copy paste code in lambda ide
    - cloud formation
- **lambda concurrent execution limit 1000**
    - out of 1000 - 100 is kept for unreserved account concurency - so we can set max 900
    - provisioned concurrency - to scale without fluctuations in latency
    - reserved concurrency - limits the maximum concurrency for the function
- **concurrent execution = (event or request per second) X (execution duration)**
- Lambda authorizer - to control access to API - 3rd party authorization strategies
    - 2 types : 
    - token based ( JWT or Oauth)
    - request parameter based
- execution env = temp runtime env
- execution context = things done above/before/outside the function handler
- 4KB limit on env var
- lambda layers - to decouple dependencies
- lambda destinations - eg. output or put msg in SQS
- to expose to public - 3 ways - create roles for public, ALB, API gateway
- create 1 alias - divide % traffic to diff versions
- lambda limits - unzipped <=250MB; /tmp 512MB ; env var 4KB
- configure auto-scaling to manage provisioned concurrency on a schedule
- Exceptions
    - InvalidParameterValueException - when you use CLI and any parameter is invalid - like role or something
    - CodeStorageExceededException
    - ResourceConflictException
    - ServiceException - for other internal errors in Lambda Service
- Step functions
    - input path and parameters
    - result path - to decide on next step
    - output path - to decide / filter what you want to send to next step
- state machines - task, choice, fail, succeed, pass, wait, parallel, map
    - you can have activities in them
- InvokeAsync - is deprecated - so use Invoke API with EVENT invoke type - for async call

## X ray
- **use X-ray to debug** - annotations or indexes in code/data/traces
    - analyze and debug distributed applications
    - for performance issues and errors
    - visualize the whole request trace
    - works with ec2, ecs, lambda, beanstalk
- X-Ray sampling 
    - control the amount of data that you record - input from client or application
    - modify sampling behavior on the fly
- X-Ray daemon - "agent" that needs to be running on ec2
    - listens for traffic on UDP port 2000, and relays it to the AWS X-Ray API every sec
- x-ray integration - for lambda
- You can use X-Ray to collect data across AWS Accounts.
- target unified account - create a role in it; allow roles in other sub-accounts to assume this role
- for containers
    - deploy the agent as side-car
    - provide IAM task roles to the container
- with SDK
    - AWS_XRAY_TRACING_NAME
    - AwS_XRAY_DEBUG_MODE
- with lambda - env variables
    - AWS_XRAY_CONTEXT_MISSING (default LOG_ERROR)
    - _X_AMZN_TRACE_ID
- segments - define fields to include in trace
- subsegments - for granular timing details
    - when resources dont provide their own segments, u can write subsegments
    - these are convered to "inferred segments" by x-ray
- both segments and subsegments may contain :
    - metadata - for details of objects and array fields
    - annotations - for indexes for filter expressions

## API gateway
- API vs API gateway
    - using API gateway we can configure which event triggers which serverless service
    - urls, http methods, SSL/TLS, targets
- API gateway caching = API response caching = stage cache
    - Cache-Control:max-age=0 -> to fetch fresh everytime - but should "Require Authorization" as well
- browsers cache against same origin - so AWS uses CORS to change origin (to refresh cache)
- API gateway errors 
    - **429 - too many requests** - throttle - exponential backoff etc
    - **504 - gateway timeout - integration failure/timeout - lambda has been running for more than 29 seconds**
- use swagger to convert APIs
- API gateway provides a pass through to SOAP - no conversions
- AWS Secure token Service - cannot be used with API gateway for authentication
- API Gateway - every change needs deployment
    - stage variables - key-value config attributes - like env var
    - mapping template - map payload (request-to-request or response-to-response)
- API Gateway Usage Plans - who can use which deployed API
- canary deployment - deploy 2 gateways with same public URL - divide % traffic to 2 lambdas
- access control - via sigv4, lambda authorizer, cognito user pool - no STS
- has not user pools or security groups
- to prevent unauthorised domain - restrict CORS
- 501 - not implemented - check x-ray
- integration types
    - lambda proxy integration - AWS_PROXY
    - lambda custom integration - AWS
    - HTTP proxy integration - HTTP_PROXY
    - HTTP custom integration - HTTP
    - mock integration - MOCK
- enable "active-tracing" to support xray and send incoming requests and traces to xray

## VPC- virtual private cloud
- region locked - subnets are per AZ
- like a VPN for your account
- private subnet id
- security group id and network ACL
- route table - default limit of 200 routes
    - a subnet can be part of one route at a time (but 1 table can have many subnets)
- vpc endpoint gateway - private subnet to connect to public aws resources like S3 and dynamoDB
- internet gateway - helps public subnet to connect to internet
- NAT gateway - helps private subnet to connect to internet - as it itself sits in public subnet
- VPC Flow logs; subnet flow logs, ENI flow logs
- VPC IP range is called CIDR range
- VPC peering - connect 1 vpc to another
- site-to-site vpn VS direct connect - on premise to vpc connection

## Dynamo DB
- noSQL - collection(table) - document(row) - key-value pairs
- good for session state storage
- **consistency model**
    - eventually consistent read - consistent in 1 sec
    - strongly consistent read
- **primary keys**
    - single attribute
    - composite = primary key + sort key
- primary key is unique ; sort key is a sequence eg date
- **indexes**
    - **when set, GSI capacity units takes precedence over the table's capacity**
    - local - same primary key, diff sort key - created with table only
    - global - diff primary key, diff sort key - can be created later
    - The GSI can throttle - provision more RCU and WCU to the GSI - 
    - sparse index - only written when sort key present in an item
- **Dynamo DB Streams = change log**
    - stores event data
    - **can be used for event triggers**
    - stores for 24hr only
    - **usually used with lambda**
    - 4 options - keys only, new image, old image, both
- **operations**
    - query - by partition key
    - scan - scan entire table - return all fields by default 
        - do many small scans for performance improvement
        - projection - when scan returns only few fields
- **performance improvements**
    - small parallel scans
    - small page size (more queries but faster with no time outs)
    - DAX for read-heavy
    - Global tables
    - choose eventually consistent over strongly consistent
- **query is more efficient in general**
- **BatchGetItem API for more efficient queries on large items**
- **Provisioned throughput**
    - RCU - read throughput - 4KB reads - 2 for eventually consistent (so divide by 2)
        - 5kb/4kb = 1.2 = considered as 2 not 1 (whole number) ??
    - WCU - write throughput - 1KB write
    - error when throughput exceeds - 400 - ProvisionedThroughputExceededException
- **DAX - Dynamo Acceleration - highly available, in-memory cache, only write through**
- **when overloaded**
    - client retry
    - exponential backoff
    - reduced request frequency
- optimistic concurrency - for conditional writes
- scales horizontally
- backups
    - automated - but we dont have access to it
    - hence use - AWS Glue, Hive + EMR, Data pipeline

## KMS - Key Management Service
- solution to create and control your encryption keys
- KMS data size limit - 4KB
- **encryption keys are regional**
- **KMS is multi tenant whereas CloudHSM is dedicated hardware solutions**
- aws kms encrypt - command to encrypt
- aws kms enable-key-rotation
- re-encrypt API call - to encrypt with new CMK
- keys
    - envelop key / data key - to encrypt data
    - master key (CMK) - to encrypt the envelop key
- **canNOT export keys**
- **max data size 4kb** - what u can send to AWS to encrypt
- **Envelope encryption** you fetch key - the key travels on network - and you use it to encrypt - performance efficient
- GenerateDataKey - returns encrypted data key and plain text data key as well
- GenerateDataKeyWithoutPlainText - only returns encrypted data key

## SQS - Simple Queue Service
- message queue - distributed - pull based
- msgs can be delivered multiple times
- first service on aws platform
- scales automatically
- **<= 256KB**
- **canNOT prioritise messages - better to set 2 queues**
- **types**
    - standard - best effort - may not be ordered - unlimited T/sec
    - FIFO - ordered - no duplicates - 300T/sec
        - MessageGroupID
        - MessageDeduplicateID
        - ContentBasedDeduplication - This is not a message parameter, but a queue setting.use an SHA-256 hash to generate the message deduplication ID using the body of the message
- **retention - 1min - 14days - default 4 days**
    - change via queue message retention setting
- **visibility timeout - <=12hours - default 30 sec (ChangeMessageVisibility API)**
- **polling**
    - short polling - response every time - whether empty or not
    - long polling - response only when data or timeout (**20sec**) - efficient repeated polling
- not supported in all regions - only in 4
- 1st million requests are free - then 0.50 
- Delay Q - msg stays invisible form 0-900sec
- for msg >256KB to 2GB - use S3 or SQS Extended Clients
    - no multi part API / concept
- **use with SNS to "fan out" msgs to multiple queues**

## SWF -Simple Workflow Service
- task based queue (rather than msg based)
- task once assigned are never duplicated; tasks are tracked
- worker programs get tasks ; decider programs control the coordination

## SNS - Simple Notification Service
- push based
- msgs CAN be customised by protocol type
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
- .config file (yaml or json) inside .ebextensions folder  - for configurations
- .yaml file - for env variables
- **jetty for jboss not supported**
- for HTTPS - update .ebextention/xxx.config , add SSL
- not for lambda deployment
- can deploy war or zip -> not tar
- command "eb deploy"

## Cloud Formation
- **script based provisioning - yaml (preferred) or json**
- AWS Template Format Version - 2010-09-09
- resulting resources called **Stack**
- if any error - rollback everything
- **Terminology**
    - parameter - what will be asked
    - conditions - tests run on parameters provided
    - mapping - to be available to stack eg region
    - transfer - include snippets sitting outside the template , specify SAM 
    - resources - mandatory - resources you want to deploy or create
    - outputs - on console or input to another stack
- SAM - serverless app model - for serverless provisionings
    - sam-package - input yaml output sam-template.yaml
    - sam-deploy
    - store template / build on S3
- **nested stack** - template referenced in another template
    - in the resources section
- **delete entire stack on failure**
    - to prevent set up "--disable-rollback" flag in CLI and/or "Rollback on failure" to NO on console
- intrinsic functions
    - !FindInMap - returns value based on mapping section
    - !Ref - value of param or resource
    - !GetAtt - attribute of a resource
    - !Sub - substitutes variables in input string
- to reference VPC stack in another stack
    - create a cross-stack reference and use the Export output field to flag the value of VPC from the network stack
    - Then use Fn::ImportValue intrinsic function to import the value of VPC into the web application stack
- !FindInMap [ MapName, TopLevelKey, SecondLevelKey ] syntax
- Exported Output Values in CloudFormation must have unique names within a single Region
- template on s3
- for lambda - u can define whole inline OR zip the whole in S3 and ref that (AWS::Lambda::Function)
- code : ->zipFile : -> inline code for lambda

## Kinesis - for streaming data
- 3 services
     - streams - in shards 
        - retention 24h-7days 
        - rapidly move data off producers 
        - realtime processing and analysing
     - firehose 
        - no retention - automated shards 
        - easiest way to load streaming data into data stores and analytics tools
        - sink types/destinations - S3, Redshift, ElasticSearch, Splunk
     - analytics - run sql queries on data (using firehose)
- massively scalable - can fan out to various endpoints

## CI/CD Services
- code commit - git repo in aws - src control
    - **set notifications - via cloud watch or SNS**
    - repos are automatically encrypted
    - create cc repo - clone git repo - push to cc repo
    - updateDefaultBranch - only for branch name
    - getBranch - to get details of the branch
- code build - compile build test
    - set timeouts if takes too long
    - to debug - run locally using CodeBuild Agent
    - you cannot ssh into docker where code build runs
    - if docker push/pull issues -  check IAM permissions
    - buildspec.yml
- code deploy - deploy on EC2, lambda etc
    - in place - rolling update
    - blue/green - immutable
    - warm standby - DR scenario
    - pilot light - DR scenario
    - cannot provision , can only deploy
- code pipeline - release management - orchestration
    - **if sees test failure - it will stop**
    - can watch S3 bucket for new zip/jar/war
    - stops immediately if one stage fails
    - appsec.yml
- **Terminology**
    - deployment group
    - deployment config - **AppSec file - yaml(EC2) - yaml or json (lambda)**
    - revision = artifact
    - application = [ group + config + revision]
- **Hooks - defines actions to be done at specific points in the deployment lifecyle**
    - application start vs validate service
    - before allow traffic vs allow traffic vs after allow traffic
- canary vs linear deployment - both can shift traffic in parts - but later uses only version

## Cloud Watch - monitoring and logs
- **monitors performance of resources and reports metrics**
- by default logs stored forever - even after termination
- **in EC2 - by default monitors host level metrics**
    - n/w, overall disk IO, CPU, status check
    - canNOT see how much space left
- **in code build** -> no of builds / success failure / duration etc
- in SQS - ApproximateNumberOfMessageVisible
- default monitoring
    - in EC2 every 5 min
    - others every 1 min
- high resolution - every sec - alarms every 10 sec
- GetMetricsStatistics API
- encryption - even after months - "associate-kms-key" 

## Cloud watch alarm - 1min/2min detailed monitoring

## Cloud Trail - API calls monitoring - resource provisioning
- account-specific history, audit
- by default - logs are encrypted using SSE-S3
- by default it tracks only bucket-level action - to track object-level you need S3 Data events
- Organization trail - log all events for all accounts in the aws org created

## AWS CLI 
- personal machine - aws configure | otherwise use IAM roles 
    - creates config and cred files
- 1 ec2 - 1 role - many policies
- Pagination
    - max-items
    - page-size
    - starting-token
- default page size 1000
    - can give timeout
    - if result size is 2500 - it will make 3 calls internally but show the result altogether
- **use small page size** to avoid timeout
- default max items 100
    - still fetch all, show these many and return
- STS decode-authorization- message - need sts access in role
- when u use roles - it is internally evaluated to temp creds
- account profiles - to use multiple accounts via cli
- MFA with CLI - STS GetSessionToken - temp token for 1 hr
    
## AWS security services
- Shield 
    - on layer 3 and 4
    - protect against distributed denial of service attacks
- WAF
    - SQL injections
    - cross site scripting
    - block IPs based on rules
- Macie
    - data loss prevention
    - uses machine learning to protect sensitive data
    
## Random points
- default region for all SDKs - north virginia - US EAST 1
- Route 53 - the AWS service which does not use key-value pairs
- AWS SSO - single sign on - is a separate service - can be connected to active directory
- x-ray - can be used with - lambda, ELB, API gateway
- to deploy SSL/TLS server certificates - use IAM or AWS Certificate Manager
- Burstable performance instances - T3, T3a, and T2 instances
    - provide a baseline level of CPU performance with the ability to burst to a higher level
    - free for a new/recent account, within certain limit
- IAM supports only one type of resource-based policy - a role trust policy
    - define who you trust to use this resource (AssumeRole)
- ACL - to control which principals in another account can access a resource
- from one default region - to execute a command to stop an EC2 instance in another region 
    - use --region parameter    
    
- AWS Systems Manager - 
    - group systems like EC2, S3, RDS by application
    - view operational data for monitoring and troubleshooting
    - take action on your groups of resources
- Config - logs - resource-specific history, audit, and compliance

- SAM - open-source framework for building serverless applications
    - AWS::Serverless::Api - for api gateway
    - AWS::Serverless::Application
    - AWS::Serverless::Function - for lambda
    - AWS::Serverless::HttpApi
    - AWS::Serverless::LayerVersion
    - AWS::Serverless::SimpleTable - for dynamoDB
    - AWS::Serverless::StateMachine
- AWS Secrets Manager
    - easily rotate, manage, and retrieve 
    - database credentials, API keys, and other secrets
- Cannot use IAM creds for code commit
- AWS requires approximately 5 weeks of usage data to generate budget forecasts
- to use AWS budget, you need NOT be part of AWS organisation

- Access Advisor feature on IAM console - to identify unused roles
- AWS trusted advisor - help provision resources (following AWS best practices on cost, security, fault tolerance, service limits, and performance)
- IAM Access Analyser - identify resources that are shared with external entity
- Amazon inspector - for security and compliance of all apps deployed

- to limit permissions - AWS org service control policy (SCP), permission boundary
- Access Keys = id + secret 
    - for CLI and API
- Key pairs - private and public - digital signature for ssh-ing into EC2
- to use billing and cost mngmt - you just need access - not root user or anything else
- dedicate host vs dedicated instances
    - DI - on VPCs - hardware is dedicated to you - cheaper
    - DH - SERVER dedicated to your use - costly
- IAM Database authentication works with MySQL and PostgresSQL
- when data goes from 1 AZ to another AZ -> very costly in AWS
- private vs public vs elastic IP (only 5)
- security groups are region locked - has only allow rules
- load balancers are region locked
- S3 and IAM both are global services
- ec2 metadata can be accessed without any role

- access aws - 3 ways - console, cli, sdk
- how cli resolves creds
    - command line
    - env var
    - cli credentials file
    - config file
    - container creds
    - instance profiles / roles
- how sdk resolves creds
    - env var
    - java system prop
    - credentials file
    - ecs
    - instance profiles / roles

- A service role is the IAM role that Elastic Beanstalk assumes when calling other services on your behalf.
    - EC2 service role with readonly access to S3 bucket - assigned to EC2
- S3 has no cache - if role/permissions are removed, takes affect immediately
- WCU RCU should never be less - choose more if exact not there
- backend and cache CANNOT be updated in a transaction - at the same time - so invalidate it
- when need to use SSH key pair across region - generate public key and import in all region instances
- fargate and lambda both are serverless - hence when dealing with rest API, you they alone not sufficient, you need API Gateway

- AWS greengrass
- AWS AppSync
- Websockets
- GraphQL - graph query language

## some architectures
- LAMP stack - linux, apache server, mysql, php
- 3 tier - route53, elb(public subnet) , ec2 on private subnet , rds and elastic cache on data subnet
- wordpress - route 53, elb, ec2, efs