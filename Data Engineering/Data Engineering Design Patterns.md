# [Data Engineering Design Patterns](https://www.oreilly.com/library/view/data-engineering-design/9781098165826/)

**Disclaimer**
This document is a personal summary and interpretation of the book "Data Engineering Design Patterns" by Bartosz Konieczny. The content provided here is intended for educational and informational purposes only. It is not a substitute for reading the original book and does not include any verbatim text from the book, except for brief quotations used under the principles of fair use.

I do not claim ownership of the book's ideas, and all rights remain with the author and publisher. If you find this summary helpful, I encourage you to support the author by purchasing the original work.

## Data Ingestion Design Patterns

### Full load
#### Pattern: Full loader  
Extract Load(EL) jobs a.k.a pass through jobs are suitable for  
1. slowly evolving entities and
2. low volumes and
3. sources that don't track updated timestamps to allow for incremental ingestion pattern

Considerations:  
1. Data Volume - scaling infrastructure for steadily increasing data volume should be accounted for.
2. Data Consistenty - consider views for downstream and blue-green approach for full publication. Rollback support to be planned for invalid publications. Delta Lake is a great way to ensure atomicity and consistency of data for consumers.


### Incremental load
#### Pattern: Incremental loader  
1. Use a delta column in source to identify records for incremental load from last successful ingestion.
2. Time-partitioned datasets allow identifying incremental data based on new partitions since last successful ingestion.

Considerations:  
1. Data mutability is a challenge for incremental ingestion. 
2. Recogniziing deletes in source. Soft deletes may be a solution to manage deleted as updates.
3. Limiting backfill jobs for backfill/reconcoliation to fewer partitions to allow for horizontal scaling of ingestion jobs.

#### Pattern: Change Data Capture
1. Low latency sync between source and target sinks. Use CDC at source to identify changes including insert, updates and deletes. Soft deletes as a workaround for identifying deletes and manage them as updates not required.

Considerations:  
1. Historical changed data feed after client starts situation may require combining with other ingestion patterns. 
2. Debezium is a very open source tool that support CDC for relational and NoSQL databases and uses Kafka Connect as the bridge between data at-rest and data in-motion.
3. Delta Lake CDF is very simple and powerful. CDFs can identify insert, updates and deletes with additional columns including change_type. For updates, one row for each update_preimage and update post_image is available.    

### Replication
#### Pattern: Passthrough replicator
Replicating data from source to target without any transformation. Useful for scenarios where data is needed in multiple locations or for backup purposes.

Considerations:  
1. Data Consistency - ensure that data is replicated consistently across all target systems.
2. Sensitive data - ensure that sensitive data is encrypted during replication to protect it from unauthorized access.
3. Use built-in replication tools like AWS s3 replication, Kafka MirrorMaker, etc. for replicating data across different systems where possible.

#### Pattern: Transformation replicator
Problem: Making production data available in staging and development environments for testing and development purposes but PII data should be masked.

Replicating data from source to target with transformations applied. Useful for scenarios where data needs to be transformed before being replicated to target systems.

Considerations:
1. Data integrity is a challenge for transformation replicators.
2. Drop unnecessary columns and mask sensitive data.

### Data compaction
#### Pattern: Compactor
Problem: Small file problem especially from streaming producers causing metadata overhead.

Compacting data files to reduce the number of files and improve query performance. Useful for scenarios where data is produced in small files and needs to be compacted for better performance. Delta Lake OPTIMIZE command can be used to compact small files.

Considerations:
1. Data consistency - ensure that data is not lost during compaction.
2. Compaction might leave old files as is leading to more metadata overhead. Delta Lake OPTIMIZE command can be used to compact small files and remove old files. Additionally Delta Lake VACUUM command can be used to remove old files.

### Data readiness
#### Pattern: Readiness marker  
Problem: Ensuring data is ready for consumption before allowing downstream systems to access it.  

