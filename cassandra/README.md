# Cassandra

## Overview

- partitioned wide column storage model
- eventually consistent
- scalable, reliable and highly available
- Full multi-master database replication
- Global availability at low latency
- Scaling out on commodity hardware
- Linear throughput increase with each additional processor
- Online load balancing and cluster growth
- Partitioned key-oriented queries
- Flexible schema

## Features

CQL CQL allows users to organize data within a cluster of Cassandra nodes using:

- Keyspace: defines how a dataset is replicated.
- Table: defines the typed schema for a collection of partitions.
- Partition: defines the mandatory part of the primary key all rows in Cassandra must have.
- Row: contains a collection of columns identified by a unique primary key made up of the partition key and optionally additional clustering keys.
- Column: A single datum with a type which belong to a row.

Cassandra does not support:

- Cross partition transactions
- Distributed joins
- Foreign keys or referential integrity.

## Operating

- cassandra.yaml
- nodetool
- `auditlogviewer` is used to view the audit logs.
- `fqltool` is used to view, replay and compare full query logs.

## Dynamo

https://cassandra.apache.org/doc/latest/architecture/dynamo.html#dynamo

### Dataset Partitioning: Consistent Hashing

- Token Ring <https://pdos.csail.mit.edu/papers/chord:sigcomm01/chord_sigcomm.pdf>
- Multiple Tokens per Physical Node (a.k.a. vnodes)

Cassandra introduces some nomenclature to handle these concepts:

- Token: A single position on the dynamo style hash ring.
- Endpoint: A single physical IP and port on the network.
- Host ID: A unique identifier for a single “physical” node, usually present at one Endpoint and containing one or more Tokens.
- Virtual Node (or vnode): A Token on the hash ring owned by the same physical node, one with the same Host ID.

### Multi-master Replication: Versioned Data and Tunable Consistency

Replication Strategy:

- NetworkTopologyStrategy
- SimpleStrategy

Tunable Consistency:

- ONE, TWO, THREE, QUORUM, ALL
- LOCAL_QUORUM
- EACH_QUORUM
- LOCAL_ONE
- ANY

https://cassandra.apache.org/doc/latest/architecture/dynamo.html#picking-consistency-levels
W + R > RF, If QUORUM is used for both writes and reads, at least one of the replicas is guaranteed to participate in both the write and the read request, which in turn guarantees that the quorums will overlap and the write will be visible to the read.

### Distributed Cluster Membership and Failure Detection

Gossip

1. Updates the local node’s heartbeat state (the version) and constructs the node’s local view of the cluster gossip endpoint state.
2. Picks a random other node in the cluster to exchange gossip endpoint state with.
3. Probabilistically attempts to gossip with any unreachable nodes (if one exists)
4. Gossips with a seed node if that didn’t happen in step 2.

Ring Membership and Failure Detection
Every node in Cassandra runs a variant of the Phi Accrual Failure Detector, in which every node is constantly making an independent decision of if their peer nodes are available or not. This decision is primarily based on received heartbeat state.

## Storage Engine

- CommitLog
- Memtables
- SSTables https://cassandra.apache.org/doc/latest/architecture/storage_engine.html#sstables

## Guarantees

- High Scalability
- High Availability
- Durability
- Eventual Consistency of writes to a single table
- Lightweight transactions with linearizable consistency
- Batched writes across multiple tables are guaranteed to succeed completely or not at all
- Secondary indexes are guaranteed to be consistent with their local replicas data

## Evaluating and Refining Data Models

- `N_v = N_r (N_c - N_{pk} - N_s) + N_s`

## Tombstones

https://cassandra.apache.org/doc/latest/operating/compaction/index.html#tombstones-and-garbage-collection-gc-grace

- This approach is used instead of removing values because of the distributed nature of Cassandra.
- The table level `gc_grace_seconds` parameter controls how long Cassandra will retain tombstones through compaction events before finally removing them.

## User case

- chat room
- you may alos like

## Interview questions

Q: What is Cassandra?
A: Apache Cassandra is an open source, distributed, NoSQL database. It presents a partitioned wide column storage model with eventually consistent semantics.

Q: How to do data modling?
A: First, we can do `Conceptual Data Modeling`. We create a simple domain model. Second, define the application queries. And then, do `logical data modling`. Logical model containing a table for each query, capturing entities and relationships from the conceptual model. Final, do `Physical Data Modeling`. walk through each of the logical model tables, assigning types to each item. Then we analyze the model by performing size calculations and testing out how the model works.

Q: How Cassandra organize data?
A: Cassandra has 6 scope of data. From top to bottom, there are cluster, keyspace, table, partition, row and colunm. Keyspace defines how a dataset is replicated. Rows contains a collection of columns identified by a unique primary key.

```cql
CREATE KEYSPACE [ IF NOT EXISTS ] keyspace_name
  WITH REPLICATION = { replication_map }
  [ AND DURABLE_WRITES = ( true | false ) ] ;


CREATE TABLE [ IF NOT EXISTS ] [keyspace_name.]table_name
  ( column_definition [ , ... ] | PRIMARY KEY (column_list) )
  [ WITH [ table_options ]
  [ [ AND ] CLUSTERING ORDER BY [ clustering_column_name order ] ]
  [ [ AND ] ID = 'table_hash_tag' ] ] ;
```

Q: How Cassandra distribute data?
A: Cassandra use consistent hashing to map every node to one or more tokens on a continuous hash ring. But with evenly spaced tokens and a small number of physical nodes, incremental scaling is difficult because there are no token selections for new nodes that can leave the ring balanced. Cassandra use virtual nodes to solve imbalance problem
