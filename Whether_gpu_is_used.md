```
benchmark1=# create table j1 (c1 smallint, c2 smallint);
CREATE TABLE
benchmark1=# insert into j1 select generate_series(1,10000), random()*32767/10;
INSERT 0 10000
benchmark1=# insert into j2^Celect generate_series(1,10000), random()*32767/10;
benchmark1=# 
benchmark1=# 
benchmark1=# create table j2 (c1 smallint, c2 smallint);
CREATE TABLE
benchmark1=# insert into j2 select generate_series(1,10000), random()*32767/10;
INSERT 0 10000
benchmark1=# create table j3 (c1 smallint, c2 smallint);
CREATE TABLE
benchmark1=# insert into j3 select generate_series(1,10000), random()*32767/10;
INSERT 0 10000

benchmark1=# set enable_hashjoin = off;
SET
benchmark1=# set enable_mergejoin = off;
SET
benchmark1=# explain analyze
benchmark1-#            select j1.c1 a, j1.c2 b, j2.c1 c, j2.c2 d, j3.c1 e, j3.c2 f
benchmark1-#              into pg_temp.hoge1
benchmark1-#              from j1 join j2 on j1.c2 = j2.c2 left join j3 on j2.c2 = j3.c2;
                                                          QUERY PLAN                                                           
-------------------------------------------------------------------------------------------------------------------------------
 Custom Scan (GpuJoin) on j1  (cost=5603.16..5603.16 rows=116561 width=12) (actual time=2974.811..2983.622 rows=96499 loops=1)
   GPU Projection: j1.c1, j1.c2, j2.c1, j2.c2, j3.c1, j3.c2
   Outer Scan: j1  (cost=0.00..145.00 rows=10000 width=4) (actual time=0.523..0.523 rows=10000 loops=1)
   Depth 1: GpuHashJoin  (plan nrows: 10000...31794, actual nrows: 10000...30602)
            HashKeys: j1.c2
            JoinQuals: (j1.c2 = j2.c2)
            KDS-Hash (size plan: 713.00KB, exec: 634.88KB)
   Depth 2: GpuHashLeftJoin  (plan nrows: 31794...116561, actual nrows: 30602...96499)
            HashKeys: j2.c2
            JoinQuals: (j2.c2 = j3.c2)
            KDS-Hash (size plan: 728.51KB, exec: 634.88KB)
   ->  Seq Scan on j2  (cost=0.00..145.00 rows=10000 width=4) (actual time=0.010..0.793 rows=10000 loops=1)
   ->  Seq Scan on j3  (cost=0.00..159.75 rows=11475 width=4) (actual time=0.007..0.775 rows=10000 loops=1)
 Planning time: 0.276 ms
 Execution time: 3204.916 ms
(15 rows)
```

disable gpu accelerating

```
benchmark1=# SET pg_strom.enabled = off;
SET
benchmark1=# set enable_hashjoin = off;
SET
benchmark1=# set enable_mergejoin = off;
SET
benchmark1=# explain analyze select j1.c1 a, j1.c2 b, j2.c1 c, j2.c2 d, j3.c1 e, j3.c2 f into pg_temp.hoge1 from j1 join j2 on j1.c2 = j2.c2 left join j3 on j2.c2 = j3.c2;
                                                       QUERY PLAN                                                       
------------------------------------------------------------------------------------------------------------------------
 Nested Loop Left Join  (cost=0.00..6269585.00 rows=101616 width=12) (actual time=0.631..51260.832 rows=96499 loops=1)
   Join Filter: (j2.c2 = j3.c2)
   Rows Removed by Join Filter: 305924983
   ->  Nested Loop  (cost=0.00..1500315.00 rows=31794 width=8) (actual time=0.600..11473.549 rows=30602 loops=1)
         Join Filter: (j1.c2 = j2.c2)
         Rows Removed by Join Filter: 99969398
         ->  Seq Scan on j1  (cost=0.00..145.00 rows=10000 width=4) (actual time=0.015..2.859 rows=10000 loops=1)
         ->  Materialize  (cost=0.00..195.00 rows=10000 width=4) (actual time=0.000..0.430 rows=10000 loops=10000)
               ->  Seq Scan on j2  (cost=0.00..145.00 rows=10000 width=4) (actual time=0.013..0.929 rows=10000 loops=1)
   ->  Materialize  (cost=0.00..195.00 rows=10000 width=4) (actual time=0.000..0.557 rows=10000 loops=30602)
         ->  Seq Scan on j3  (cost=0.00..145.00 rows=10000 width=4) (actual time=0.011..1.017 rows=10000 loops=1)
 Planning time: 0.575 ms
 Execution time: 51316.924 ms
(13 rows)

```
