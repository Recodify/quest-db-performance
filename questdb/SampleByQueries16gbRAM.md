## Notes

- Queries were in order presented here, so data cached for latter ones? Mysql, yes. Quest?
- Mysql queries run via workbench
- QuestDb queries run via web console

## QuestDB 6.4.2

### Environment

- EC2 c5.2xlarge
- vCPU 8
- 16gb RAM
- Official AMI 6.4.2
- EBS 750gb - no provisioned IOPS

**NB: Single user dev server**

## QuestDB 6.5.3

### Environment

- Same EC instance as above but running in docker
- Putting to the same data `/var/lib/questdb/db`

**NB: Single user dev server**

## Queries

## Average temperature for single property `TestProperty123`, by device, by day YTD.

### QuestDb

```
SELECT
    readingDate,
    AVG(r.value),
    r.deviceId
FROM
    reading r
    INNER JOIN device d ON d.id = r.deviceid
    INNER JOIN property p ON p.id = d.propertyid
WHERE
    readingDate BETWEEN '2022-01-01 00:00:00.000' AND '2022-10-25 04:59:59.000'
    AND r.readingTypeId = 'environment.temperature.indoor'
    AND p.reference = 'TestProperty123'
    SAMPLE BY 1d;
```

**6.4.2 Metal**

First:

Second:

**6.4.2 Docker**

First:

Second:

**6.5.4 Docker**

First:

Second:


### MySql

```
SELECT
    readingYear,
    readingMonth,
    readingDay,
    avg(value) as value,
    device.id
FROM
    reading
    INNER JOIN device ON device.id = reading.deviceid
    INNER JOIN property ON device.propertyId = property.id
WHERE
    readingDate BETWEEN '2022-01-01 00:00:00.000' AND '2022-10-25 04:59:59.000'
    AND readingTypeId = 'environment.temperature.indoor'
    AND property.reference = 'TestProperty123'
GROUP BY
    device.id,
    readingYear,
    readingMonth,
    readingDay
```

First run:
- Duration: 12.045s
- Fetch: 0.017s

Second run:
- Duration: 1.225s
- Fetch: 0.016s

---

## Average temperature for single property `TestProperty123`, by device, by day MTD.

### QuestDb

```
SELECT
    readingDate,
    AVG(r.value),
    r.deviceId
FROM
    reading r
    INNER JOIN device d ON d.id = r.deviceid
    INNER JOIN property p ON p.id = d.propertyid
WHERE
    readingDate BETWEEN '2022-10-01 00:00:00.000' AND '2022-10-25 04:59:59.000'
    AND r.readingTypeId = 'environment.temperature.indoor'
    AND p.reference = 'TestProperty123'
    SAMPLE BY 1d;

```

**6.4.2 Metal**

First run:


Second run:


**6.4.2 Docker**
First run:


Second run:


**6.5.4 Docker**

First run:


Second run:


### MySql

```
select
  avg(value) as value,
  readingYear,
  readingMonth,
  readingDay,
  device.id
from
  reading
  inner join device on device.id = reading.deviceid
  inner join property on device.propertyId = property.id
where
  readingDate BETWEEN '2022-10-01 00:00:00.000' AND '2022-10-25 04:59:59.000'
  and readingTypeId = 'environment.temperature.indoor'
  and property.reference = 'TestProperty123'
group by
  device.id,
  readingYear,
  readingMonth,
  readingDay
```

First run:

Second run:


## Average temperaure for a single device `000255DE`, by day, MTD

### QuestDb

```
SELECT
    readingDate,
    AVG(r.value),
    r.deviceId
FROM
    reading r
    INNER JOIN device d ON d.id = r.deviceid
WHERE
    readingDate BETWEEN '2022-10-01 00:00:00.000' AND '2022-10-25 04:59:59.000'
    AND r.readingTypeId = 'environment.temperature.indoor'
    AND d.serialnumber = '000255DE'
    SAMPLE BY 1d;
```

**6.4.2 Metal**

First run:

Second run:



**6.5.4 Docker**

First run:


Second run:


### Mysql

```
select
  avg(value) as value,
  readingYear,
  readingMonth,
  readingDay,
  device.id
from
  reading
  inner join device on device.id = reading.deviceid
where
  readingDate BETWEEN '2022-10-01 00:00:00.000' AND '2022-10-25 04:59:59.000'
  and readingTypeId = 'environment.temperature.indoor'
  and device.serialnumber = '000255DE'
group by
  device.id,
  readingYear,
  readingMonth,
  readingDay
```

First run:
- Duration: 0.031s
- Fetch: 0.00002s

Second run:
- Duration: 00.17s
- Fetch: 0.00002s


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