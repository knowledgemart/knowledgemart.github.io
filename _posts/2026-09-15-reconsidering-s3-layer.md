---
title: "Reconsidering the S3 Landing Layer in API-to-Snowflake Batch Pipelines"
date: 2026-09-15 00:00:00 +0100
tags: [Snowflake, AWS, S3, DataEngineering]
---

A conventional batch-ingestion architecture for external APIs typically places object storage between the extraction process and the analytical warehouse. The resulting flow commonly looks like this:

```text
External API
    |
    v
Airflow / Dagster / Prefect
    |
    v
Amazon S3 / Google Cloud Storage (GCS) / Azure Blob Storage
    |
    v
Snowflake
```

The pattern is common enough that the object-storage landing layer is frequently treated as a default rather than as an explicit architectural decision. And there are good reasons for that convention. Object storage provides inexpensive durable persistence, integrates naturally with Snowflake bulk loading, separates source extraction from warehouse ingestion, and provides a convenient place to retain raw source artifacts. At the same time, modern Snowflake deployments provide several alternative ingestion mechanisms: direct writes into tables, Snowpipe Streaming, and Snowflake-managed internal stages.

The relevant design question is therefore which guarantees and operational properties the additional persistence layer provides, and whether those properties justify another system boundary in a particular pipeline.

## The Standard Landing-Zone Pattern

The S3-based implementation separates the pipeline into two independently persisted operations:

```text
Extraction:
External API -> S3

Load:
S3 -> Snowflake
```

Once the extraction has produced a complete object or set of objects in S3, the source interaction is finished. Snowflake ingestion can subsequently succeed, fail, or be retried without another request to the external API.

This creates an explicit persistence boundary between source acquisition and warehouse ingestion.

A typical implementation would not necessarily retain the source response as one large object. An extractor can parse the response incrementally and generate a set of appropriately sized files:

```text
s3://raw/vendor/transactions/
    extraction_date=2026-09-14/
        run_id=01JXYZ.../
            part-00000.ndjson.gz
            part-00001.ndjson.gz
            part-00002.ndjson.gz
            ...
            manifest.json
```

The manifest can contain control metadata such as:

```json
{
  "run_id": "01JXYZ...",
  "source": "vendor-transactions-api",
  "requested_at": "2026-09-14T01:00:00Z",
  "completed_at": "2026-09-14T01:18:42Z",
  "record_count": 8123421,
  "file_count": 94,
  "uncompressed_bytes": 48721381231,
  "content_type": "application/json",
  "schema_version": "2026-09",
  "status": "COMPLETE"
}
```

The manifest, or an equivalent completion marker, is important because the existence of several output objects does not imply that the logical batch was captured completely. If the extraction process crashes after writing 61 out of 94 expected files, downstream consumers need a mechanism that distinguishes an incomplete run from a valid batch.

The loader can then process only batches for which extraction has reached a committed state. The load step itself can be run by the orchestrator as a scheduled `COPY INTO`, or delegated to Snowpipe, which loads new objects automatically in response to S3 event notifications. Snowpipe ingests objects as they arrive rather than per batch, so with it the completeness check moves downstream: files land in a raw table continuously, and the batch is promoted only once its manifest is present.

## What the Landing Layer Changes

The primary effect of the landing layer is on the failure graph.

Consider a direct pipeline:

```text
API -> extractor -> Snowflake
```

Assume the source contains ten million records and the extractor has already processed seven million when the Snowflake write path becomes unavailable. If records have been committed incrementally, Snowflake now contains a partial representation of the source response. Recovery depends on the ingestion protocol and the idempotency strategy. The system must determine whether the partially committed batch should be deleted, resumed, replayed, or reconciled.

For a source that supports cursors or offsets, this may be straightforward. For a problematic API, with no pagination, no cursor or continuation token, it is not – a retry may require executing the complete request again.

By contrast, when the complete extraction has already been persisted to S3, a later Snowflake failure does not propagate back to the source:

