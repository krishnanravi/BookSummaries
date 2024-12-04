# [Data Engineering Design Patterns](https://www.oreilly.com/library/view/data-engineering-design/9781098165826/)

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

