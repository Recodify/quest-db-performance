## Notes

- Queries were in order presented here, so data cached for latter ones? Mysql, yes. Quest?
- Mysql queries run via workbench
- QuestDb queries run via web console

## QuestDB 6.4.2

### Environment

- EC2 c5.xlarge
- vCPU 4
- 8gb RAM
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

First run:
- 1,452 rows in 60.23s
- Execute: 60.17s
- Network: 59.11ms
- Total: 60.23s
- Count: 19.91s
- Compile: 45.54ms

Second run:
- 1,452 rows in 60.32s
- Execute: 60.1s
- Network: 214.36ms
- Total: 60.32s
- Count: 19.89s
- Compile: 0

**6.4.2 Docker**

(WAT?!)
2,298 rows in 116.56s
Execute: 116.22sNetwork: 333.17msTotal: 116.56s
Count: 75.35sCompile: 51.34ms


**6.5.4 Docker**

(WAT?!)
2,298  rows in 126.16s
Execute: 126.06s
Network: 99.6ms
Total: 126.16s




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
- 216 rows in 16.56s
- Execute: 16.42s
- Network: 145.52ms
- Total: 16.56s
- Count: 19.89s (this value is always the same and longer than total?)
- Compile: 0

Second run:

- 216 rows in 4.45s
- Execute: 4.34s
- Network: 108.02ms
- Total: 4.45s
- Count: 19.89s
- Compile: 0

**6.4.2 Docker**
First run:

216 rows in 4.39s
Execute: 4.27sNetwork: 119.19msTotal: 4.39s

Second run:
216 rows in 4.07s
Execute: 4.05sNetwork: 22.98msTotal: 4.07s

**6.5.4 Docker**

First run:
216 rows in 4.56s
Execute: 4.26sNetwork: 297.56msTotal: 4.56s

Second run:
216 rows in 4.26s
Execute: 4.14sNetwork: 111.61msTotal: 4.26s
Count: 0Compile: 0


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

- Duration: 0.360s
- Fetch: 0.00012s

Second run:
- Duration: 0.270s
- Fetch: 0.0004s

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
- 16 rows in 1.81s
- Execute: 1.55s
- Network: 259.41ms
- Total: 1.81s
- Count: 19.89s
- Compile: 0

Second run:

- 16 rows in 1.59s
- Execute: 1.54s
- Network: 42.51ms
- Total: 1.59s
- Count: 19.89s
- Compile: 0

**6.5.4 Docker**

First run:
16 rows in 1.96s
Execute: 1.85sNetwork: 109.83msTotal: 1.96s

Second run:
16 rows in 1.71s
Execute: 1.62sNetwork: 88.8msTotal: 1.71s

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
- 16 rows in 1.19s
- Execute: 807.46ms
- Network: 380.54ms
- Total: 1.19s -
- Count: 19.89s
- Compile: 1.63ms

Second run:
- 16 rows in 856ms
- Execute: 812.13ms
- Network: 43.87ms
- Total: 856ms
- Count: 19.89s
- Compile: 0

**6.5.4 Docker**

First:
16 rows in 588ms
Execute: 507.35msNetwork: 80.65msTotal: 588ms

Second:
16 rows in 433ms
Execute: 342msNetwork: 91msTotal: 433ms

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
- Duration: 0.016s
- Fetch: 0.00002s

Second run:
- Duration: 00.16s
- Fetch: 0.00002s
