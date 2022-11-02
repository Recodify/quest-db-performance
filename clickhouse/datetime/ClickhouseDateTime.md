# Intro

I have two tables that are exactly the same apart from one has a `DateTime64(3)` and the other has as a `DateTime`. When executing a query
against both, the one with the `DateTime` is slightly slower and looks at more rows! This is counter intuitive, can anyone help me understand why?

# Environment:

Version: 22.10.1.1877
Host: Docker/EC2/c5.2xlarge

# Schema

```
CREATE TABLE click.readingopx2
(
    `deviceId` Int32,
    `readingTypeId` Int32,
    `value` Float32,
    `readingDate` DateTime64(3) CODEC(DoubleDelta)
)
ENGINE = MergeTree
PRIMARY KEY (readingTypeId, deviceId, readingDate)
ORDER BY (readingTypeId, deviceId, readingDate)
SETTINGS index_granularity = 8192
```

```
CREATE TABLE click.readingopx7
(
    `deviceId` Int32,
    `readingTypeId` Int32,
    `value` Float32,
    `readingDate` DateTime CODEC(DoubleDelta)
)
ENGINE = MergeTree
PRIMARY KEY (readingTypeId, deviceId, readingDate)
ORDER BY (readingTypeId, deviceId, readingDate)
SETTINGS index_granularity = 8192
```

# Test Query

```
SELECT
    AVG(value) AS value,
    deviceId,
    readingTypeId,
    MIN(readingDate) AS DATE
FROM
    click.readingopx[7|2]
WHERE
    readingTypeId = dictGet('sensorium.readingType', 'seqId', 'environment.temperature.indoor')
    AND readingDate BETWEEN '2022-01-01 00:00:00' AND '2022-11-01 07:59:59'
    AND deviceId IN (SELECT dictGet('sensorium.device', 'seqId', id) FROM click.device WHERE propertyId IN (SELECT id FROM click.property WHERE reference = 'DEMO123'))
GROUP BY
    YEAR(readingDate),
    MONTH(readingDate),
    DAY(readingDate),
    readingTypeId,
    deviceId
```

## Results

After warm up query:

### DateTime64/readingopx2:
Elapsed: 0.020 sec, read 1.25 million rows, 17.67 MB

### DateTime/readingopx7:
Elapsed: 0.025 sec, read 1.77 million rows, 19.62 MB


# Extra details

## Table stats

| parts.table | rows      | disk_size | primary_keys_size | engine    | bytes_size | compressed_size | uncompressed_size | compression_ratio | compression_percentage |
|-------------|-----------|-----------|-------------------|-----------|------------|-----------------|-------------------|-------------------|------------------------|
| readingopx2 | 444211909 | 1.26 GiB  | 847.41 KiB        | MergeTree | 1357781672 | 1.26 GiB        | 8.27 GiB          | 0.152             | 84.785                 |
| readingopx7 | 444211909 | 1.13 GiB  | 635.63 KiB        | MergeTree | 1214418150 | 1.12 GiB        | 6.62 GiB          | 0.17              | 83                     |

## Query plan

### DateTime64/readingopx2:

	Expression ((Projection + Before ORDER BY))
	  Aggregating
	    Expression (Before GROUP BY)
	      Filter (WHERE)
	        ReadFromMergeTree (click.readingopx2)

### DateTime/readingopx7:

	Expression ((Projection + Before ORDER BY))
	  Aggregating
	    Expression (Before GROUP BY)
	      Filter (WHERE)
	        ReadFromMergeTree (click.readingopx7)


## Pipeline


### DateTime64/readingopx2:

	ExpressionTransform × 8
	  (Aggregating)
	  Resize 6 → 8
	    AggregatingTransform × 6
	      StrictResize 6 → 6
	        (Expression)
	        ExpressionTransform × 6
	          (Filter)
	          FilterTransform × 6
	            (ReadFromMergeTree)
	            MergeTreeThread × 6 0 → 1

### DateTime/readingopx7:

	ExpressionTransform × 8
	  (Aggregating)
	  Resize 8 → 8
	    AggregatingTransform × 8
	      StrictResize 8 → 8
	        (Expression)
	        ExpressionTransform × 8
	          (Filter)
	          FilterTransform × 8
	            (ReadFromMergeTree)
	            MergeTreeThread × 8 0 → 1

# benchmark

`clickhouse-benchmark -t 5 <<< "[QueryAbove]"`

### DateTime64/readingopx2:

