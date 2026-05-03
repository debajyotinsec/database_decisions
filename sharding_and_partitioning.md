## Sharding and Partitioning

### Preface
Often when we encounter an data intensive application, we decide to scale up or scale out. In addition we hear two terms `sharding` and `partitioning`.
I want to take a few minutes and answer below questions.

- What is sharding?
- What is partitioning?
- Differences between sharding and paritioning?
- What factors to consider/questions to ask before we take either of the steps.

No matter the step we take each has its own architectural impact and trade offs.


### Sharding
Sharding is a technique where the database is split across multiple database instances or nodes (horizontal data distribution). Data is split in shards, each shard is on a different database instance. Whenever data is inserted, application/router decides which database instance (shard) should hold the data.

Key idea: Scaling beyond one machine.

