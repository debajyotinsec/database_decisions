# Sharding and Partitioning

## Preface
Often when we encounter a data intensive application, we decide to scale up or scale out. In addition we hear two terms `sharding` and `partitioning`.
I want to take a few minutes and answer below questions.

- What is sharding?
- What is partitioning?
- Differences between sharding and partitioning?
- What factors to consider/questions to ask before we take either of the steps?

No matter the step we take each has its own architectural impact and trade offs.


## Sharding
Sharding is a technique where the database is split across multiple database instances or nodes (horizontal data distribution). Data is split in shards, each shard is on a different database instance. Whenever data is inserted, application/router decides which database instance (shard) should hold the data.

Key idea: Scaling beyond one machine.


#### Sharding Strategies
- **Hash-based sharding**: Consistent hashing, hash function selection
- **Range-based sharding**: Date ranges, ID ranges, hotspot risks
- **Directory/Lookup-based**: Flexibility vs lookup overhead
- **Geographic sharding**: Data sovereignty, latency optimization
- **Entity-based sharding**: User-based, tenant-based (multi-tenancy)


## Partitioning
Partitioning on the other hand is a technique where a table in a database instance is split across multiple smaller tables, usually on the basis of a partitioning key.

Key idea: Partitioning is about managing large tables efficiently inside one database


### Partitioning Types
- **Range Partitioning**: Time-series data, archival strategies
- **List Partitioning**: Categorical data (regions, status)
- **Hash Partitioning**: Even distribution, limited range queries
- **Composite Partitioning**: Range-hash, range-list combinations


## Difference between Sharding and partitioning
|Metric|Sharding|Partitioning|
|-------------|------------------|-----------------|
|Scope|Across multiple databases|Across tables in a single database|
|Managed by|Application/Router|Database engine|
|Purpose|Horizontal scalability|Performance and maintenance|
|Data presence|Across multiple machines|On one machine|
|Joins|Hard across shards|Easy, as it is same database|


### Challenges
**_Sharding:_**
- Cross-shard queries and joins (scatter-gather pattern)
- Distributed transactions (2PC, Saga pattern)
- Resharding/rebalancing (downtime, data migration)
- Shard key selection (immutable, high cardinality)
- Hotspot shards (celebrity problem)
- Schema changes across shards
- Backup and recovery complexity

**_Partitioning:_**
- Partition pruning effectiveness
- Partition key selection (query patterns alignment)
- Partition maintenance (DROP vs DELETE for archival)
- Global indexes vs local indexes
- Partition-wise joins



## Have we asked the right questions?
Before we embark on a database slicing dicing journey, we must assess the current state and what the future looks like.
I would ask the below questions:
- Data scale and growth.
    - What is the current database size?
    - How quickly is it growing?
    - Can it fit in a single database instance enough?
    - How does data retention look like?

- Read/Write patterns
    - Is your application read-heavy or write-heavy?
    - If read-heavy:
        - have you considered creating a read-replica?
        - can you cache some of the information? for eg. redis, memcached
    - If write-heavy:
        - is batch writes an option?
        - can you dump the data in a queue and a listener writes to the database once?


- Query patterns
    - Do most of the select queries use primary keys?
    - How many queries are not using primary keys or other indexes?

- Latency requirements
    - Are the users distributed globally?
    - Is low latency a global requirement?

- Operational overhead and maintenance 
    - Added complexity for deployment and monitoring
    - Team should be mature enough to handle such databases
    - Do we have the right tools?
    - What are the backup strategies? How does regular maintenance look like?

- Consistency and Failure isolation
    - CAP theorem implications
    - Is eventual consistency an option?

- Future scaling
    - Resharding and repartitioning strategies and data migration
    - Growth projections


### Anti-patterns
- Premature optimization (can you vertically scale first?)
- Poor shard key selection leading to uneven distribution
- Over-partitioning (too many small partitions)
- Sharding when read replicas + caching would suffice


## How different database engines handle partitioning?
**MySQL:**
- Partition pruning with EXPLAIN PARTITIONS
- Limitations: Foreign keys with partitioning
- InnoDB native partitioning, requires partition key in primary key

**PostgreSQL:**
- Declarative partitioning (11+) vs inheritance
- Partition-wise joins and aggregates
- pg_partman for automated partition management
- Global indexes don’t exist (before v15; even later limited)
- Index is local per partition. You may end up with duplicate primary key across partition if not designed properly.

**Oracle:**
- Partition exchange loading (fast bulk loads)
- Interval partitioning (automatic partition creation)
- Partition advisor for recommendations
- Supports global and local indexes, thus enforces primary key across partitions.

