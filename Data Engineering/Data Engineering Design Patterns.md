# [Data Engineering Design Patterns](https://www.oreilly.com/library/view/data-engineering-design/9781098165826/)

**Disclaimer**
This document is a personal summary and interpretation of the book "Data Engineering Design Patterns" by Bartosz Konieczny. The content provided here is intended for educational and informational purposes only. It is not a substitute for reading the original book and does not include any verbatim text from the book, except for brief quotations used under the principles of fair use.

I do not claim ownership of the book's ideas, and all rights remain with the author and publisher. If you find this summary helpful, I encourage you to support the author by purchasing the original work.

## Data Ingestion Design Patterns

### Full load
Pattern: Full loader
Extract Load(EL) jobs a.k.a pass through jobs are suitable for
1. slowly evolving entities and
2. low volumes and
3. sources that don't track updated timestamps to allow for incremental ingestion pattern

Considerations
1. Data Volume - scaling infrastructure for steadily increasing data volume should be accounted for.
2. Data Consistenty - consider views for downstream and blue-green approach for full publication. Rollback support to be planned for invalid publications. Delta Lake is a great way to ensure atomicity and consistency of data for consumers.


### Incremental load
Pattern: Incremental loader
1. Use a delta column in source to identify records for incremental load from last successful ingestion.
2. Time-partitioned datasets allow identifying incremental data based on new partitions since last successful ingestion.

Considerations
1. Data mutability
2. Recogniziing deletes in source. Soft deletes may be a solution to manage deleted as updates.
3. Limiting backfill jobs for backfill/reconcoliation to fewer partitions to allow for horizontal scaling of ingestion jobs.


Pattern: Change Data Capture
1. Low latency sync between source and target sinks. Use CDC at source to identify changes including insert, updates and deletes. Soft deletes as a workaround for identifying deletes and manage them as updates not required.

Considerations
1. Historical changed data feed after client starts situation may require combining with other ingestion patterns. 
2. Debezium is a very open source tool that support CDC for relational and NoSQL databases and uses Kafka Connect as the bridge between data at-rest and data in-motion.
3. Delta Lake CDF is very simple and powerful. CDFs can identify insert, updates and deletes with additional columns including change_type. For updates, one row for each update_preimage and update post_image is available.    

### Replication
Pattern: Passthrough replicator
Pattern: Transformation replicator

### Data compaction
Pattern: Compactor

### Data readiness
Pattern: Readiness marker
### Event-driven
Pattern: External trigger


## Error Management Design Patterns

## Idempotency Design Patterns
### Overwriting

### Updates

### Database

### Immutable Dataset


## Data Value Design Patterns

## Data Flow Design Patterns

## Data Security Design Patterns

## Data Storage Design Patterns
### Partitioning
While adding compute might help with reducing query runtimes for smaller datasets, a more practical way is to optimize the storage layer. Using metadata for data pruning in favor of scans is something tools like Delta Lake can help with.
Pattern: Horizontal Partitioner
Partitioning is widely implemented in distributed frameworks like Spark, SQL databases and streaming storage solutions like Kafka.

Considerations:
1. Too many partition values can cause metadata overhead. Use bucket design pattern for high cardinality columns.
2. Static nature of partitions - partitions should be static and not change frequently. Requires data rewrites when downstream read patterns change.

Pattern: Vertical Partitioner
Vertical partitioning is a way to split a table into multiple tables with fewer columns. Can be useful in scenarios where life cycle and mutability of columns are different.

Considerations:
1. Query performance - vertical partitioning can help with query performance by reducing the number of columns scanned.
2. Fragmentation - vertical partitioning can lead to fragmentation of data and increase the number of joins required in queries.

Pattern: Delta Lake Liquid Clustering
Alternative to partitioning and bucketing, Delta Lake Liquid Clustering is a way to optimize data storage by clustering data based on the most frequently used columns. Provides flexibility to change clustering columns without rewriting data.

Considerations:
1. Limits on number of clustering columns - Delta Lake supports up to 32 clustering columns.

### Records Organization
Pattern: Bucket
Bucketing is a way to organize data in storage by grouping records into buckets based on a hash function. Useful for optimizing query performance by reducing the number of records scanned.

Considerations:
1. Bucketing columns - choose columns with high cardinality for better distribution of records across buckets.
2. Bucketing schema is immutable - changing bucketing schema requires rewriting data.
3. Identifying right bucket size is a challenge - too many buckets can lead to metadata overhead, while too few buckets can lead to data skew. 
4. Changing bucket size during write `df.write.bucketBy(8, "columnName")` in spark might lead to inconsistent bucketing schema for new vs. old data. Delta Lake can help with this by tracking bucketing schema changes and error out. `org.apache.spark.sql.AnalysisException: The bucket specification does not match the existing table definition.` 
5. Bucketing schema should be same for same column in multiple tables to allow for joins optimization and to avoid unnecessary data issues. 

