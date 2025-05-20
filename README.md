# Database Modelling with Apache Cassandra - Sparkify

## Introduction
A startup called Sparkify wants to analyze the data they've been collecting on songs and user activity on their new music streaming app. The analysis team is particularly interested in understanding what songs users are listening to. Currently, there is no easy way to query the data to generate the results, since the data reside in a directory of CSV files on user activity on the app.

They'd like a data engineer to create an Apache Cassandra database which can create queries on song play data to answer the questions, and wish to bring you on the project. Our role is to create a database for this analysis. 

This project is part of the Udacity Data Engineering nanodegree.

## Data Model Requirements & Methodology
For this project, we will be working with one dataset: **event_data_new.csv**. We will process this file to create a denormalized dataset based on the queries. 

  - Design tables to answer the queries outlined in the project template
  - Write Apache Cassandra `CREATE KEYSPACE` and `SET KEYSPACE` statements
  - Develop the `CREATE` statement for each of the tables to address each question
  - Load the data with `INSERT` statement for each of the tables
  - Include `IF NOT EXISTS` clauses in your `CREATE` statements to create tables only if the tables do not already exist. It is recommended we also include `DROP TABLE` statement for each table, this way we can run drop and create tables whenever we want to reset the database and test our ETL pipeline
  - Test by running the proper select statements with the correct `WHERE` clause
  - Don't use `ALLOW FILTERING` to test and run queries.

### Query 1: `SELECT artist, song_title, song_length FROM table1 WHERE session_id = 338 AND item_in_session = 4;`

  - We require the columns `artist`, `song_title`, `song_length` based on specific `session_id` and `item_in_session`. Based on this, we need to partition the table in a way that it looks for rows only for `session_id = 338` and `item_in_session = 4`. This allows us to filter-through unwanted rows and make the read of the table efficient.
  - We will set partition keys to both `session_id` and `item_in_session` as both of them will be used in the `WHERE` statement to get the desired rows

### Query 2: `SELECT artist, song, user FROM table2 WHERE user_id = 10 and session_id = 182;`

  - We require the columns `artist`, `song_title`, `user` based on specific `user_id` and `session_id`. Based on this, we need to partition the table in a way that it looks for rows only for `user_id = 10` and `session_id = 182`. This allows us to filter-through unwanted rows and make the read of the table efficient. We also need to sort the rows by `item_in_session` (ascending as it is not explicitly mentioned to sort descending), to do this we need to add `item_in_session` in the composite partition key.
  - We will set partition keys to both `user_id` and `session_id` as both of them will be used in the `WHERE` statement to get the desired rows. Along with the partition key, we also need a cluster key to sort the rows based on `item_in_session`, which will be the column itself.
  - One additional thing to take care of in this query is the column `user`. This will be a calculated column as we required both first and last name of a user together in this column.

### Query 3: `SELECT user FROM table3 WHERE song = 'All Hands Against His Own';`

  - This is a relatively simpler query. We only need to return a single column `user`, again, this will be a calculated column similar to what we did in Query 2. We will also add the column `song` in the table because we need to filter for the song **All Hands Against His Own**. `song` and `user` both will be partition keys.
  - One thing to keep in mind is the order of the column we add to the partition key - if we add `user` followed by `song`, the query will look at every row in `user` and then partition using the `song` column. This is not efficient for a Cassandra database because it is not optimised for reads, and won't be able to process large amounts of data efficiently. In fact, the query won't work unless we add `ALLOW FILTERING` to the query, which is never advised unless you are absolutely certain of your query (and the data is not too big).

## Files
  - `event_data_new.csv`: raw data to be denormalized 
  - `data_model.ipynb`: this is the full data model which processes the data into denormalized tables

## Acknowledgements
A big thanks to Udacity for providing the dataset and project template. 

