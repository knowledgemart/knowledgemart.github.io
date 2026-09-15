---
title: Solve gaps and islands problem in AWS Redshift
date: 2020-03-20 00:00:00 +0100
tags: [SQL, AWS, Redshift]
---

A gaps and islands problem refers to a situation where there is a **sequence of rows** in a table that should appear at some **regular intervals** (daily measures of temperature, weekly summary of sales, etc.) but **some entries are missing**. Although such an issue appears most often in the case of data ordered by a date or a timestamp, in general, it can be any sequence ordered by any other type of column which could be treated as a sequence of consecutive values (integers, alphabet letters and so on). Islands refer to chunks of sorted consecutive records and gaps are simply missing values between the islands. This might sound a little bit enigmatic but in reality, is very intuitive. If we have a dataset of some measures taken in March 2020 ordered by day and there are missing measures for the 10th and 24th of March (for whatever reason), the gaps and islands are as follows:

Island: 1<sup>st</sup> - 9<sup>th</sup> March

Gap: 10<sup>th</sup> March

Island: 11<sup>th</sup> - 23<sup>th</sup> March

Gap: 24<sup>th</sup> March

Island: 25<sup>th</sup> - 31<sup>st</sup> March

Real-life examples of such situations might include the temporary unavailability of systems acting as a source of data, any kinds of errors in the stage of dataset creation, or simply raw data structure being not rich enough to be directly used in an analysis or a dashboard.

The challenge in solving this problem is identifying the islands of data that are separated by the gaps of missing values, and then performing some action on those islands, such as counting, summing, or averaging the values within each island.

# Business problem to solve

Let's imagine there's an application. Everyone can use it as long as they create a user account. Some functionalities are available for free and some are only for paid premium accounts. There is also a 1-week free trial. Depending on the usage of the app we can divide users into 3 categories: `beginner`, `contributor`, and `expert`. We gather all possible logs of users' behaviors and attributes in a database. Every time a new user account is created, the log records a unique ID of this account and all user attributes. However, in the case of a change in any of the attributes, the log sends only the ID and the new attribute. Let's look at sample data and see how the log entries look.

```sql
CREATE TABLE user_sample
(
    log_date        date,
    user_account_id integer,
    event_type     varchar(32),
    account_type    varchar(16),
    user_level      varchar(16)
);

INSERT INTO user_sample VALUES ('2020-02-15', 111, 'user account created', 'free (basic)', 'beginner');
INSERT INTO user_sample VALUES ('2020-02-18', 111, 'user account updated', 'trial', NULL);
INSERT INTO user_sample VALUES ('2020-02-25', 111, 'user account updated', NULL, 'contributor');
INSERT INTO user_sample VALUES ('2020-03-01', 111, 'user account updated', 'premium', NULL);
INSERT INTO user_sample VALUES ('2020-03-02', 111, 'user account updated', NULL, 'expert');
INSERT INTO user_sample VALUES ('2020-03-06', 111, 'user account deleted', NULL, NULL);
INSERT INTO user_sample VALUES ('2020-02-19', 222, 'user account created', 'free (basic)', 'beginner');
INSERT INTO user_sample VALUES ('2020-03-01', 333, 'user account created', 'free (basic)', 'beginner');
INSERT INTO user_sample VALUES ('2020-03-04', 333, 'user account deleted', NULL, NULL);

SELECT *
FROM user_sample
ORDER BY user_account_id, log_date;
```

Output:

| log_date | user\_account\_id | event_type | account_type | user_level |
| :--- | :--- | :--- | :--- | :--- |
| 2020-02-15 | 111 | user account created | free (basic) | beginner |
| 2020-02-18 | 111 | user account updated | trial | NULL |
| 2020-02-25 | 111 | user account updated | NULL | contributor |
| 2020-03-01 | 111 | user account updated | premium | NULL |
| 2020-03-02 | 111 | user account updated | NULL | expert |
| 2020-03-06 | 111 | user account deleted | NULL | NULL |
| 2020-02-19 | 222 | user account created | free (basic) | beginner |
| 2020-03-01 | 333 | user account created | free (basic) | beginner |
| 2020-03-04 | 333 | user account deleted | NULL | NULL |

Note that different events (see column: `event_type`) come with different attributes (e.g. `user account created` events always come with the `user_level` attribute, while `user account deleted` always have null `account_type` and `user_level`). This is how events typically look in production environments. They are put in separate tables that can later be joined. Here all the data was inserted in just one table as a starting point for building a user dimension.

In simple terms, in this example, we want to build a user history that shows all possible attributes (`user_account_id`, `account_type `, `user_level`). In data warehousing, this is called a slowly changing dimension with a daily grain.

