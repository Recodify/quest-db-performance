# Intro

More detail and depth available in other files but this is as simple a demonstration as there is.

Machines both had vCPU 8 / 16gb RAM

**No joins YTD AVG bucketed by day: **
- Questdb 60020ms
- MySql 121ms

**No joins MTD AVG bucketed by day: **
- Questdb 250ms
- MySql 16ms

## NO JOINS - Average temperaure for a single device `000255DE`, by day, MTD

### QuestDb

```
SELECT
    readingDate,
    AVG(r.value)
FROM
    reading r
WHERE
    readingDate BETWEEN '2022-10-01 00:00:00.000' AND '2022-10-25 04:59:59.000'
    AND r.readingTypeId = 'environment.temperature.indoor'
    AND r.deviceId = '1ae9ccb4-c3ae-4287-8c50-41b604c95eff'
    SAMPLE BY 1d;

```

**6.4.2 Metal**

First run:

16 rows in 637ms
Execute: 539.05msNetwork: 97.95msTotal: 637ms

Second run:
16 rows in 613ms
Execute: 467.09msNetwork: 145.91msTotal: 613ms


**6.5.4 Docker**

First:
16 rows in 281ms
Execute: 243.87msNetwork: 37.13msTotal: 281ms


Second:

16 rows in 250ms
Execute: 184.72msNetwork: 65.28msTotal: 250ms


### MySql

```
select
  avg(value) as value,
  readingYear,
  readingMonth,
  readingDay
from
  reading
where
  readingDate BETWEEN '2022-10-01 00:00:00.000' AND '2022-10-25 04:59:59.000'
  and readingTypeId = 'environment.temperature.indoor'
  and reading.deviceId = '1ae9ccb4-c3ae-4287-8c50-41b604c95eff'
group by
  readingYear,
  readingMonth,
  readingDay

```

First run:
- Duration: 16ms
- Fetch: 0.00002s

Second run:
- Duration: 16ms
- Fetch: 0.00002s

## NO JOINS - Average temperaure for a single device `000255DE`, by day, YTD

### QuestDb

```
SELECT
    readingDate,
    AVG(r.value)
FROM
    reading r
WHERE
    readingDate BETWEEN '2022-01-01 00:00:00.000' AND '2022-10-25 04:59:59.000'
    AND r.readingTypeId = 'environment.temperature.indoor'
    AND r.deviceId = '1ae9ccb4-c3ae-4287-8c50-41b604c95eff'
    SAMPLE BY 1d;

```

**6.5.4 Docker**

First:
275 rows in 60.02s
Execute: 59.83sNetwork: 189.48msTotal: 60.02s

Second:
275 rows in 60.02s
Execute: 59.83sNetwork: 189.48msTotal: 60.02s


### MySql

```
select
  avg(value) as value,
  readingYear,
  readingMonth,
  readingDay
from
  reading
where
  readingDate BETWEEN '2022-01-01 00:00:00.000' AND '2022-10-25 04:59:59.000'
  and readingTypeId = 'environment.temperature.indoor'
  and reading.deviceId = '1ae9ccb4-c3ae-4287-8c50-41b604c95eff'
group by
  readingYear,
  readingMonth,
  readingDay

```

First run:
- Duration: 121ms
- Fetch: 0.000014s

Second run:
- Duration: 114ms
- Fetch: 0.000014s