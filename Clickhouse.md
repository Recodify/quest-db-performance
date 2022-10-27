# Intro

More detail and depth available in other files but this is as simple a demonstration as there is.

Machines both had vCPU 8 / 16gb RAM

**Full joins Property YTD AVG bucketed by day**
- Questdb 116560ms / 116560ms
- Mysql 12045ms / 1225ms
- ClickHouse: 74ms

**No joins Device YTD AVG bucketed by day**
- Questdb 60020ms / 60020ms
- MySql 121ms
- ClickHouse 7ms / 4ms

**No joins Device MTD AVG bucketed by day**
- Questdb 250ms
- MySql 16ms / 16 ms
- ClickHouse 17ms* / 2ms

- * very first quest

## Average temperature for single property `TestProperty123`, by device, by day YTD.

###

```
SELECT
    year(readingDate),
    month(readingDate),
    day(readingDate),
    AVG(r.value),
    r.deviceId
FROM
    click.reading r
WHERE
    readingDate BETWEEN '2022-01-01 00:00:00.000' AND '2022-10-25 04:59:59.000'
    AND r.readingTypeId = 'environment.temperature.indoor'
    AND r.deviceId in (select id from click.device where propertyId in (select id from click.property where reference = 'AICO_HOMELINK_DEMO_CTO101'))
group by
   year(readingDate),
   month(readingDate),
   day(readingDate),
    r.deviceId
```

First: 277ms
Second: 74ms

## NO JOINS - Average temperaure for a single device `000255DE`, by day, MTD

### Clickhouse

```
select
  avg(value) as value,
   year(readingDate),
   month(readingDate),
   day(readingDate)
from
  click.reading
where
  readingDate BETWEEN '2022-10-01 00:00:00.000' AND '2022-10-25 04:59:59.000'
  and readingTypeId = 'environment.temperature.indoor'
  and reading.deviceId = '1ae9ccb4-c3ae-4287-8c50-41b604c95eff'
group by
   year(readingDate),
   month(readingDate),
   day(readingDate)
```

First: 17ms
Second: 2ms

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

### Clickhouse

```
select
  avg(value) as value,
   year(readingDate),
   month(readingDate),
   day(readingDate)
from
  click.reading
where
  readingDate BETWEEN '2022-01-01 00:00:00.000' AND '2022-10-25 04:59:59.000'
  and readingTypeId = 'environment.temperature.indoor'
  and reading.deviceId = '1ae9ccb4-c3ae-4287-8c50-41b604c95eff'
group by
   year(readingDate),
   month(readingDate),
   day(readingDate)
```

First: 7ms
Second: 4ms



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