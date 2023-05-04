---
title: "Scenario"
layout: default
nav_order: 2
has_children: false
---

# Scenario for all sessions


## Database model

### Description 

![Model](images/DatabaseModel.png)

### SQL to create the model

```sql
-- --------------------------------------------------------
-- Host:                         192.168.0.11
-- Server version:               8.0.32 - Homebrew
-- Server OS:                    macos13.0
-- HeidiSQL Version:             12.4.0.6659
-- --------------------------------------------------------

/*!40101 SET @OLD_CHARACTER_SET_CLIENT=@@CHARACTER_SET_CLIENT */;
/*!40101 SET NAMES utf8 */;
/*!50503 SET NAMES utf8mb4 */;
/*!40103 SET @OLD_TIME_ZONE=@@TIME_ZONE */;
/*!40103 SET TIME_ZONE='+00:00' */;
/*!40014 SET @OLD_FOREIGN_KEY_CHECKS=@@FOREIGN_KEY_CHECKS, FOREIGN_KEY_CHECKS=0 */;
/*!40101 SET @OLD_SQL_MODE=@@SQL_MODE, SQL_MODE='NO_AUTO_VALUE_ON_ZERO' */;
/*!40111 SET @OLD_SQL_NOTES=@@SQL_NOTES, SQL_NOTES=0 */;


-- Dumping database structure for yardsales
CREATE DATABASE IF NOT EXISTS `yardsales` /*!40100 DEFAULT CHARACTER SET utf8mb3 COLLATE utf8mb3_bin */ /*!80016 DEFAULT ENCRYPTION='N' */;
USE `yardsales`;

-- Dumping structure for table yardsales.Admins
CREATE TABLE IF NOT EXISTS `Admins` (
  `Login` varchar(50) COLLATE utf8mb3_bin NOT NULL,
  `Password` varchar(50) COLLATE utf8mb3_bin NOT NULL COMMENT 'not for production use!',
  `Name` varchar(100) COLLATE utf8mb3_bin NOT NULL,
  PRIMARY KEY (`Login`)
) ENGINE=InnoDB DEFAULT CHARSET=utf8mb3 COLLATE=utf8mb3_bin;

-- Data exporting was unselected.

-- Dumping structure for table yardsales.ItemCategories
CREATE TABLE IF NOT EXISTS `ItemCategories` (
  `Id` int unsigned NOT NULL AUTO_INCREMENT,
  `Name` varchar(50) COLLATE utf8mb3_bin NOT NULL,
  PRIMARY KEY (`Id`)
) ENGINE=InnoDB AUTO_INCREMENT=12 DEFAULT CHARSET=utf8mb3 COLLATE=utf8mb3_bin;

-- Data exporting was unselected.

-- Dumping structure for table yardsales.Locations
CREATE TABLE IF NOT EXISTS `Locations` (
  `Id` int unsigned NOT NULL,
  `Longitude` double NOT NULL DEFAULT '0',
  `Latitude` double NOT NULL DEFAULT '0',
  `Created` datetime DEFAULT NULL,
  PRIMARY KEY (`Id`),
  CONSTRAINT `FK_Locations_Participants` FOREIGN KEY (`Id`) REFERENCES `SalesParticipant` (`Id`) ON DELETE CASCADE
) ENGINE=InnoDB DEFAULT CHARSET=utf8mb3 COLLATE=utf8mb3_bin;

-- Data exporting was unselected.

-- Dumping structure for table yardsales.ParticipantItemCategories
CREATE TABLE IF NOT EXISTS `ParticipantItemCategories` (
  `IdParticipant` int unsigned NOT NULL,
  `IdCategory` int unsigned NOT NULL,
  `Comment` text CHARACTER SET utf8mb3 COLLATE utf8mb3_bin,
  PRIMARY KEY (`IdParticipant`,`IdCategory`),
  KEY `FK_Categories` (`IdCategory`),
  CONSTRAINT `FK_Categories` FOREIGN KEY (`IdCategory`) REFERENCES `ItemCategories` (`Id`),
  CONSTRAINT `FK_Participants` FOREIGN KEY (`IdParticipant`) REFERENCES `SalesParticipant` (`Id`) ON DELETE CASCADE
) ENGINE=InnoDB DEFAULT CHARSET=utf8mb3 COLLATE=utf8mb3_bin;

-- Data exporting was unselected.

-- Dumping structure for table yardsales.SalesParticipant
CREATE TABLE IF NOT EXISTS `SalesParticipant` (
  `Id` int unsigned NOT NULL AUTO_INCREMENT,
  `Email` varchar(100) COLLATE utf8mb3_bin NOT NULL,
  `SalesId` int unsigned NOT NULL,
  `Name` varchar(100) COLLATE utf8mb3_bin NOT NULL,
  `Street` varchar(100) COLLATE utf8mb3_bin NOT NULL,
  `Zip` char(5) COLLATE utf8mb3_bin NOT NULL DEFAULT '',
  `City` varchar(100) COLLATE utf8mb3_bin NOT NULL,
  `State` char(2) COLLATE utf8mb3_bin NOT NULL DEFAULT '',
  `MapUrl` varchar(1024) COLLATE utf8mb3_bin DEFAULT NULL,
  PRIMARY KEY (`Id`),
  KEY `FK_SALES` (`SalesId`),
  CONSTRAINT `FK_SALES` FOREIGN KEY (`SalesId`) REFERENCES `YardSales` (`Id`) ON DELETE CASCADE
) ENGINE=InnoDB AUTO_INCREMENT=14 DEFAULT CHARSET=utf8mb3 COLLATE=utf8mb3_bin;

-- Data exporting was unselected.

-- Dumping structure for table yardsales.YardSales
CREATE TABLE IF NOT EXISTS `YardSales` (
  `Id` int unsigned NOT NULL AUTO_INCREMENT,
  `EventStart` datetime NOT NULL,
  `EventEnd` datetime NOT NULL,
  `Title` varchar(255) COLLATE utf8mb3_bin NOT NULL DEFAULT '',
  `Logo` longblob,
  `Thumb` mediumblob,
  PRIMARY KEY (`Id`)
) ENGINE=InnoDB AUTO_INCREMENT=4 DEFAULT CHARSET=utf8mb3 COLLATE=utf8mb3_bin;

-- Data exporting was unselected.

/*!40103 SET TIME_ZONE=IFNULL(@OLD_TIME_ZONE, 'system') */;
/*!40101 SET SQL_MODE=IFNULL(@OLD_SQL_MODE, '') */;
/*!40014 SET FOREIGN_KEY_CHECKS=IFNULL(@OLD_FOREIGN_KEY_CHECKS, 1) */;
/*!40101 SET CHARACTER_SET_CLIENT=@OLD_CHARACTER_SET_CLIENT */;
/*!40111 SET SQL_NOTES=IFNULL(@OLD_SQL_NOTES, 1) */;
```