```text
API -> S3        complete

S3 -> Snowflake  failed
S3 -> Snowflake  retry
```

The architectural difference is not that S3 is assumed to be more available than Snowflake. Both are distributed services and both can fail. The difference is that successful persistence in S3 terminates the source-side operation and converts the extracted dataset into a replayable input for downstream processing.

In other words, object storage changes the scope of a retry.

## Failure Semantics

The distinction can be expressed more precisely by considering several failure points.

| Failure point                                                 | Direct API -> Snowflake                                   | API -> S3 -> Snowflake                           |
| ------------------------------------------------------------- | --------------------------------------------------------- | ------------------------------------------------ |
| API disconnects before complete response                      | Source request must usually be repeated                   | Source request must usually be repeated          |
| Extractor process crashes before complete source capture      | Extraction must be retried                                | Extraction must be retried                       |
| Snowflake unavailable before any records are committed        | Extraction may need to be restarted                       | No impact if source payload is already persisted |
| Snowflake fails after partial ingestion                       | Requires partial-load recovery and possibly source replay | Reload from persisted batch                      |
| Warehouse transformation logic is later found to be incorrect | Replay may require source access                          | Replay can begin from landed batch               |
| Source data changes between requests                          | Re-extraction may produce a different snapshot            | Original captured snapshot is retained           |
| Source API is unavailable during downstream recovery          | Recovery may be blocked                                   | Recovery remains possible from object storage    |

The landing layer does not improve the first two cases. If the source connection itself fails before the logical extraction has completed and the API does not support resumability, S3 cannot reconstruct data that was never received. Its benefit begins after the source payload has been captured successfully.

## Direct-to-Snowflake Is a Valid Alternative

A direct path should not be treated as an anti-pattern. It can be entirely appropriate when its recovery semantics are well defined.

It is worth being precise about what "direct" means in Snowflake. Most connector-level bulk write paths – `write_pandas` in the Python connector, Snowpark DataFrame writes, large batched bindings in JDBC – upload files to an internal stage and run `COPY INTO` behind the scenes. The only ingestion path that writes rows into a table without an intermediate file is Snowpipe Streaming. A pipeline built on a standard connector is therefore usually an internal-stage pipeline with the staging step hidden inside the driver, which matters for the comparison later in this article.

One implementation is to assign a unique batch identifier to each extraction and attach it to every record written into a raw table:

```text
RAW_TRANSACTIONS
----------------
batch_id
source_record_id
ingested_at
payload
```

If a batch fails midway, the pipeline can remove the incomplete batch and replay it:

```sql
DELETE FROM raw_transactions
WHERE batch_id = '<failed_batch_id>';
```

Alternatively, it can merge by a stable source key or maintain an ingestion ledger that records which logical units were committed successfully. Snowpipe Streaming provides a native form of that ledger: each channel carries an offset token that the client attaches to every row batch, and the token of the last committed batch can be read back from Snowflake after a failure. The extractor can then resume from that offset instead of deleting and replaying the batch – provided the source can be re-read from the same position, which brings the discussion back to source semantics.

The feasibility of this approach depends heavily on those semantics. If source requests are cheap, deterministic, and reproducible, replaying the API may be acceptable. If a request is expensive, rate-limited, non-repeatable, or returns a changing snapshot, the cost of coupling warehouse recovery to source replay becomes much higher.

Direct ingestion therefore does not eliminate the need for durable-state design. It moves that responsibility into the warehouse ingestion protocol.

## Raw Tables and Raw Artifacts Are Different Abstractions

One argument against a separate landing layer is that the raw source representation can simply be stored in Snowflake. That is often true, but two different notions of "raw" are involved.

A Snowflake raw table usually preserves the source data before business transformations. For example:

```sql
CREATE TABLE raw_api_transactions (
    batch_id STRING,
    ingested_at TIMESTAMP_TZ,
    payload VARIANT
);
```

This is a raw logical representation of the source record.

An object-store landing layer can instead preserve the source artifact itself: the original response body or a deterministic segmentation of that response into files.

