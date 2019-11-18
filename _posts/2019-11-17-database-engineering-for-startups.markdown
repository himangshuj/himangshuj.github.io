---
layout: post
title: Database Engineering for B2B2C Startups
image: dashboard.jpg
date:  2019-11-17 12:00:00
description: A summary of how I leveraged Redash to build a dashboard for my Company . The dashboard enabled a single engineer to keep tab of every money movement in the company. It also kept tab of any engineering failures and product performace of the company. As with all things startup. This was a continuous journey that saw me abandon nosql databases use foreign tables to do complex joins and build alarms around system truths. 
---
## Some Truths
 
* All code that has been written or will be written will fail one day in production. 
* Customer is not logical , any feature built needs continuous improvement.
* Customers are forgiving about mistakes if we are able to communicate with them transparently and inform them about what is happening? This is particularly true when dealing with money.
* It is important that we not only maintain control of any mishap in production setup but also give appearance of the same.
* If anything goes wrong in production servers, it is always better if we discover the issue before the customer discovers it. 
* Better dashboards leads to better decisions.
* Analytics and Dashboard usually get lower priority than core features.

## Lesson 1: Stick to one SQL database (Postgres or Mysql)

SQL has been around since since 1970s . Many things have changed since then. Today we will be hardpressed to find fresh code written in Fortran or COBOL. Procedural languages gave way to OOP and there is a recent craze around Functional Programming. Yet, SQL continues to rule the roost. In my interactions with non engineers, I usually downplay the need to learn programming but always maintain that everyone needs to know to use SQL if you want to be in a postion of making decisions. Data is the king.  A few years ago, there was a craze around NOSQL databases . Almost all arguments made around SQL databases are around speed performance. Yes, joins are expensive, rigid data structures straighjacket future product enhancements. It is much more difficult to change a column in a mature product than to add additional columns. A lot of these arguments fail to recognize that most end consumers do not care about the savings of a few micro seconds when clicking a button. Most of the customer time is still spend in transmitting data from web servers to end computers vs web servers and db servers. While it is true that doing joins across tables slow down applications, it is impossible to measure performance of the product without looking at data across multiple tables. It is also impossible to check system correctness by looking at one data set alone. Hence, we are faced with a choice either stick to postgres/mysql and do analytics of production data or built an etl pipeline and do custom analytics on a second database. My rule of thumb is not to do ETL pipeline till there are fifty engineers in the team. Building and maintaining ETL pipeline is a full time job. If there are performance issues, Analytics can be done of a read replica. Analytics usually requires only read access and not write access and Analytics looks at a period data rather than latest data so delayed replicas serve the purpose without loosing out on engineering bandwidth. 

 > *I moved from MYSQL to Postgres when Oracle bought MYSQL. At that time I had an apprehension that Oracle was going to sabotage MYSQL. The rest of the blog is going to be restricted to Postgresql*



## Lesson 2: Postgres Foreign Tables are God Sent.

Currently , there is a majority opinion regarding having multiple databases for applications. I think splitting of databases should be postponed to the point that the engineering team is so big that having a single database is serverly limiting the speed of development. Having said that, sometimes you do end up with multiple databases. Whatever be the benefit of using multiple database, it does bring into picture the problem of data consistency. As with any distributed system, multiple databases data may be out of sync. If data goes out of sync,engineering should intervene to fix the system . This should be done by engineering before customer is aware of the impact. POSTGRES does not allow join queries across multiple databases. However we can now import all tables from different tables into the same database and do join across them. As an added bonus, we could give only read permissions to foreign tables and databases. 


**Some Gotchas on Foreign Tables and How to avoid them**


- Name Conflicts: If two remote databases have same table name, 
then it will create conflict while importing them. To solve this, we can foreign tables for all databases in separate schemas.


