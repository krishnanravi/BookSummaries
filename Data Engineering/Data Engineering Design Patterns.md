# [Data Engineering Design Patterns](https://www.oreilly.com/library/view/data-engineering-design/9781098165826/)

## Data Ingestion Design Patterns

### Full load
Pattern: Full loader
Extract Load(EL) jobs a.k.a pass through jobs are suitable for 
1. slowly evolving entities and
2. low volumes and
3. sources that don't track updated timestamps to allow for incremental ingestion pattern

##### Considerations
1. Data Volume - scaling infrastructure for steadily increasing data volume should be accounted for.
2. Data Consistenty - consider views for downstream and blue-green approach for full publication. Rollback support to be planned for invalid publications. Delta Lake is a great way to ensure atomicity and consistency of data for consumers. 


### Incremental load
Pattern: Incremental loader
1. Use a delta column in source to identify records for incremental load from last successful ingestion.
2. Time-partitioned datasets allow identifying incremental data based on new partitions since last successful ingestion.

##### Considerations
1. Data mutability
2. Recogniziing deletes in source. Soft deletes may be a solution to manage deleted as updates.
3. Limiting backfill jobs for backfill/reconcoliation to fewer partitions to allow for horizontal scaling of ingestion jobs.


Pattern: Change Data Capture

### Replication
Pattern: Passthrough replicator
Pattern: Transformation replicator

### Data compaction
Pattern: Compactor

### Data readiness
Pattern: Readiness marker
### Event-driven
Pattern: External trigger