The distinction matters when parsing or normalization changes the physical representation. Converting JSON into `VARIANT`, for example, preserves its logical structure but not necessarily its original byte-level serialization, compression, encoding, whitespace, or file boundaries.

In many analytical systems this distinction has no value. In others, exact source capture is useful for reconciliation, forensic debugging, auditability, or reproducing parser behavior against historical inputs.

The requirement should therefore be stated explicitly: does the platform need the raw logical data, or does it need the source artifact?

## Internal Snowflake Stages Complicate the Comparison

The comparison is also not strictly between S3 and direct row insertion.

Snowflake supports internal stages, which allow files to be uploaded to Snowflake-managed storage before they are loaded into tables:

```text
External API
    |
    v
Extractor
    |
    v
Snowflake internal stage
    |
    v
COPY INTO
    |
    v
Raw table
```

This preserves several properties of the external-landing design. File creation and table loading remain separate operations, `COPY INTO` retains its normal bulk-loading semantics, and failed table loads do not necessarily require another API call if the staged files are still available.

The more accurate comparison is therefore between three designs:

```text
1. API -> direct Snowflake ingestion

2. API -> Snowflake internal stage -> Snowflake table

3. API -> external object storage -> Snowflake table
```

The distinction between the second and third options is mostly about platform boundaries and ownership of persisted raw data. With an internal stage, the landing layer is still part of the Snowflake platform. With S3, the raw copy exists independently of Snowflake.

## Why Warehouse Independence May Matter

An external landing zone gives the raw dataset a lifecycle that is independent of the warehouse. The same objects can be consumed by Spark, another warehouse, a data-quality system, an ML pipeline, or a migration process without exporting them back out of Snowflake first. This is relevant in architectures where object storage is the authoritative raw-data layer and Snowflake is one compute and serving system among several. It is less relevant in organizations where Snowflake is deliberately treated as the central persistence and processing platform.

Cost follows the same split. Object storage is billed at object-storage rates and supports lifecycle policies, so raw artifacts can be moved to colder tiers or expired on a schedule. Internal stage storage is billed through Snowflake and has no equivalent tiering. Long retention of raw data is therefore usually cheaper outside the warehouse, while transfer between S3 and Snowflake within the same region carries no egress charge.

Governance points the other way. A raw copy in S3 is a second security perimeter: IAM policies, a storage integration, bucket encryption, and a separate scope for audit and compliance. Data in an internal stage stays inside Snowflake's role-based access control and masking policies. In regulated environments this is often the deciding argument for the second design.

The difference between the two file-based options is therefore one of architectural ownership rather than ingestion capability. If the organization does not require warehouse-independent raw storage, internal stages provide nearly all of the practical benefits of file-based loading while reducing the number of systems – and security boundaries – involved.

## Backpressure and Lifetime Coupling

Direct streaming also creates a throughput dependency between the source connection and the destination. Consider an extractor reading an API response while concurrently forwarding parsed batches to Snowflake:

```text
API -> parser -> Snowflake
```

If the Snowflake write path slows down, the extractor must either buffer more data or reduce the rate at which it consumes the HTTP response. Application-level flow control handles the mechanics of slowing down, but API servers are rarely tolerant of it: a response consumed slowly runs into idle timeouts, gateway limits, or a maximum request duration, and the server closes the connection. For a source without resumability, a warehouse slowdown then becomes a failed extraction that must be repeated from the beginning.

In practice an extractor avoids this by draining the response at the rate the API allows and buffering it locally, on disk or in memory, ahead of the warehouse write. At that point the pipeline already contains a landing layer; it is simply an ephemeral one, on a single host, that disappears with the process.

With an external landing layer:

```text
API -> S3

S3 -> Snowflake
```

that buffer becomes explicit and durable, and the two throughput domains are separated. Source extraction is limited primarily by the API, network, parsing, compression, and object-store write rate. Warehouse ingestion runs independently afterwards, and a slow or unavailable warehouse no longer holds a source connection open.

