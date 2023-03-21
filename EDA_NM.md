# EDA of 'flights' table

## Addressing size concerns

The size of the 'flights' table to be used for flight prediction has been noted as very large. For dealing with this data it is important to understand the magnitude of this concern and how it may affect the machine being used to query it.

### Determining the size of 'flights'

```sql
SELECT pg_size_pretty(pg_total_relation_size('flights')) as size;
```

The flights table is 4267 MB (4.267GB). 

The computer querying this table, to query the entire table, will need at least 4.267 GB of accessible memory (typically 'RAM', or 'unified memory' in the case of Apple m1 architecture ) as well as overhead for query processing.

My system is a MacBook Air (m1, 2020) with 8GB of unified memory. A check of the Activity Monitor shows a few important things:
* the system requires ~1.25 GB of 'Wired Memory' for the system to operate
* during my typical usage, I use ~7GB of the memory

Theoretically, if I were to clear cached memory, close all other apps, quit all other processes, etc.. I should be able to clear most of the 6.75GB that isn't Wired Memory. Practically, I would like to keep my other running apps and projects open. For a wide safety margin, I will be aiming to stay within 1GB required memory per query, which equates to referencing no more than 1/5 of the rows at time.

The size of the additional tables are:
* passengers = 830MB
* fuel_comsumption = 0.752 MB
* flights_test = 124MB

To query the table 'flights' for the number of rows would require a count and that would be referencing the entire table. To get the number of rows without querying 'flights' the estimate of rows stored as metadata in pg_class can be queried instead.

### Getting an estimate of rows in 'flights'

```sql
SELECT reltuples::bigint AS row_count
FROM pg_class
WHERE relname = 'flights';
```
The estimate of rows is 15_207_047.

By that estimate, the safe amount of rows to query for my system is ~3_000_000 or less. The safe amount of columns to query is ~8 or less (depending on data types). 