---
title: Redshift Sort Keys Explained
date: 2020-09-29 00:00:00 +0100
tags: [AWS, Redshift]
---

## Amazon Redshift – When (and Why) to Use Sort Keys
In Amazon Redshift, sort keys determine the physical order in which table rows are stored on disk. Redshift stores columnar data in 1 MB blocks, along with metadata that describes each block — including its minimum and maximum values.

When a query filters data by a range, the optimizer uses these min/max values to skip over entire blocks that don’t match the filter. For example, if a table contains five years of data sorted by `date` and you query just one month, Redshift can skip scanning most of the data, greatly reducing I/O and query time.

Sort keys are defined when a table is created (as part of the DDL). During initial data load, rows are written in sorted order. This metadata guides the query planner and can significantly improve performance — especially for analytical queries that use filters, joins, and aggregations.

## Compound Sort Keys
A compound sort key consists of one or more columns defined in a specific order. Redshift sorts data by the first column, then by the next, and so on. Compound sort keys work best when your queries consistently filter or join on the same set of columns — especially when predicates reference the leading column. They also improve the performance of `JOIN`, `GROUP BY`, and `ORDER BY` operations, as well as window functions that use `PARTITION BY` or `ORDER BY`.

According to AWS best practices:
- If recent data is queried most frequently, make the timestamp column the leading column in the sort key.
- If you frequently filter or group by one column, use that column as the sort key.
- For dimension tables used in joins, use the join column as the sort key.

Compound sort keys are generally preferred for most workloads, especially those with frequent updates or incremental loads, as they are easier to maintain than interleaved keys.

## Interleaved Sort Keys
An interleaved sort key gives equal weight to each column in the key, unlike compound keys where order determines priority. This design can improve performance for workloads that filter on multiple different columns — for example, queries that sometimes filter by `customer_id`, sometimes by `region`, and other times by `date`.

However, interleaved sort keys require more maintenance. Frequent updates or uneven data distribution can cause key skew, reducing their effectiveness. Interleaved keys also take longer to vacuum and analyze. You can define up to eight columns in an interleaved sort key.

## Choosing the Right Key

| Scenario                                               | Recommended Sort Key Type                     |
| ------------------------------------------------------ | --------------------------------------------- |
| Queries always filter by the same column or time range | **Compound** (leading column = filter column) |
| Queries filter by several different columns            | **Interleaved**                               |
| Tables updated frequently or incrementally loaded      | **Compound**                                  |
| Tables used mainly for analytical reads                | **Either**, depending on filter patterns      |

## Additional Resources

* [AWS Documentation: Sorting Data](https://docs.aws.amazon.com/redshift/latest/dg/t_Sorting_data.html)
* [AWS Documentation: Interleaved Sort Keys](https://docs.aws.amazon.com/redshift/latest/dg/t_Sorting_data-interleaved.html)
* [AWS Blog: Redshift Engineering’s Advanced Table Design Playbook](https://aws.amazon.com/blogs/big-data/amazon-redshift-engineerings-advanced-table-design-playbook-compound-and-interleaved-sort-keys/)
