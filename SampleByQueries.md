## QuestDB

### Notes

- Queries were in order presented here, so data cached for latter ones? Mysql, yes. Quest?
- Mysql queries run via workbench
- QuestDb queries run via web console

### Environment

- EC2 c5.xlarge
- Official AMI 6.4.2
- EBS 750gb - no provisioned IOPS

**NB: Single user dev server**

### Schema

```
CREATE TABLE 'property' (
  id STRING,
  reference STRING,
  displayReference STRING,
  landlordId STRING
);
```
```
CREATE TABLE 'device' (
  id STRING,
  serialnumber STRING,
  propertyId STRING,
  locationId STRING
);
```
```
CREATE TABLE 'reading' (
  deviceId STRING,
  readingTypeId SYMBOL capacity 256 CACHE,
  value FLOAT,
  readingDate TIMESTAMP
) timestamp (readingDate) PARTITION BY DAY;
```

### Data Volume

- Property: 26044 rows
- Devices: 197478 rows
- Readings: 444258950 rows // size on disk: 42gb

## MySql

### Environment

- RDS t3.xlarge
- MySql 8.0.28
- EBS 605gb - no provisioned IOPS

**NB: Live production server so has other workloads**

### Schema

```
CREATE TABLE `landlord` (
  `id` char(36) NOT NULL,
  `name` varchar(255) NOT NULL,
  `reference` varchar(255) NOT NULL,
  PRIMARY KEY (`id`),
  UNIQUE KEY `reference` (`reference`)
) ENGINE=InnoDB DEFAULT CHARSET=latin1;
```

```
CREATE TABLE property (
    `id` char(36) NOT NULL,
    `reference` varchar(255) NOT NULL,
    `displayReference` varchar(255) DEFAULT NULL,
    `landlordId` char(36) NOT NULL,
	PRIMARY KEY (`id`),
    UNIQUE KEY `reference` (`reference`),
    KEY `idx_propertyReference` (`reference`),
    KEY `landlordId` (`landlordId`),
    CONSTRAINT `property_landlord` FOREIGN KEY (`landlordId`) REFERENCES `landlord` (`id`) ON DELETE RESTRICT ON UPDATE RESTRICT
) ENGINE=InnoDB DEFAULT CHARSET=latin1;
```

```
CCREATE TABLE `device` (
  `id` char(36) NOT NULL,
  `serialNumber` varchar(255) NOT NULL,
  `propertyId` char(36) NOT NULL,
  `locationId` varchar(255) NOT NULL,
  PRIMARY KEY (`id`),
  UNIQUE KEY `serialNumber` (`serialNumber`),
  KEY `idx_devicePropertyId` (`propertyId`),
  CONSTRAINT `device_property` FOREIGN KEY (`propertyId`) REFERENCES `property` (`id`) ON DELETE RESTRICT ON UPDATE RESTRICT
) ENGINE=InnoDB DEFAULT CHARSET=latin1;
```

```
CREATE TABLE `reading` (
  `id` bigint NOT NULL AUTO_INCREMENT,
  `deviceId` char(36) NOT NULL,
  `value` decimal(10,4) NOT NULL,
  `readingTypeId` varchar(255) NOT NULL,
  `readingDate` datetime(3) NOT NULL,
  `readingYear` int GENERATED ALWAYS AS (year(`readingDate`)) STORED,
  `readingMonth` int GENERATED ALWAYS AS (month(`readingDate`)) STORED,
  `readingDay` int GENERATED ALWAYS AS (dayofmonth(`readingDate`)) STORED,
  `readingHour` int GENERATED ALWAYS AS (hour(`readingDate`)) STORED,
  `readingMinute` int GENERATED ALWAYS AS (minute(`readingDate`)) STORED,
  PRIMARY KEY (`id`),
  KEY `idx_readingDeviceTypeReadingDateDesc` (`deviceId`,`readingTypeId`,`readingDate`),
  CONSTRAINT `reading_ibfk_1` FOREIGN KEY (`deviceId`) REFERENCES `device` (`id`) ON DELETE RESTRICT ON UPDATE RESTRICT
) ENGINE=InnoDB AUTO_INCREMENT=523426979 DEFAULT CHARSET=latin1;
```

### Data Volume

*live server so at least what questdb has plus some*
- Property: 26044++ rows
- Devices: 197478++ rows
- Readings: 444258950++ rows


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
    AND p.reference = 'AICO_HOMELINK_DEMO_CTO101'
    SAMPLE BY 1d;

```

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
  and property.reference = 'AICO_HOMELINK_DEMO_CTO101'
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

First run:
- 6 rows in 1.19s
- Execute: 807.46ms
- Network: 380.54ms
- Total: 1.19s -
- Count: 19.89s
- Compile: 1.63ms

Second run:
- 6 rows in 856ms
- Execute: 812.13ms
- Network: 43.87ms
- Total: 856ms
- Count: 19.89s
- Compile: 0


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