Delta Lake implements a transaction log that records all changes to the table. The transaction log is used to ensure that data is ready for consumption before allowing downstream systems to access it. Delta Lake provides ACID transactions to ensure data consistency.  

Considerations:
1. Late data updating previously consumed data can be a challange. Consider making partitions immutable after they are consumed.
2. Consider creating custom readiness markers to track data readiness for downstream systems. Something similar to Spark's _SUCCESS file can be used to track data readiness.

### Event-driven
#### Pattern: External trigger
Problem: Schedule-based ingestion jobs may result in compute resources being utilized even when there is no data to process. 

Triggering ingestion jobs based on external events. Useful for scenarios where data is produced sporadically and needs to be ingested as soon as it is available. Delta Lake provides support for event-driven ingestion using the Delta Lake event log.

Considerations:
1. Push vs. pull - consider using push-based ingestion to reduce latency and improve data freshness.
2. Execution context in trigger - ensure that the trigger has the necessary context to execute the ingestion job. Delta Lake provides support for event-driven ingestion using the Delta Lake event log.
3. Error management - ensure that errors are handled gracefully and that the ingestion job can be retried in case of failure.


## Error Management Design Patterns
### Unprocessable Records
#### Pattern: Dead Letter
Problem: Handling records that cannot be processed due to data quality issues or schema violations.

Identify problematic data records and add them to a dead letter queue for further processing using another instance of the pipeline. Useful for scenarios where data quality issues or schema violations prevent records from being processed.

Considerations:  
1. Snowball backfilling effect - when bad records are dead-lettered to allow partial outputs, downstream consumers would process partial data. When reprocessing happens upstream, the downstream consumers would process the same data again leading to snowball effect.
2. Ordering and consistency: Maintaining order and consistency of records between successful and dead-lettered records is a challenge.
3. Error safe functions: Use error safe functions to handle errors gracefully and avoid job failures. However this might lead to partial data being processed as well as actual exceptions being masked.

### Duplicated Records
#### Pattern: Windowed Deduplicator  
Problem: In distributed systems, data producers might produce data to achieve at-least once delivery semantics. This can lead to duplicate records being ingested into the data lake. However on the consumption side we might have a need to process each record only once.

Maintain state, typically in a state store, to track processed items and avoid processing duplicates. Useful for scenarios where data producers might produce duplicate records and consumers need to process each record only once.   
State store can be:  
1. Local
2. Local with fault tolerance by committing to remote store regularly.
3. Remote store, typically a low-latency store like DynamoDb, Redis, Cassandra, etc.

Considerations:  
1. Space vs. time tradeoff - maintaining state for deduplication can lead to increased storage costs. Use TTLs to remove old state. On the flip side reduced windos size can lead to missing some duplicates.
2. If the consumer is also a data producer for downstream, exactly-once processing does not necessarily guarantee exactly-once delivery. This is a challenge in distributed systems.  

### Late data  
#### Pattern: Late Data Detector


### Filtering

### Fault-Tolerance

## Idempotency Design Patterns
### Overwriting

### Updates

### Database

### Immutable Dataset

## Data Value Design Patterns
### Data Enrichment

### Data Decoration

### Data Aggregation

### Sessionization

### Data Ordering

## Data Flow Design Patterns
### Sequence

### Fan-In

### Fan-Out

### Orchestration

## Data Security Design Patterns
### Data Removal

### Access Control

### Data Protection

### Connectivity

## Data Storage Design Patterns
### Partitioning
While adding compute might help with reducing query runtimes for smaller datasets, a more practical way is to optimize the storage layer. Using metadata for data pruning in favor of scans is something tools like Delta Lake can help with.
#### Pattern: Horizontal Partitioner
Partitioning is widely implemented in distributed frameworks like Spark, SQL databases and streaming storage solutions like Kafka.

Considerations:  
1. Too many partition values can cause metadata overhead. Use bucket design pattern for high cardinality columns.
2. Static nature of partitions - partitions should be static and not change frequently. Requires data rewrites when downstream read patterns change.

