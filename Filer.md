This page aims to consolidate the pages on the [[single-node filer|Directories and Files]] and [[distributed filer]] into one.

## Background

SeaweedFS comes with a lightweight "filer" server, which provides a RESTful wrapper around SeaweedFS's arbitrary blob API, mapping content to a traditional file directory of paths.

## Backends

SeaweedFS's built-in filer supports three different backends (although pull requests to add more are always welcome).

The default backend, LevelDB, is for simple, non-distributed single nodes.

The other backends, Redis and Cassandra, are for clustering backing stores that can be distributed across several nodes at high scale.
