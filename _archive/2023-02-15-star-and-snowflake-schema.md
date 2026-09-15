---
title:  "Star vs Snowflake Schema – Choosing the Right Model"
date:   2023-02-15 00:00:00 +0000
tags: [DataModeling]
---

Data warehousing is an essential part of modern businesses, where organizations store and analyze vast amounts of data. In this context, the way data is structured and organized is critical to ensure data integrity and effective analysis. Star and snowflake schema are two of the most common data structures used in data warehousing.

## Star Schema

The star schema is a central component of Ralph Kimball's data warehousing methodology. It is a simple and intuitive data structure where the central fact table is surrounded by dimension tables. The fact table contains the data to be analyzed and the dimension tables provide additional contexts, such as time, location, product, or customer information. The central fact table is connected to the dimension tables through keys, and the relationships between the fact table and dimension tables are represented by lines connecting the keys.

### Fact table

A fact table is a central table in a star schema. Fact tables store events (pieces of information about a business process) like sales transactions, inventory movement, marketing campaigns, etc. They typically contain foreign keys to dimensional data where descriptive information is kept. The number of dimension columns determines the *dimensionality* of a fact table, while the dimension values (the level of detail or precision of the information) determine the *granularity* of a fact table. Higher dimensionality means greater complexity of the schema and possible slower performance of the database. A higher level of granularity provides more detailed information but can result in a larger fact table and slower query performance, while a lower level of granularity provides less detailed information, but can result in a smaller fact table and faster query performance.

### Dimension tables

A dimension is a data structure that categorizes data in a specific way, such as by time, geography, product, customer, or any other relevant attribute. Dimensions are used to describe the context of the facts or events stored in a fact table and provide the basis for analyzing and aggregating the data.

A dimension typically consists of a set of attributes or characteristics that describe an aspect of the business, such as the date of a sale, the location of a store, or the name of a product. These attributes are organized into a hierarchical structure, such as a time dimension with year, quarter, month, and day attributes, or a product dimension with a product category, product line, and product attributes.

Dimensions provide the structure for grouping and filtering the data, and allow the user to analyze the data from multiple perspectives. The use of dimensions in dimensional modeling helps to simplify the data and improve the performance of queries and reports.


The advantages of the star schema include:

**Simplicity**: The star schema is easy to understand and navigate, even for users who are not familiar with the data.

**Performance**: The star schema is optimized for querying, providing fast results even with large amounts of data.

**Scalability**: The star schema is easy to expand, as new dimension tables can be added as needed.

## Snowflake Schema

While the snowflake schema is not a concept from Kimball's theory, it is a more complex version of the star schema, where the dimension tables are normalized to reduce data redundancy and is often used in conjunction with the star schema to improve the data integrity of data warehousing systems. In a snowflake schema, the dimension tables are connected to each other through relationships, and the fact table is connected to the dimension tables through keys. This normalization helps to reduce data redundancy and saves storage space, but it can also lead to slower query performance.

The advantages of the snowflake schema include:

**Data integrity**: The normalization of the dimension tables ensures data integrity, as data is stored in a consistent and standardized manner.

**Space efficiency**: The normalization of the dimension tables reduces data redundancy, leading to more efficient use of storage space.

**Flexibility**: The snowflake schema is flexible and can be easily adapted to changing business requirements.

## Conclusion

The star schema is simple and optimized for querying, while the snowflake schema is more complex but provides better data integrity and space efficiency. The choice between star and snowflake schema will depend on the organization's specific needs, and it is important to carefully consider the trade-offs between simplicity, performance, scalability, data integrity, and space efficiency when choosing a data structure for a data warehousing project. The same applies to dimensionality and granularity choice.

As per the introduction to the first chapter of [Kimball's Data Warehouse Toolkit](https://www.amazon.pl/Data-Warehouse-Toolkit-Definitive-Dimensional/dp/1118530802): *(...) first and foremost, the DW/BI system must consider the needs of the business. With the business needs firmly in hand, we work backwards through the logical and then physical designs, along with decisions about technology and tools.*
