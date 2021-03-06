# AWS Data Stores

## General

* Types of data stores
	- Persistent
		- e.g. RDS, Glacier
	- Transient
		- e.g. SQS, SNS
	- Ephemeral
		- Memcached, EC2 Instance Store
* IOPS vs Throughput
	- IOPS ("sports car"): how fast can you read/write to/from the device
		- i.e. I/O operations per second
	- Throughput ("dump truck"): how much data can be moved
* Consistency Models
	- ACID
		- Atomicity
			- Transactions are all or nothing
		- Consistency
			- Transactions must be valid (e.g. key constraints respected)
		- Isolation
			- Transactions can't interact with each other
		- Durability
			- Completed transaction must persist
	- BASE
		- Basically available: the system guarantees availability, in terms of the CAP theorem
		- Soft state: the state of the system may change over time, even without input (due to eventual consistency model)
		- Eventual consistency: the system will become consistent over time
	- CAP theorem
		- States that a distributed system cannot guarantee the following three properties at the same time:
			- Consistency
			- Availability
			- Partition tolerance
		- BASE system is (Availability, Partition tolerance)
		- ACID system is (Consistency, Availability)


## [Amazon Simple Storage Service (S3)](../../deep-dives/s3/)

## Glacier

* Glacier Vault
	- like a bucket
* Glacier Archive
	- like an object
	- file, zip, tar, etc
	- max size 40TB
	- immutable
* Glacier Vault Lock
	- different than vault access (IAM) policy
	- enforce rules like no deletes or MFA
	- immutable
	- two step process:
		- 1. initiate vault lock
		- 2. 24 hr period in which to either abort or complete
* Access policies
	- Via IAM


## EBS

* Tied to a single AZ
* EBS snapshots
	- cost-effective and easy backup strategy
	- share data sets with other users or accounts
	- migrate system to new AZ or region
	- convert unencrypted to encrypted volume
		- take snapshot of unencrypted volume
		- restore snapshot to new volume, choosing to enable encryption
* Data Lifecycle Manager
	- schedule snapshots for volumes or instances every X hours
	- provides retention rules to remove stale snapshots
* EBS and RAID
	- Types of RAID
		- RAID0 (striping)
		- RAID 1 (mirroring)
		- RAID 5 (fault-tolerance via parity bits)
			- 2 drives store data, 1 drive does parity bit
			- can lose 1 drive and rebuild data
			- writing parity bits can take up to 20-30% of IO
	- NOTE: Because of performance overhead, AWS does *not* recommend RAID5
* EC2 instance store
	- when you stop/terminate instance, it goes away
	- direct attached storage
	- temporary, ideal for caches/buffers


## Elastic File Service (EFS)

* Implementation of NFS 
* Elastic storage capacity and pay for only what you use (in contrast to EBS)
* Multi-AZ metadata and data storage
* Configure mount points in 1 or many AZs
* Can be mounted from on-prem systems
	- careful: NFS not secure protocol
		- you should create VPN tunnel to ensure encryption when going over internet
	- alternatively use Amazon DataSync
* EFS 3x more expensive than EBS, 20x more expensive than S3


## [Storage Gateway](../../deep-dives/storage-gateway/)


## RDS

* Supports the following relational databases:
	- Mysql
		- non-transactional storage engines like MyISAM don't support replication; you must use InnoDB (or XtraDB on Maria)
	- MariaDB
		- open source fork of MySQL
	- Postgres
	- MS SQL Server
	- Oracle
	- Aurora
* Replication
	- Multi-AZ
		- replication between master and standby is sync
	- Read replicas
		- Replication between master and read replica is async
		- NOTE: Read replicas service regional users
* In Multi-AZ
	- if master dies, then one of the standbys is promoted to new master
	- if entire region dies, you can use read replica
		- 1. promote read-replica to stand-alone (single-AZ)
		- 2. single_AZ reconfigured to multi-AZ
* RDS Encryption
    - RDS supports encryption at rest
        - Applies to DB instances, automated backups, read replicas, snapshots
    - Uses AES-256 encryption
    - Uses KMS
        - Master key is used to create encryption keys (i.e. envelope encryption)
    - All logs, backups, and snapshots are encrypted
    - Read replicas are encrypted using same key when both in same region
    - You can only enable encryption when you create the DB instance
    - You cannot modify an encrypted instance to disable encryption
    - You cannot restore an unencrypted backup or snapshot to encrypted DB instance
