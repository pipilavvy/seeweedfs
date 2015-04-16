This page aims to consolidate the pages on the [[single-node filer|Directories and Files]] and [[distributed filer]] into one.

## Background

SeaweedFS comes with a lightweight "filer" server, which provides a RESTful wrapper around SeaweedFS's arbitrary blob API, mapping content to a traditional file directory of paths.

## Backends

SeaweedFS's built-in filer supports three different backends (although pull requests to add more are always welcome).

The default backend, LevelDB, is for simple, non-distributed single nodes.

The other backends, Redis and Cassandra, are for clustering backing stores that can be distributed across several nodes at high scale.

The LevelDB backend is very capable and efficient; the main disadvantage it has, relative to the distributed backends, is that it presents a single point of failure. In "[pets vs. cattle][pvc]" terms, the LevelDB backend is only suitable for "pet" servers, while the Redis and Cassandra backends are suitable for "cattle" servers.

[pvc]: https://blog.engineyard.com/2014/pets-vs-cattle

## Initialization

The LevelDB and Redis backends need no initialization.

### Cassandra backend

Here is the CQL to create the table used by SeaweedFS's Cassandra store, as well as a keyspace for specifying the replication strategy to use.

While the table name and field structure must match what is written here, you are free to rename the keyspace and use whatever replication settings you wish. For production, you would want to set replication_factor to 3
if there are at least 3 Cassandra servers.

```cql
create keyspace seaweed WITH replication = {
  'class':'SimpleStrategy',
  'replication_factor':1
};

use seaweed;

CREATE TABLE seaweed_files (
   path varchar,
   fids list<varchar>,
   PRIMARY KEY (path)
);
```