```
Loaded 1 queries.

Queries executed: 30.

localhost:9000, queries 30, QPS: 29.252, RPS: 36463774.899, MiB/s: 493.055, result RPS: 67513.212, result MiB/s: 1.545.

0.000%          0.031 sec.
10.000%         0.032 sec.
20.000%         0.033 sec.
30.000%         0.033 sec.
40.000%         0.033 sec.
50.000%         0.034 sec.
60.000%         0.034 sec.
70.000%         0.035 sec.
80.000%         0.035 sec.
90.000%         0.036 sec.
95.000%         0.037 sec.
99.000%         0.046 sec.
99.900%         0.046 sec.
99.990%         0.046 sec.



Queries executed: 61.

localhost:9000, queries 31, QPS: 30.221, RPS: 37672433.441, MiB/s: 509.398, result RPS: 69751.061, result MiB/s: 1.596.

0.000%          0.030 sec.
10.000%         0.031 sec.
20.000%         0.031 sec.
30.000%         0.032 sec.
40.000%         0.032 sec.
50.000%         0.033 sec.
60.000%         0.033 sec.
70.000%         0.034 sec.
80.000%         0.035 sec.
90.000%         0.035 sec.
95.000%         0.036 sec.
99.000%         0.039 sec.
99.900%         0.039 sec.
99.990%         0.039 sec.



Queries executed: 92.

localhost:9000, queries 31, QPS: 30.125, RPS: 37552359.279, MiB/s: 507.774, result RPS: 69528.742, result MiB/s: 1.591.

0.000%          0.031 sec.
10.000%         0.032 sec.
20.000%         0.032 sec.
30.000%         0.032 sec.
40.000%         0.033 sec.
50.000%         0.033 sec.
60.000%         0.034 sec.
70.000%         0.034 sec.
80.000%         0.034 sec.
90.000%         0.035 sec.
95.000%         0.037 sec.
99.000%         0.037 sec.
99.900%         0.037 sec.
99.990%         0.037 sec.



Queries executed: 123.

localhost:9000, queries 31, QPS: 30.182, RPS: 37623703.409, MiB/s: 508.739, result RPS: 69660.837, result MiB/s: 1.594.

0.000%          0.030 sec.
10.000%         0.031 sec.
20.000%         0.032 sec.
30.000%         0.032 sec.
40.000%         0.032 sec.
50.000%         0.033 sec.
60.000%         0.033 sec.
70.000%         0.033 sec.
80.000%         0.034 sec.
90.000%         0.035 sec.
95.000%         0.037 sec.
99.000%         0.039 sec.
99.900%         0.039 sec.
99.990%         0.039 sec.


Stopping launch of queries. Requested time limit is exhausted.

Queries executed: 152.

localhost:9000, queries 152, QPS: 30.066, RPS: 37478950.319, MiB/s: 506.782, result RPS: 69392.825, result MiB/s: 1.588.

0.000%          0.030 sec.
10.000%         0.031 sec.
20.000%         0.032 sec.
30.000%         0.032 sec.
40.000%         0.033 sec.
50.000%         0.033 sec.
60.000%         0.033 sec.
70.000%         0.034 sec.
80.000%         0.035 sec.
90.000%         0.035 sec.
95.000%         0.036 sec.
99.000%         0.039 sec.
99.900%         0.046 sec.
99.990%         0.046 sec.
```

### DateTime/readingopx7:

```
Loaded 1 queries.

Queries executed: 29.

localhost:9000, queries 29, QPS: 28.786, RPS: 50975410.050, MiB/s: 538.579, result RPS: 66438.288, result MiB/s: 1.267.

0.000%          0.032 sec.
10.000%         0.033 sec.
20.000%         0.033 sec.
30.000%         0.033 sec.
40.000%         0.034 sec.
50.000%         0.035 sec.
60.000%         0.035 sec.
70.000%         0.035 sec.
80.000%         0.036 sec.
90.000%         0.037 sec.
95.000%         0.038 sec.
99.000%         0.041 sec.
99.900%         0.041 sec.
99.990%         0.041 sec.



Queries executed: 59.

localhost:9000, queries 30, QPS: 29.235, RPS: 51770317.750, MiB/s: 546.977, result RPS: 67474.323, result MiB/s: 1.287.

0.000%          0.032 sec.
10.000%         0.032 sec.
20.000%         0.033 sec.
30.000%         0.033 sec.
40.000%         0.034 sec.
50.000%         0.034 sec.
60.000%         0.035 sec.
70.000%         0.035 sec.
80.000%         0.035 sec.
90.000%         0.036 sec.
95.000%         0.037 sec.
99.000%         0.038 sec.
99.900%         0.038 sec.
99.990%         0.038 sec.



Queries executed: 89.

localhost:9000, queries 30, QPS: 29.064, RPS: 51467270.334, MiB/s: 543.775, result RPS: 67079.350, result MiB/s: 1.279.

0.000%          0.032 sec.
10.000%         0.033 sec.
20.000%         0.033 sec.
30.000%         0.034 sec.
40.000%         0.034 sec.
50.000%         0.035 sec.
60.000%         0.035 sec.
70.000%         0.035 sec.
80.000%         0.035 sec.
90.000%         0.036 sec.
95.000%         0.037 sec.
99.000%         0.037 sec.
99.900%         0.037 sec.
99.990%         0.037 sec.



Queries executed: 119.

localhost:9000, queries 30, QPS: 29.126, RPS: 51577913.942, MiB/s: 544.944, result RPS: 67223.556, result MiB/s: 1.282.

0.000%          0.033 sec.
10.000%         0.033 sec.
20.000%         0.033 sec.
30.000%         0.034 sec.
40.000%         0.034 sec.
50.000%         0.034 sec.
60.000%         0.035 sec.
70.000%         0.035 sec.
80.000%         0.035 sec.
90.000%         0.036 sec.
95.000%         0.036 sec.
99.000%         0.036 sec.
99.900%         0.036 sec.
99.990%         0.036 sec.



Queries executed: 146.

localhost:9000, queries 146, QPS: 28.984, RPS: 51326548.205, MiB/s: 542.289, result RPS: 66895.941, result MiB/s: 1.276.

0.000%          0.032 sec.
10.000%         0.033 sec.
20.000%         0.033 sec.
30.000%         0.034 sec.
40.000%         0.034 sec.
50.000%         0.034 sec.
60.000%         0.035 sec.
70.000%         0.035 sec.
80.000%         0.035 sec.
90.000%         0.036 sec.
95.000%         0.037 sec.
99.000%         0.038 sec.
99.900%         0.041 sec.
99.990%         0.041 sec.
```