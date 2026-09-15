---
title: How to Monitor Credit Usage in Snowflake
date: 2023-09-15 00:00:00 +0100
tags: [Snowflake]
---

Understanding how your Snowflake credits are consumed is key to keeping costs under control. Snowflake provides detailed, query-level usage data that can help you monitor warehouse activity, identify expensive workloads, and improve cost efficiency.

## Warehouse Credit Usage
Snowflake tracks compute consumption in the view `SNOWFLAKE.ACCOUNT_USAGE.WAREHOUSE_METERING_HISTORY` that reports hourly credit usage for each virtual warehouse. The metrics include all queries executed within the hour, along with the cost of keeping the warehouse running, even if idle, and related cloud services.

> Usage data in `ACCOUNT_USAGE` views typically has a latency of 45–180 minutes.
{: .prompt-warning }

## Estimating Query-Level Costs
If you want to attribute credit usage to individual queries, you can approximate it by isolating workloads and distributing credits proportionally. The cleanest method is to run the queries of interest on a dedicated warehouse — this avoids cross-query noise. Then, for each query:

```text
Estimated Query Credits =
  (Query Execution Time / Total Warehouse Runtime) × Total Credits Consumed
```

This approach gives a reasonable estimate but may overstate cost if the warehouse sat idle for part of that hour.

## Benefits of Credit Monitoring
- Visibility – Understand where compute spend originates.
- Optimization – Identify long-running or inefficient queries.
- Accountability – Attribute usage to teams, pipelines, or workloads.
- Forecasting – Detect cost spikes early and plan scaling policies accordingly.

## Example Queries

Snowflake exposes multiple system views that help you analyze credit usage and query activity:

* `SNOWFLAKE.ACCOUNT_USAGE.QUERY_HISTORY` – query metrics (~45 min latency)
* `SNOWFLAKE.ACCOUNT_USAGE.TASK_HISTORY` – task and DAG execution metadata
* `SNOWFLAKE.ACCOUNT_USAGE.WAREHOUSE_METERING_HISTORY` – hourly credit usage

### 1. Retrieve the DAG run details

Below queries identify a specific task graph (DAG) run in Snowflake’s `TASK_HISTORY` or `COMPLETE_TASK_GRAPHS` views and extract its `run_id`, along with detailed execution metadata such as start and end times, query states, and errors. This forms the entry point for analyzing all queries executed within a single workflow run.
  
To find the DAG run_id:
- After running the DAG go to: Activity > Task History
- Select graph run
- Little button on the right: Open underlying SQL query in worksheet

    ```sql
    -- Auto-generated timeline query:
    SELECT
    name,
    schema_name,
    database_name,
    CONVERT_TIMEZONE('UTC', QUERY_START_TIME) as query_start_time,
    CONVERT_TIMEZONE('UTC', COMPLETED_TIME) as completed_time,
    CONVERT_TIMEZONE('UTC', SCHEDULED_TIME) as scheduled_time,
    DATEDIFF('millisecond', QUERY_START_TIME, COMPLETED_TIME) as duration,
    state,
    error_code,
    error_message,
    query_id,
    graph_version,
    0 as attempt_number,
    run_id
    FROM
    table(
        <database_name>.information_schema.task_history(
        RESULT_LIMIT => 1000,
        ROOT_TASK_ID => <root_task_id>,
        SCHEDULED_TIME_RANGE_START => TO_TIMESTAMP_LTZ('2023-10-25T11:10:35.156Z', 'AUTO'),
        SCHEDULED_TIME_RANGE_END => TO_TIMESTAMP_LTZ('2023-10-25T13:06:07.164Z', 'AUTO')
        )
    )
    WHERE
    run_id = <run_id>
    ORDER BY
    query_start_time ASC
    LIMIT
    250;
    ```

Alternatively, you can retrieve the latest completed DAG using:

    ```sql
    select * from snowflake.account_usage.complete_task_graphs
    where state = 'SUCCEEDED'
    order by query_start_time desc;
    ```

### 2. Calculate query- and workflow-level metrics
Using the retrieved `run_id`, below queries join `QUERY_HISTORY` with `WAREHOUSE_METERING_HISTORY` to measure execution performance and estimate the credit cost of each query in the workflow. 