#### Pattern: Vertical Partitioner  
Vertical partitioning is a way to split a table into multiple tables with fewer columns. Can be useful in scenarios where life cycle and mutability of columns are different.

Considerations:  
1. Query performance - vertical partitioning can help with query performance by reducing the number of columns scanned.
2. Fragmentation - vertical partitioning can lead to fragmentation of data and increase the number of joins required in queries.

#### Pattern: Delta Lake Liquid Clustering
Alternative to partitioning and bucketing, Delta Lake Liquid Clustering is a way to optimize data storage by clustering data based on the most frequently used columns. Provides flexibility to change clustering columns without rewriting data.

Considerations:
1. Limits on number of clustering columns - Delta Lake supports up to 32 clustering columns.

### Records Organization
#### Pattern: Bucket
Bucketing is a way to organize data in storage by grouping records into buckets based on a hash function. Useful for optimizing query performance by reducing the number of records scanned.

Considerations:
1. Bucketing columns - choose columns with high cardinality for better distribution of records across buckets.
2. Bucketing schema is immutable - changing bucketing schema requires rewriting data.
3. Identifying right bucket size is a challenge - too many buckets can lead to metadata overhead, while too few buckets can lead to data skew. 
4. Changing bucket size during write `df.write.bucketBy(8, "columnName")` in spark might lead to inconsistent bucketing schema for new vs. old data. Delta Lake can help with this by tracking bucketing schema changes and error out. `org.apache.spark.sql.AnalysisException: The bucket specification does not match the existing table definition.` 
5. Bucketing schema should be same for same column in multiple tables to allow for joins optimization and to avoid unnecessary data issues. 

#### Pattern: Sorter  
Z-ordering is a way to organize data in storage by sorting records based on multiple columns. Useful for optimizing query performance by reducing the number of records scanned.

Considerations:  
Similar to partitions, Z-order indexes will soon be replaced by the new Delta Lake feature, liquid clustering, as the preferred technique to simplify data layout and optimize query performance. Delta Lake liquid clustering is not compatible with user-specified table partitions or Z-ordering.

### Read Performance Optimization
#### Pattern: Metadata Enhancer  
Delta Lake collects statistics about data stored in the table and uses them to optimize query performance. Delta Lake statistics include the number of distinct values, minimum and maximum values, and the number of null values for each column. This improves query performance by allowing the query optimizer to make better decisions

Considerations:  
1. Delta Lake statistics are collected automatically when writing data to a Delta Lake table.
2. Overhead - collecting statistics can add overhead to write operations. Delta Lake allows you to disable statistics collection by setting the `spark.databricks.delta.stats.autoUpdate.enabled` configuration to `false`.

#### Pattern: Dataset Materializer  
Materialized views are precomputed views that store the results of a query in a table. Materialized views can be used to optimize query performance by reducing the amount of data that needs to be scanned.

Considerations:  
1. Incremental refresh - Delta Lake provides a `MERGE INTO` command that can be used to incrementally refresh materialized views by only updating the rows that have changed since the last refresh. Of course this requires a change tracking mechanism in place using some timestamp column or change data capture mechanism.
2. Data access permissions - ensuring users have access to view only if they have access to underlying data. Fine-grained access control can be implemented.
3. Storage Overhead - materialized views can add storage overhead. 

#### Pattern: Manifest  
Delta Lake manifest files are used to track the list of data files that belong to a Delta Lake table. Manifest files are used to optimize query performance by allowing the query optimizer to skip reading unnecessary data files.

Considerations:  
1. Manifest files are automatically created when writing data to a Delta Lake table.
2. Overhead - manifest files can add overhead to write operations. Delta Lake allows you to disable manifest file creation by setting the `spark.databricks.delta.manifest.enabled` configuration to `false`.

### Data Representation
#### Pattern: Normalizer  
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