For a batch workload, where sub-second or sub-minute latency is not required, this separation often simplifies failure recovery and capacity management.

## File Layout Is Part of the Ingestion Design

When an external landing layer is used, the extraction process also controls the physical layout of the dataset that Snowflake will subsequently load. This is relevant because file boundaries influence the amount of parallelism available during bulk ingestion.

A single API response does not need to map to a single object in the landing layer. The extractor can rotate output files as it processes the response, using a target file size or record count as the criterion:

```python
for record in response_stream:
    writer.write(record)

    if writer.size >= target_size:
        finalize(writer)
        writer = create_next_writer()
```

The `part-*.ndjson.gz` layout shown earlier is the result. It allows Snowflake to load multiple files concurrently rather than treating the entire extraction as a single unit of work. The layout should provide enough independent files to exploit the available warehouse parallelism without producing an excessive number of small files, which increases per-file scheduling and metadata overhead.

File boundaries also provide useful operational granularity. Individual objects can be validated independently, associated with checksums and record counts, and tracked by the manifest as discrete units of ingestion. The physical organization of the landed data does not have to reflect the shape of the source API response: even if the source exposes the dataset as one large response, the extraction layer can produce a layout optimized for downstream bulk loading.

## Idempotency Is Still Required After Introducing S3

Object storage does not eliminate duplicate-load problems.

Assume the Snowflake `COPY INTO` succeeds, but the orchestration task crashes before recording its successful completion. The scheduler may subsequently retry the task. The load path therefore still needs idempotent semantics.

The first line of defence is built into `COPY INTO` itself. Snowflake keeps load metadata for every file loaded into a table, and for 64 days a subsequent `COPY INTO` that encounters an already-loaded file skips it unless `FORCE = TRUE` is set. A retried load over the same set of objects is therefore a no-op for the files that were already committed, which covers the most common orchestration failure without any additional bookkeeping. The guarantee applies equally to internal and external stages, which is one reason the file-based designs are easier to make idempotent than row-level writes. It is also bounded: it is scoped to one target table, expires after the retention window, and knows nothing about logical batches, so a replay after 64 days or into a different table is not deduplicated by it.

Beyond load history, idempotency can be achieved through deterministic file names, batch identifiers, staging tables, transactional promotion, or explicit merge logic. A common pattern is:

```text
S3 batch
   |
   v
Snowflake transient staging table
   |
   v
validation
   |
   v
MERGE / INSERT into raw table
   |
   v
mark batch as loaded
```

The landing layer does not replace idempotency. It narrows the part of the pipeline for which idempotency must be implemented – and, for file-based loads, Snowflake already implements most of it.

## When the S3 Layer Is Actually Useful

An external landing zone provides meaningful value when at least some of the following properties are required:

- downstream failures should not force another source extraction;
- source requests are expensive, rate-limited, or non-repeatable;
- the original source artifact must be retained;
- historical loads need to be replayable independently of the API;
- raw data should remain available independently of Snowflake;
- multiple consumers need access to the same landed data;
- source and warehouse throughput should be decoupled;
- retention policy for raw data differs from retention policy for warehouse tables.

If none of these requirements applies, the S3 layer may be unnecessary. A direct ingestion path or a Snowflake internal stage can reduce system complexity while still providing adequate recovery semantics.

## Conclusion

The S3 landing pattern is common because it provides a useful set of properties for batch ingestion: a durable boundary after source extraction, independent replay of downstream processing, separation between source and warehouse throughput, and a warehouse-independent copy of raw data. Those properties are not universally required, and Snowflake can reproduce most of them through internal stages or a carefully designed direct-ingestion workflow.

For a poorly resumable external API, the most significant benefit of S3 is that successful source acquisition becomes a completed and durable operation before Snowflake ingestion begins. For sources with strong replay semantics, or in Snowflake-centric platforms where independent raw storage is unnecessary, the additional layer may provide little benefit relative to its cost. The architecture should be selected on the required persistence and recovery semantics, not on the assumption that every API-to-Snowflake pipeline needs an external object store.