Pattern: Sorter
Z-ordering is a way to organize data in storage by sorting records based on multiple columns. Useful for optimizing query performance by reducing the number of records scanned.

Considerations:
Similar to partitions, Z-order indexes will soon be replaced by the new Delta Lake feature, liquid clustering, as the preferred technique to simplify data layout and optimize query performance. Delta Lake liquid clustering is not compatible with user-specified table partitions or Z-ordering.

### Read Performance Optimization
Pattern: Metadata Enhancer
Delta Lake collects statistics about data stored in the table and uses them to optimize query performance. Delta Lake statistics include the number of distinct values, minimum and maximum values, and the number of null values for each column. This improves query performance by allowing the query optimizer to make better decisions

Considerations:
1. Delta Lake statistics are collected automatically when writing data to a Delta Lake table.
2. Overhead - collecting statistics can add overhead to write operations. Delta Lake allows you to disable statistics collection by setting the `spark.databricks.delta.stats.autoUpdate.enabled` configuration to `false`.

Pattern: Dataset Materializer
Materialized views are precomputed views that store the results of a query in a table. Materialized views can be used to optimize query performance by reducing the amount of data that needs to be scanned.

Considerations:
1. Incremental refresh - Delta Lake provides a `MERGE INTO` command that can be used to incrementally refresh materialized views by only updating the rows that have changed since the last refresh. Of course this requires a change tracking mechanism in place using some timestamp column or change data capture mechanism.
2. Data access permissions - ensuring users have access to view only if they have access to underlying data. Fine-grained access control can be implemented.
3. Storage Overhead - materialized views can add storage overhead. 

Pattern: Manifest
Delta Lake manifest files are used to track the list of data files that belong to a Delta Lake table. Manifest files are used to optimize query performance by allowing the query optimizer to skip reading unnecessary data files.

Considerations:
1. Manifest files are automatically created when writing data to a Delta Lake table.
2. Overhead - manifest files can add overhead to write operations. Delta Lake allows you to disable manifest file creation by setting the `spark.databricks.delta.manifest.enabled` configuration to `false`.

### Data Representation
Pattern: Normalizer
Normalization Forms:
Here's a quick summary of the main **Normalization Forms (NF)** in database design:

1. **First Normal Form (1NF)**:
    - Ensure atomic values (no lists or arrays in a single column).
    - Each row has a unique identifier.
2. **Second Normal Form (2NF)**:
    - Meet 1NF.
    - Eliminate partial dependencies (non-key attributes must depend on the entire primary key).
3. **Third Normal Form (3NF)**:
    - Meet 2NF.
    - Eliminate transitive dependencies (non-key attributes should depend only on the primary key, not other non-key attributes).
4. **Boyce-Codd Normal Form (BCNF)**:
    - Meet 3NF.
    - Ensure that every determinant is a candidate key (addresses anomalies not covered by 3NF).
5. **Fourth Normal Form (4NF)**:
    - Meet BCNF.
    - Eliminate multi-valued dependencies (a record should not have two or more independent multi-valued facts).
6. **Fifth Normal Form (5NF)**:
    - Meet 4NF.
    - Decompose tables to eliminate redundancy from join dependencies while preserving data integrity.

Considerations:
1. Normalization can lead to complex queries and joins.
2. Archival and historical data - normalized data can be difficult to manage for archival and historical data. 
3. Denormalization can be used to optimize read performance. 
4. Delta Lake supports both normalized and denormalized data representations. 
5. Delta Lake supports ACID transactions for both normalized and denormalized data representations.

Pattern: Denormalizer
Denormalization is the process of combining normalized tables into a single table to optimize query performance. Denormalization can be used to reduce the number of joins required in queries and improve query performance.
Array columns in Delta Lake can be used to store nested data structures in a single column. Array columns can be used to denormalize data and improve query performance.
Struct columns in Delta Lake can be used to store nested data structures in a single column. Struct columns can be used to denormalize data and improve query performance.

Considerations:
1. Data consistency - denormalization can lead to data redundancy and inconsistency. Delta Lake provides ACID transactions to ensure data consistency.
2. One Big Table Anti-pattern - denormalization can lead to the creation of a single large table that contains all the data in the system. Try to give the table a name and if it uses a lot of conjunctions, it's a sign that it's too big.


## Data Quality Design Patterns

## Data Observability Design Patterns