- DDL vs DML: Any data change in foreign database is automatically reflected in foreign tables . 
However any DDL changes are not propagated. In many ways foreign tables are like views. 
DDL changes need to be followed by recreating the foreign table. We can solve this by recreating foreign tables using [flyway](https://hub.docker.com/r/flyway/flyway).All we have to do with every ddl
change was to rebuild this docker with updated sql versions and voila our combined database which we can call "warehousedb" will have the latest changes.


- Joins: Joins across multiple remote databases can be tricky as they do not use indexes. Hence it is better to create temporary tables with with clause and then join across them. Withing one database, joins are used but when querying across dbs, joins are not used. 

### Code Sample 1 : Creating Remote Server

 ```sql
CREATE FOREIGN DATA WRAPPER warehouse_fdw;
CREATE SERVER app1_server FOREIGN DATA WRAPPER warehouse_fdw 
OPTIONS (host 'app1_db_host', dbname 'app1_db_name', port '5432');
CREATE USER MAPPING FOR redash_user SERVER 
app1_server OPTIONS (user 'remote_db_user', password 'secret');
 ```

### Code Sample 2: Importing public Schema
 ```sql
 DROP SCHEMA  IF EXISTS app1 CASCADE;
CREATE SCHEMA app1;
IMPORT FOREIGN SCHEMA public EXCEPT (password) FROM SERVER
 app1_server INTO app1;
GRANT SELECT ON ALL TABLES IN SCHEMA app1 TO redash_user;
GRANT USAGE ON SCHEMA app1_server TO redash_user;
 ```

### Code Sample 3: Docker For Recreating DB after DDL changes
```dockerfile
FROM boxfuse/flyway:5.1.4-alpine

COPY warehousedb/sqls /flyway/sql
```

### In warehousedb/sqls 
We need to have normal sql files with versioning


## Lesson 3: Have daily jobs and store them in DB.

When dealing in online credit systems, certains things are basic facts. 

- Fact A: There will be network calls. Network calls will fail sometimes.
- Fact B: Failed network calls needs to be retried.
- Fact C: If after a lot of retries , job still does not succeed, it should be investigated manually.
- Fact D: There will be race conditions which may make consumer balance inconsistent. 
- Fact E: Having daily jobs to ensure that all hanging network calls are retried and balances are made consistent.
- Fact F: Core systems may go out of sync. For B2B2C, data may go out of sync between B2B, a daily job ensures that such problems are flagged and wherever possible corrected automatically.
- Fact G: Not all api calls are equal. Some calls should be allowed only if a call of higher priority is completed. A call to email consumers about daily balance should not happen till the call to make payment has succeeded.
- Fact H: Business logic throttling, consumer should not be allowed to go over balance if he tries to withdraw multiple amounts of money, they should be processed in sequence but consumer A's payment shold not affect consumer B.

These are some of the things I have observed. After a lot of iteration, the architecture that I zeroed in was to have a jobs table in database. It queued jobs for the future. It has a unit called reference_id which was used by individual jobs to perform business logic. In addition, it had columns called run_after, priority, state, attempts_remaining and primary_critical_section and secondary_critical_section. Run_after and priority and state controlled which job will be run. Attempts Remaining was used to control retry . Primary critical section ensured throttling as described in Fact H. Secondary critical section was used to ensure that day end statistics jobs did not run untill all individual API calls were complete. A single DB table controlled this flow and everyday I used to check the status of the table. If some jobs were not completed, affected parties were informed adn the issue fixed. This ensured all our systems are in sane condition.

## Lesson 4: Use Redash to have a single dashboard and create tickets using alerts

Silly as it may sound, it is not enough to have the data, it is important to have the data easily and accessible . Redash allows us to connect to databases and build dashboards and alarms around them. After incorporating the lessons in 1,2 and 3 . I was able to create a single dashboard which had all system truths in one url . Redash also allows us to build alarams and I had built simplistic alarms which send an email to our ticketing system when anything went wrong. Redash automatically refreshed the data every few hours. 

- One-Page Redash Dashboad

![Redash Dashboard](/assets/img/redash1.png "Redash Dashboard")

- Redash Alerts (System Truths)

![Redash Alerts](/assets/img/redash3.png "Redash Alerts")

- Redash Alert Destinations (System Truths Escalations)

![System Truths Escalations](/assets/img/redash4.png "System Truths Escalationss")