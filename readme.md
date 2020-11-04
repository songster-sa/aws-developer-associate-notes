# AWS Developer Associate Exam Notes 2019-2020

Based on Udemy course by Ryan, own research and practise tests.
The topics on which most questions are based are highlighted in bold.

## IAM - Identity Access Manager
- Users, roles, groups, policy document, secret key and access key
- Roles - provide permissions to resources / services


## EC2 - Elastic Compute Cloud
- types - FIGHT DR MC PIX
- **Price options**
    - on demand
    - reserved
    - spot
    - dedicated hosts
- provisions via images
- max of tags allowed 50
- IAAS - infrastructure as a service

    
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

## Elastic BeanStalk
- PAAS - provisioning as a service

## ECS / ECR - elastic containers service / elastic containers registry

## Serverless computing - Lambda and friends
- event driven, API driven
- node.js, python, java, c#, go
- scale out, not scale up
- lambda functions have to be independent
- use X-ray to debug
- API vs API gateway
    - using API gateway we can configure which event triggers which serverless service
    - urls, http methods, SSL/TLS, targets
- API caching = API response caching
- browsers cache against same origin - so AWS uses CORS to change origin (to refresh cache)

       