There are numerous other types of SCDs and other ways of tackling irregularly changing data. I recommend [the Wikipedia article on SCD](https://en.wikipedia.org/wiki/Slowly_changing_dimension) as a reference.

# Step 1: propagate user attributes using window functions and frames

Let's run the following query to start with:

```sql
SELECT
    log_date
  , user_account_id
  , event_type
  , LAST_VALUE(account_type IGNORE NULLS)
    OVER (PARTITION BY user_account_id ORDER BY log_date ROWS UNBOUNDED PRECEDING) AS account_type
  , LAST_VALUE(user_level IGNORE NULLS)
    OVER (PARTITION BY user_account_id ORDER BY log_date ROWS UNBOUNDED PRECEDING) AS user_level
FROM user_sample
ORDER BY user_account_id, log_date;
```

Output:

| log\_date | user\_account\_id | event\_type | account\_type | user\_level |
| :--- | :--- | :--- | :--- | :--- |
| 2020-02-15 | 111 | user account created | free \(basic\) | beginner |
| 2020-02-18 | 111 | user account updated | trial | beginner |
| 2020-02-25 | 111 | user account updated | trial | contributor |
| 2020-03-01 | 111 | user account updated | premium | contributor |
| 2020-03-02 | 111 | user account updated | premium | expert |
| 2020-03-06 | 111 | user account deleted | premium | expert |
| 2020-02-19 | 222 | user account created | free \(basic\) | beginner |
| 2020-03-01 | 333 | user account created | free \(basic\) | beginner |
| 2020-03-04 | 333 | user account deleted | free \(basic\) | beginner |

The query retrieves data from the `user_sample` table. It selects specific columns from the table (`log_date`, `user_account_id`, `event_type`) and also adds additional columns with calculated values.

The additional columns are generated using the `LAST_VALUE` function in combination with the `OVER` clause. This function is used to retrieve the last non-null value in the specified column, over a defined partition and ordering.

The partition is defined as `PARTITION BY user_account_id` which means that the function will only consider values in the specified column that have the same `user_account_id`. The ordering is defined as `ORDER BY log_date` which means that the values will be considered in the order of `log_date`.

The `ROWS UNBOUNDED PRECEDING` frame clause means that the function will consider all the rows in the partition that come before the current row. The final result will be a table with the specified columns and two additional columns for the `account_type` and `user_level`, which will contain the last non-null value of the `account_type` and `user_level` column over the defined partition and ordering.

The query is also ordering the final result set by `user_account_id` and `log_date`.

I recommend the [AWS Window Function Syntax Documentation](https://docs.aws.amazon.com/redshift/latest/dg/r_Window_function_synopsis.html) and [Window Functions in Redshift Doc](https://sonra.io/2017/07/11/introduction-window-functions-redshift/) if you want to understand frame clauses better.

# Step 2: implement date ranges for every user state

With the above query, I eliminated the nulls. Now, if I query the table for any record, the result consists of all possible attributes of a user at a particular moment in time. However, there is still some information missing from this table. What if I want to check what was the status of all users of the app as of 2020-03-03? There is no row with such a `log_date`.

To easily query the user report by a specific date, we need to add time ranges to the table. 

`log_date` acts as a **start date** and there is one more column added named `valid_to` that acts as an **end date** and is equal to one day prior to the next `log_date`. The most recent row is the current status of an **active user** and typically it gets assigned a surrogate end date equal to `9999-12-31`. The most recent row of a **deleted user** has equal `valid_from` and `valid_to` dates.

Let's generate such data and write the result to a new table named `users_final_report`:

```sql
CREATE TABLE users_final_report AS (
    WITH users_no_gaps AS (
        SELECT
            log_date
          , user_account_id
          , event_type
          , last_value(account_type IGNORE NULLS)
            OVER (PARTITION BY user_account_id ORDER BY log_date ROWS UNBOUNDED PRECEDING) AS account_type
          , last_value(user_level IGNORE NULLS)
            OVER (PARTITION BY user_account_id ORDER BY log_date ROWS UNBOUNDED PRECEDING) AS user_level
        FROM user_sample)
        
    SELECT
        log_date AS valid_from
      , CASE
            WHEN event_type != 'user account deleted' 
            THEN NVL(LEAD(log_date, 1) OVER (PARTITION BY user_account_id ORDER BY log_date) - 1, '9999-12-31')
            ELSE valid_from 
            END AS valid_to 
      , user_account_id
      , event_type
      , account_type
      , user_level
    FROM users_no_gaps
    ORDER BY user_account_id, valid_from)
;
```

Now that the final result is ready, I can go back to the question I asked before. I want to see the state of all users as of 2020-03-03. 

The query is pretty straightforward:

```sql
SELECT *
FROM users_final_report
WHERE '2020-03-03' BETWEEN valid_from AND valid_to;
```

Output:

| valid\_from | valid\_to | user\_account\_id | event\_type | account\_type | user\_level |
| :--- | :--- | :--- | :--- | :--- | :--- |
| 2020-02-19 | 9999-12-31 | 222 | user account created | free \(basic\) | beginner |
| 2020-03-01 | 2020-03-03 | 333 | user account created | free \(basic\) | beginner |
| 2020-03-02 | 2020-03-05 | 111 | user account updated | premium | expert |






