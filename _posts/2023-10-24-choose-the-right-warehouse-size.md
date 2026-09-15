---
title: Choosing the Right Snowflake Warehouse Size
date: 2023-10-24 00:00:00 +0100
tags: [Snowflake]
---

Selecting the right Snowflake virtual warehouse is all about finding the right balance between query performance and cost efficiency. This guide explains how warehouse sizing works, when to scale, and which performance metrics help you decide — so you can get the most from your Snowflake environment without overspending.

## What Is a Snowflake Warehouse?

A Snowflake warehouse is a cluster of compute resources used to run queries and tasks. It operates independently of storage and scales on demand. Each size (X-Small, Small, Medium, and so on) doubles the compute power and cost of the previous one.

| Warehouse size | Number of nodes | Cost (credits/hour) | Cost (credits/second) |
|----------------|-----------------|------|---------|
| XS             | 1               | 1    | 0.0003  |
| S              | 2               | 2    | 0.0006  | 
| M              | 4               | 4    | 0.0011  |
| L              | 8               | 8    | 0.0022  |
| XL             | 16              | 16   | 0.0044  |
| 2XL            | 32              | 32   | 0.0089  |
| 3XL            | 64              | 64   | 0.0178  |
| 4XL            | 128             | 128  | 0.0356  |
| 5XL            | 256             | 256  | 0.0711  |
| 6XL            | 512             | 512  | 0.1422  |

Both compute capacity and cost double with each warehouse size.

> All warehouses, regardless of size, are charged based on the amount of time they are running, whether actively processing queries or running idle.
{: .prompt-warning }

> Snowflake charges **a minimum of 60 seconds per query** on a running or resized warehouse. Even if a query runs for only a few seconds, the user will be charged for a full minute of usage.
{: .prompt-warning }

## How to Right-Size a Snowflake Warehouse

### Start Small and Scale Up as Needed
Always begin with an X-Small warehouse and increase the size only when metrics justify it.

Scale up if you notice:
- Frequent query queuing
- Long execution times
- Memory spill to disk (local or remote)

Scaling up makes sense only if the larger warehouse provides at least 1.5x to 1.8x faster performance, since cost doubles with each step. Choose the size that offers the best cost-to-performance ratio rather than simply the fastest execution time.

### Automate Suspension and Resumption

Snowflake warehouses automatically suspend after a set period of inactivity (default: 600 seconds). Lowering this to 60 seconds can significantly reduce costs, but be mindful of the trade-off: suspending clears the local cache. If your workloads include repeated queries on the same data, a longer auto-suspend period may help preserve performance by keeping the cache warm. Finding the right balance is key.

### Watch for Under- or Over-Provisioning

Under-provisioned warehouses struggle to handle workloads efficiently — queries queue, performance drops, and execution time increases. To identify under-provisioning, monitor performance indicators such as: **query execution time**, **queue time**, and **the number of queued queries**. If these metrics consistently show poor performance, increasing the warehouse size to allocate more resources may be necessary.

Over-provisioned warehouses waste resources and money without significant performance gains. To identify over-provisioning, analyze the **CPU** and **memory usage**. If these metrics consistently show low utilization, it may be more cost-effective to reduce the warehouse size.

Scaling decisions should always be driven by data, not guesswork.

### Monitor Disk Spillage
In Snowflake, when a warehouse cannot process operations fully in memory, it spills data to disk — first to local, then remote storage. This “spillage” degrades performance and appears in the query profile as “Bytes spilled to local/remote storage.”

To minimize the impact of spilling:

- Increase the warehouse size to provide more memory and local disk space.
- Review and optimize the query for better efficiency.
- Reduce the data processed (e.g., prune partitions, select fewer columns). 
- Limit the number of parallel queries on the same warehouse.

### Find the Sweet Spot: Performance vs. Cost
Start small and scale up gradually. When you reach a point where doubling the warehouse size no longer halves query execution time, you’ve found your performance-cost sweet spot. This is where additional compute delivers diminishing returns.

### Review Query History
Use Snowflake’s Query History to spot signs of resource pressure.

Look out for error messages such as **"Warehouse full"** or **"Insufficient credit"**, which can indicate that the warehouse is unable to accommodate the query workload. Such alerts help pinpoint performance or budgeting issues early.

## Summary
To recap:

1. Start with the smallest warehouse
2. Scale up only when metrics demand it - consistent queuing, long execution times, or memory spill.
3. A larger warehouse should deliver at least ~1.6× performance to justify its higher cost.
4. Fine-tune auto-suspend settings to balance cache efficiency and idle time.
5. Track queue times, execution duration, and disk spilling.
6. Keep an eye on utilization and query history to avoid over-provisioning.

Right-sizing a Snowflake warehouse is an iterative process that requires testing, monitoring, and gradual adjustments.