These queries compute per-query resource usage — bytes scanned, rows processed, spill volume, and execution time — and proportionally distribute warehouse credits to each query based on runtime share. They can optionally include stored procedures and aggregate workflow totals to assess overall efficiency and cost.

    ```sql
    -- Details of the queries executed by the task graph run
    -- (does not include queries ran inside stored procedures)
    select
        m.start_time as warehouse_metering_start,
        m.end_time as warehouse_metering_end,
        q.query_id,
        q.query_text,
        q.query_type,
        q.warehouse_size,
        q.warehouse_name,
        q.compilation_time,
        q.execution_time,
        q.total_elapsed_time,
        q.bytes_scanned,
        q.bytes_written,
        q.bytes_spilled_to_local_storage,
        q.bytes_spilled_to_remote_storage,
        q.bytes_sent_over_the_network,
        q.rows_produced,
        q.rows_inserted,
        q.rows_updated,
        q.rows_deleted,
        q.partitions_scanned,
        q.partitions_total,
        --q.credits_used_cloud_services as q_credits_used_cloud_services, -- too much detail
        zeroifnull(sum(q.total_elapsed_time / 1000) over (partition by q.warehouse_name, warehouse_metering_start)) as warehouse_execution_time,
        zeroifnull(m.credits_used) as warehouse_credits_used,
        zeroifnull(total_elapsed_time / 1000 / nullif(warehouse_execution_time, 0)) as query_percentage_warehouse,
        query_percentage_warehouse * warehouse_credits_used as query_warehouse_credits_used
        --m.credits_used_cloud_services as m_credits_used_cloud_services, -- too much detail
        --m.credits_used_compute as m_credits_used_compute, -- too much detail
        --m.credits_used as m_credits_used -- equal to sum of m_credits_used_cloud_services and m_credits_used_compute, also equal to warehouse_credits_used
    from snowflake.account_usage.query_history q
    left join snowflake.account_usage.warehouse_metering_history m
    on m.warehouse_name = q.warehouse_name
        and q.start_time between m.start_time and m.end_time
    where q.query_id in (
        select query_id
        from snowflake.account_usage.task_history
        where run_id = <run_id>)
    order by q.start_time desc
    ;
    ```

    ```sql
    -- Details of the queries executed by the task graph run
    -- (including queries ran inside stored procedures)
    select
        m.start_time as warehouse_metering_start,
        m.end_time as warehouse_metering_end,
        q.query_id,
        q.query_text,
        q.query_type,
        q.warehouse_size,
        q.warehouse_name,
        q.compilation_time,
        q.execution_time,
        q.total_elapsed_time,
        q.bytes_scanned,
        q.bytes_written,
        q.bytes_spilled_to_local_storage,
        q.bytes_spilled_to_remote_storage,
        q.bytes_sent_over_the_network,
        q.rows_produced,
        q.rows_inserted,
        q.rows_updated,
        q.rows_deleted,
        q.partitions_scanned,
        q.partitions_total,
        --q.credits_used_cloud_services as q_credits_used_cloud_services, -- too much detail
        zeroifnull(sum(q.total_elapsed_time / 1000) over (partition by q.warehouse_name, warehouse_metering_start)) as warehouse_execution_time,
        zeroifnull(m.credits_used) as warehouse_credits_used,
        zeroifnull(total_elapsed_time / 1000 / nullif(warehouse_execution_time, 0)) as query_percentage_warehouse,
        query_percentage_warehouse * warehouse_credits_used as query_warehouse_credits_used
        --m.credits_used_cloud_services as m_credits_used_cloud_services, -- too much detail
        --m.credits_used_compute as m_credits_used_compute, -- too much detail
        --m.credits_used as m_credits_used -- equal to sum of m_credits_used_cloud_services and m_credits_used_compute, also equal to warehouse_credits_used
    from snowflake.account_usage.query_history q
    left join snowflake.account_usage.warehouse_metering_history m
    on m.warehouse_name = q.warehouse_name
        and q.start_time between m.start_time and m.end_time
    where q.session_id in (
        select session_id
        from snowflake.account_usage.query_history
        where query_id in (
            select query_id
            from snowflake.account_usage.task_history
            where run_id = <run_id>
            )
        )
        and query_type != 'CALL' -- exclude, otherwise this will duplicate some info
    order by q.start_time desc
    ;
    ```

    ```sql
    -- Workflow totals - compute sums
    select
        sum(q.compilation_time) as sum_compilation_time,
        sum(q.execution_time) as sum_execution_time,
        sum(q.total_elapsed_time) as sum_total_elapsed_time,
        sum(q.bytes_scanned) as sum_bytes_scanned,
        sum(q.bytes_written) as sum_bytes_written,
        sum(q.bytes_spilled_to_local_storage) as sum_bytes_spilled_to_local_storage,
        sum(q.bytes_spilled_to_remote_storage) as sum_bytes_spilled_to_remote_storage,
        sum(q.bytes_sent_over_the_network) as sum_bytes_sent_over_the_network,
        sum(q.rows_produced) as sum_rows_produced,
        sum(q.rows_inserted) as sum_rows_inserted,
        sum(q.rows_updated) as sum_rows_updated,
        sum(q.rows_deleted) as sum_rows_deleted,
        sum(q.partitions_scanned) as sum_partitions_scanned,
        sum(q.partitions_total) as sum_partitions_total
    from snowflake.account_usage.query_history q
    where q.session_id in (
        select session_id
        from snowflake.account_usage.query_history
        where query_id in (
            select query_id
            from snowflake.account_usage.task_history
            where run_id = <run_id>
            )
        )
        and query_type != 'CALL' -- exclude, otherwise this will duplicate some info
    ;
    ```

## Summary

* Use `WAREHOUSE_METERING_HISTORY` to monitor hourly credit usage.
* Isolate queries on dedicated warehouses when precise measurement is needed.
* Treat proportional credit estimates as **approximations**, since idle time is also billed.
* Combine `QUERY_HISTORY` and `METERING_HISTORY` for a complete performance-and-cost view.

With these tools, you can move beyond guessing your Snowflake spend.