* RDS Snapshots
    - Copying a snapshot
        - You can copy automated or manual DB snapshots
        - Post copy, you have a manual snapshot
        - Possible copy destinations
            - Copy snapshot within same region
            - Copy snapshot across regions
            - Copy snapshot across AWS accounts
                - If automated snapshot, you first copy to make a manual snapshot
                - Then manual snapshot can be copied to other account
    - Snapshot retention
        - Automated snapshots are deleted when:
            - End of retention period
            - When you disable automated snapshots for DB instance
            - When DB instance is deleted
        - Manual snapshots are retained until you specifically delete
    - Encrypted snapshots
        - If you copy an encrypted snapshot, the copy of the snapshot must also be encrypted
        - Copying encrypted snapshots
            - Within same region, just need access to KMS key
            - Across regions, you must specify KMS key valid in destination region
                - This is because KMS keys are region-specific
        - You can encrypt copy of unencrypted snapshot
            - Allows you to quickly add encryption to a previously unencrypted DB instance
* HOWTO: Add encryption to an unencrypted DB instance
    - First make snapshot of source DB instance
    - Create copy of snapshot and specify encryption with KMS key
    - Restore the encrypted snapshot to new instance 


## [DynamoDB](../../deep-dives/dynamodb/)


## Redshift

* Fully managed clustered peta-byte scale data warehouse
* Very cost-effective compared to other data warehouse platforms
* Postgres compatible with JDBC/ODBC drivers
	- compatible with most BI tools out of the box
* Parallel processing and columnar data stores which are optimized for complex queries
* Does not support multi-AZ deployments
	- For best HA, use multi-node cluster with replication
* Single node cluster does not support replication
	- you have to restore from snapshot on S3 if drive fails
* Option to query directly from data files on S3 via Redshift Spectrum
* Data Lakes
	- Store data in S3
	- Use Redshift Spectrum (or Athena) to query
	- Use QuickSight for visualization


## Elasticache

* Fully managed implementations of Redis and Memcached
* Push button scalability for memory, writes and reads
* In memory key/value store - not persistent in traditional sense
* Billed by node size and hours of use
* Use Cases
	- web session store
		- store session data in Redis
	- database caching
	- leaderboards
	- streaming data dashboards
* Two types
	- Memcached
		- simple, no frills
		- can scale out and scale in as demand changes
		- need to run multiple CPU cores and threads
		- you need to cache objects (like db queries)
		- NOT HA
		- Does *not* support backup/restore
	- Redis
		- Supports encryption
		- HIPAA compliance
		- Support for clustering for HA
			- Automatic failover
			- Replication
		- complex data types
		- pub/sub capability
		- geospatial indexing
		- backup and restore
* Caching Strategies
    - Lazy Loading
        - Loads data into the cache only on read misses
            - App requests data from cache. 
                - If hit, return data.
                - If miss, fetch from database and then write results to cache
        - Pros
            - Only requested data is cached
            - Tolerant to node failures (albeit with performance penalty of having empty cache)
        - Cons
            - Cache misses incur 3 requests (cache read, db read, cache write)
            - Data can be stale
    - Write-Through
        - Adds/updates cache whenever data is written to the database
        - Pros
            - Data is never stale
            - Writes incur 2 requests (cache write, db write)
            - Cache misses incur 2 requests (cache read, db read)
        - Cons
            - Stores more data than necessary (most data won't be read)
                - Can limit with TTLs
            - With node failure, data won't be in cache until a write operation occurs


## Other Database Options

* Athena
	- SQL engine overlaid on S3 using Presto
	- query raw data objects as they sit in S3 bucket
	- can convert data to Parquet format for big perf jump
	- similar in concept to Redshift Spectrum but...
		- use Athena: data lives mostly on S3 w/o need for joins with other sources
		- use Redshift Spectrum: want to join S3 data with existing Redshift tables or create union products
* Quantum Ledger Database
	- based on blockchain
	- provides immutable and transparent journal as a service 
	- centralized design (as opposed to decentralized consensus-based design for common blockchain frameworks) - allows for high perf and scalabillity
	- append only concept where each record contributes to the integrity of the chain
		- when you write a record to the ledger, you use the hash of the previous record to compute hash of new record
		- that's why you cannot delete/change any records - it will invalidate the chain
* Amazon Managed Blockchain
	- fully managed blockchain framework
		- supports either Hyperledger Fabric or Ethereum
	- distributed consensus-based concept consisting of network, members (other AWS accounts), nodes (instances) and potentially apps
	- uses QLDB ordering service to main complete history of all transactions
* Amazon Timestream Database
	- fully managed db for storing/analyzing time-series data
	- alternative to DynamoDB or Redshift and includes some built in analytics like interpolation and smoothing
	- use cases:
		- sensor networks
		- equipment telemetry
* DocumentDB
	- MongoDB-compatible
	- multi-AZ HA, scalable, integrated with KMS, backed up to S3
* ElasticSearch
	- Mostly a search  engine but also a document store
	- Aka ELK stack
		- Kibana (analytics)
		- Logstash (intake)
		- Elasticsearch (search and storage)


## Links

* [Encrypting Amazon RDS Resources](https://docs.aws.amazon.com/AmazonRDS/latest/UserGuide/Overview.Encryption.html)
* [Copying a Snapshot](https://docs.aws.amazon.com/AmazonRDS/latest/UserGuide/USER_CopySnapshot.html)
