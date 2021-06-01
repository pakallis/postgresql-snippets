# PostgreSQL snippets

A collection of not so obvious snippets to perform various tasks with postgreSQL. Measure performance bottlenecks, cache hit rate etc.


## Cache Hit Rate

Find the database cache hit rate. 
Usually this number should be at least 99%. If it is significantly less,
you need to upgrade your database instance size.

```SQL
SELECT SUM(HEAP_BLKS_READ) AS HEAP_READ,
	SUM(HEAP_BLKS_HIT) AS HEAP_HIT,
	SUM(HEAP_BLKS_HIT) / (SUM(HEAP_BLKS_HIT) + SUM(HEAP_BLKS_READ) + 0.00001) AS RATIO
FROM PG_STATIO_USER_TABLES;
```

## Index usage

Show a list of tables and the percentage of time they use an index:
```SQL
SELECT RELNAME,
	100 * IDX_SCAN / (SEQ_SCAN + IDX_SCAN + 10e-6) PERCENT_OF_TIMES_INDEX_USED,
	N_LIVE_TUP ROWS_IN_TABLE
FROM PG_STAT_USER_TABLES
WHERE SEQ_SCAN + IDX_SCAN > 0
ORDER BY N_LIVE_TUP DESC;
```
if you’re not somewhere around 99% on any table over 10,000 rows you may want to consider adding an index

## Index Cache Hit Rate

```SQL
SELECT SUM(IDX_BLKS_READ) AS IDX_READ,
	SUM(IDX_BLKS_HIT) AS IDX_HIT,
	(SUM(IDX_BLKS_HIT) - SUM(IDX_BLKS_READ)) / (SUM(IDX_BLKS_HIT) + 10e-6) AS RATIO
FROM PG_STATIO_USER_INDEXES;
```

### Resources

- https://www.craigkerstiens.com/2012/10/01/understanding-postgres-performance/