#### Pattern: Denormalizer  
Denormalization is the process of combining normalized tables into a single table to optimize query performance. Denormalization can be used to reduce the number of joins required in queries and improve query performance.
Array columns in Delta Lake can be used to store nested data structures in a single column. Array columns can be used to denormalize data and improve query performance.
Struct columns in Delta Lake can be used to store nested data structures in a single column. Struct columns can be used to denormalize data and improve query performance.

Considerations:  
1. Data consistency - denormalization can lead to data redundancy and inconsistency. Delta Lake provides ACID transactions to ensure data consistency.
2. One Big Table Anti-pattern - denormalization can lead to the creation of a single large table that contains all the data in the system. Try to give the table a name and if it uses a lot of conjunctions, it's a sign that it's too big.


## Data Quality Design Patterns
### Quality Enforcement

### Schema Consistency

### Quality Observation


## Data Observability Design Patterns
Even with AWAP(Audit-Write-Audit-Publish) pattern, it is important to have observability in place to monitor the data pipeline and ensure data quality and consistency. For instance when upstream processes are interrupted and your processes did not run, the worst part is you not knowing about it.  

### Data Detectors
#### Pattern: Flow Interruption Detector
Problem: Detecting when data flow is interrupted and alerting the data engineering team. Since there is no failure, there were no alerts until downstream consumers reported missing data.

Inspect metadata, data or storage layer to detect when data flow is interrupted and alert the data engineering team. Useful for scenarios where data flow interruptions can go unnoticed and cause data quality issues.

Considerations:
1. Deciding on right threshold for alerting - too many alerts can lead to alert fatigue, while too few alerts can lead to missing data.

#### Pattern: Skew Detector

Problem: Unbalanced, skewed data could be indicators of bad or missing data.

Perform window-to-window comparison to detect data skew and alert the data engineering team. Useful for scenarios where data skew can cause data quality issues.
Use standard deviation or other statistical measures to detect data skew.

Considerations:
1. Data skew can be caused by missing data, bad data, or other data quality issues. Investigate the root cause of data skew to prevent future occurrences.
2. There could be false positives due to natural data skew. Use statistical measures to detect significant data skew. Use mechanisms to resume processing from the last successful window.
3. Comparison metrics should be based on successful windows to avoid false positives.

### Time Detectors
#### Pattern: Lag Detector
Problem: Detecting lag in data processing due to possible increase in volumes upstream. 

Detect current processing timestamp to compare to recently available data timestamp and alert the data engineering team.
lag_metric = last_available_unit - current_processing_unit
You can compute lag_metric for all partitions and check the P90, P95, P99, etc. to detect outliers. Note: Average could be misleading to detect outliers.

#### SLA Misses Detector
Problem: Detecting when data processing does not meet SLA requirements.

Batch Job: If the processing time of a job exceeds allowable time, it is considered a miss.
Streaming Job: If the processing time of a batch exceeds allowable time, it is considered a miss.

Considerations:
1. For batch data, SLA misses detector is relatively straightforward to implement.
2. For streaming data, late data can pose challenges in detecting SLA misses. 

### Data Lineage

#### Pattern: Dataset Tracker
Problem: Track down root cause of data quality issues and data flow interruptions you are seeing.

1. Use common data orchestration layer to track lineage. Have all data producers report this external data lineage layer.
2. Construct lineage from database query logs.
3. Use a lineage UI to visualize the data lineage.
4. Tools like OpenLineage can help with tracking data lineage.

Considerations:
1. Vendor lock in - ensure that the data lineage tool is vendor agnostic to avoid vendor lock-in.
2. Custom workflows - ensure that the data lineage tool can handle custom workflows and data sources.

#### Pattern: Fine-grained Tracker

Problem: Dataset tracker solves for dataset dependencies but not for columns or row-level lineage.

Column-level tracking can be used to track lineage at a more granular level. Spark has native support for OpenLineage to track column-level lineage.
Row-level tracking might require additional data decorators to track producers at row level.

Considerations:
1. Custom code and additional maintenance - column-level tracking might require custom code and additional maintenance.





