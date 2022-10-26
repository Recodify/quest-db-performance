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