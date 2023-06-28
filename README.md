# Brains4Buildings2022 dataset 
This repository contains a dataset with CO₂ concentration, ventilation flow rate and occupancy data of 3 office spaces at Windesheim University of Applied Sciences collected for the Brains4Buildings project.

## Table of contents
* [General info](#general-info)
* [Data management](#data-management)
* [Subject recruitment](#subject-recruitment)
* [Inclusion criteria](#inclusion-criteria)
* [Metadata](#metadata) 
* [Data](#data) 
* [Status](#status)
* [License](#license)
* [Credits](#credits)

## General info

For the [Brains4Buildings project](https://brains4buildings.org/), we collected data about CO₂ concentration, ventilation flow rate and occupancy for several weeks between October 10 and November 2, 2022. This data was collected to answer two research questions:

A. To what extent can we reliably derive the ventilation flow rate in a room from:
 * Measured time series data about the occupancy in that room, and
 * Measured data about the CO₂ concentration in a room?

B.	To what extent can we reliably derive occupancy in a room based on:
 * Measured data about the ventilation flow rate in a room, and 
 * Measured data about the CO₂ concentration in a room?

## Data management

Before we started recruitment, we requested and [obtained approval for our study from Windesheim Research Ethics Committee](https://edu.nl/vfcgp), based on a description of the research, the [privacy policy](https://edu.nl/x8wpd) and [Data Management Policy](https://edu.nl/y7tyx). 

## Subject recruitment

Subjects were recruited via a recruitment e-mail targeted at people known to work in office rooms that satisfied the [inclusion criteria](#inclusion-criteria) for office rooms.

## Inclusion criteria

Inclusion criteria for office rooms at Windesheim  were:
* PIR-based occupancy data, CO₂ concentration data and ventilation data was available via the existing Building Management System (BMS);
* 75% or more of the occupants of an office room must consent to their presence being tracked.  

Inclusion criteria for subjects were:
* subjects provided informed by filling out an [online recruitment survey](https://edu.nl/cnh8v), ([also available in Qualtrics qsf-format](https://edu.nl/t6pbv)), which also referred to the [privacy policy](https://edu.nl/x8wpd) and verified the inclusion criteria below;
* subjects must work at Windesheim University of Applied Sciences in one of the eligible office rooms, typically for more than one hour per week;
* subjects must have a smartphone running Android or an Apple iPhone;
* subjects must give the static Bluetooth MAC address of their smartphone to the researchers for the purpose of the research;
* subjects must be willing to turn / leave on their Bluetooth on their smartphone when they were at Windesheim.

We only installed a [Twomes M5Stack CoreInk + SCD41](https://edu.nl/ca8py) measurement device in an office room if more than 75% of the known number of regular office room occupants provided informed consent. We never tracked presence via Bluetooth without informed consent; the measurement devices we use are technically incapable of tracking Bluetooth based presence without a static Bluetooth MAC-address.

## Metadata

The file [b4b-room-metadata.zip](metadata/b4b-room-metadata.zip) contains metadata that may be needed for analysis for each of the three rooms in the open dataset. 

| id     | room__m3 <sup>1</sup> | vent_max__m3_h_1 <sup>2</sup>|
|--------|----------|------------------|
| 999169 | 75       | 240              |
| 917810 | 75       | 240              |
| 925038 | 60       | 210              |

<sup>1</sup>, <sup>2</sup>: We rounded `room__m3`, the room volume in  m<sup>3</sup>, to the nearest 5 m<sup>3</sup>, and `vent_max__m3_h_1`, the maximum ventilation flow rate of the room in m<sup>3</sup>/h, to the nearest 30 to the nearest 5 m<sup>3</sup>/h we can guarantee room privacy. In particular, we chose a level of privacy for the room that is equivalent to the level of privacy required for persons participating in medical research in the Netherlands, i.e. the chance of re-identification should be less than 9%.


## Data

In the sections below, the data pre-processing and data formats used in the data files will be described.

### Measurement devices and other data sources 

We used the following measurement device types and data sources to collect data.

| Source/Device type name | Description                                | Open source repository of device                             |
| ----------------------- | ------------------------------------------ | ------------------------------------------------------------ |
| `CO2-meter-SCD4x`       | M5Stack CoreInk SCD41 measurement device for CO₂ and occupancy | [twomes-scd41-presence-firmware](https://edu.nl/ca8py) |
| `bms`                   | building management system                 |                                                              |
| `xovis`                 | Xovis PC2SE 3Dsensor                       |                                                              |
| `human_observer`        | data collected by human observers          |                                                              |

### Date and time information

All timestamps collected by the `CO2-meter-SCD4x` devices were measured in [Unix time](https://en.wikipedia.org/wiki/Unix_time) format, using device clocks synchronized  via NTP with the correct UTC time, immediately after the measuement device was installed and connected to the itnernet and every 6 hours after that. Uploads of measurement data (which could contain more than one measurement) were timestamped both by the measurement device according to the local device clock and by the server. We did not yet check for deviations between the last device timestamp of a measurement upload and the upload timestamp at the server.

Timestamps of `bms` and `xovis` sources were recorded in the [Europe/Amsterdam](https://en.wikipedia.org/wiki/Time_in_the_Netherlands) time zone. 

Timestamps collected by human observers were also registered in the [Europe/Amsterdam](https://en.wikipedia.org/wiki/Time_in_the_Netherlands) time zone.

Timestamps were converted to a time zone aware `pandas.Timestamp` value, in the [Europe/Amsterdam](https://en.wikipedia.org/wiki/Time_in_the_Netherlands) time zone. In the csv files we use [ISO 8601 format with time offset](https://en.wikipedia.org/wiki/ISO_8601): `YYYY-MM-DDThh:mm:ss±hhmm`.


### Raw measurements 
 Raw masurements will be available in the folder [/raw-measurements/](/raw-measurements/) in three formats:

 - [b4b_raw_measurements.parquet](/raw-measurements/b4b_raw_measurements.parquet): a single [parquet](https://parquet.apache.org/) file with data for all ids;
 - nnnnnn_raw_measurements.parquet: 3 [parquet](https://parquet.apache.org/) files, one for each id;
 - nnnnnn_raw_measurements.zip: 3 [zip](https://en.wikipedia.org/wiki/ZIP_(file_format))ped [csv](https://en.wikipedia.org/wiki/Comma-separated_values) files, one for each id;

All measurement data is structured according to the table below. By importing the parquet variant using [pandas.read_parquet()](https://pandas.pydata.org/pandas-docs/stable/reference/api/pandas.read_parquet.html), you automatically get a DataFrame with the recommended indices and data types. 

Alternatively, you can also read the zipped csv files, but this typically takes much longer.

| **Index/Column** | **Name** | **Type**     | **Description**                                                                                                                                       |
| ---------- | --------------- | ------------ | ----------------------------------------------------------------------------------------------------------------------------------------------------- |
| index       | `id`            | `category`   | unique code of the home                                                                                                                               |
| index       | `device_name`   | `category`   | unique name of the measurement device                                                                                                                 |
| index       | `source`        | `category`   | [device type name](###measurement-devices) of the measurement device                                                                                  |
| index        | `timestamp`     | `Timestamp` | start of the interval (time zone aware) |
| index       | `property`      | `category`   | property name of the measurement                                                                                                                      |
| column      | `value`         | `object`     | value of the measurement                                                                                                                              |
| column      | `unit`          | `category`   | unit of the measurement value                                                                                                                         |

### Raw properties 
 In the folder [/raw-properties/](raw_properties) we will make various measured properties available in an 'unstacked' format with each property in its own column and an appropriate datatype. Similar to measurements, we will make data available in three formats:

 - [b4b_raw_properties.parquet](raw_properties\b4b_raw_properties.parquet): a single [parquet](https://parquet.apache.org/) file with data for all ids;
 - nnnnnn_raw_properties.parquet: 3 [parquet](https://parquet.apache.org/) files, one for each id;
 - nnnnnn_raw_properties.zip: 3 [zip](https://en.wikipedia.org/wiki/ZIP_(file_format))ped [csv](https://en.wikipedia.org/wiki/Comma-separated_values) files, one for each id;

All property data is structured according to the table below. By importing the parquet variant using [pandas.read_parquet()](https://pandas.pydata.org/pandas-docs/stable/reference/api/pandas.read_parquet.html), you automatically get a DataFrame with the recommended indices and data types. 

Alternatively, you can also read the zipped csv files, but this typically takes much longer.

| **Index/Column** | **Name**                             | **Type**    | **Description**                                              |
| ---------------- | ------------------------------------ | ----------- | ------------------------------------------------------------ |
| index            | `id`                                 | `category`  | unique code of the home                                      |
| index            | `source`                             | `category`  | [device type name](###measurement-devices) of the measurement device |
| index            | `timestamp`                          | `Timestamp` | start of the interval (time zone aware)                       |
| column           | property_1; see property table below | data_type_1 | measured value of this property                              |
| column           | property2                            | data_type_2 | measured value of this property                              |
| ...              | ...                                  | ...         | ...                                                          |
| column           | property_n                           | data_type_n | measured value of this property                              |

### Measured Properties 

Below is a table that lists all properties that were measured, the data type in the [raw-properties](#raw-poperties) DataFrame, the measurement unit, the measurement interval, the source device and sensor that measured it, as well as the property name and value format as retrieved from the Twomes database.

| Property          | Pandas Type      | Unit | Measurement interval \[h:mm:ss\] | Description                                                | Source            | Sensor                                                       | Database property  | [Database format](https://en.wikipedia.org/wiki/Printf_format_string) |
| ----------------- | --------- | ---- | -------------------------------- | ---------------------------------------------------------- | ----------------- | ------------------------------------------------------------ | ------------------ | ------------------------------------------------------------ |
| `co2__ppm`        | `float32` | ppm  | 0:10:00                          | CO₂ concentration                                          | `CO2-meter-SCD4x` | [SCD41](https://sensirion.com/products/catalog/SCD41/)       | `CO2concentration` | %u                                                           |
| `temp_in__degC`   | `float32` | ˚C   | 0:10:00                          | air temperature                                            | `CO2-meter-SCD4x` | [SCD41](https://sensirion.com/products/catalog/SCD41/)       | `roomTemp`         | %.1f                                                         |
| `rel_humidity__0` | `float32` | -    | 0:10:00                          | relative humidity                                          | `CO2-meter-SCD4x` | [SCD41](https://sensirion.com/products/catalog/SCD41/)       | `relativeHumidity` | %.1f                                                         |
| `occupancy__p`    | `Int8`    | -    | 0:10:00                          | number of smartphones responding to Bluetooth name request | `CO2-meter-SCD4x` | ESP32                                                        | `countPresence`    | %u                                                           |
| `co2__ppm`        | `float32` | ppm  | 0:01:00/1:00:00                  | CO₂ concentration                                          | `bms`             |                                                              |                    |                                                              |
| `temp_in__degC`   | `float32` | ˚C   | 0:01:00/1:00:00                  | air temperature                                            | `bms`             |                                                              |                    |                                                              |
| `rel_humidity__0` | `float32` | -    | 0:01:00/1:00:00                  | relative humidity                                          | `bms`             |                                                              |                    |                                                              |
| `valve_frac__0` <sup>1</sup>| `float32` | -    | 0:01:00/1:00:00                  | valve opening fraction*                                          | `bms`             |                                                              |                    |                                                              |
| `occupancy__bool` | `Int8`    | 0/1  | 0:01:00/1:00:00                  | at least 1 person in the room? | `bms`             | PIR                                                          |                    |                                                              |
| `occupancy__p`    | `Int8`    | 0    | 0:05:00/0:15:00                  | number of persons in room                                   | `xovis`           | [PC2SE](https://www.xovis.com/technology/sensor/pc2se-sensor) |                    |                                                              |                                                   |
| `occupancy__bool` | `Int8`    | 0/1  | a few times per week | at least 1 person in the room? | `human_observer`             | PIR                                                          |                    |                                                              |
| `occupancy__bool` | `Int8`    | 0/1  | a few times per week | at least 1 person in the room? | `human_observer`             | human  observer |                    |                                                              |
| `window_open__bool` | `Int8`    | 0/1  | a few times per week | at least window in the room open? | `human_observer`             | human observer |                    |                                                              |
| `door_open__bool` | `Int8`    | 0/1  | a few times per week | at least door in the room open? | `human_observer`             | human observer |                    |                                                              |

<sup>1</sup>: `valve_frac__0` contains a fraction, i.e. a float between 0 and 1 that expressed the fraction of the maximum ventilation flow in that room;
- This value was derived from a voltage value registered by the Building Management System (BMS):
- The ventilation valves are physically constrained such that when the valve position voltage was registered at 0 V, the minimum opening of 20% is in effect (i.e. `valve_frac__0` = 0.20) and when the valve valve-position voltage of 10V means the valve is 100% open (i.e. `valve_frac__0` = 1.00).
- The ventilation system uses a regulator that makes sure there is a constant pressure, so the ventilation flow rate is not dependent on recent valve openings in nearby rooms, or only for very brief moments a few seconds, or in any case within a minute or so.
- We measured data in 6 rooms in 2 different buildings, each with a different BMS. For one building, the valve fraction data were only available for export for 24h after they were recorded. This initially implied we would not have retrospective `valve_frac__0` data for 4 of the 6 rooms we measured. Since the covid-19 pandemic, however, ventilation systems were set to max. At least, that was the intention. This was implemented by setting the ventilation setpoint to 400 ppm, implying the `valve_frac__0` = 1.00 at all ties would be a reasonable assumption. Unfortunately, for 3 out of the 4 rooms in the building concerned, the room CO₂ sensors connected the bms were not calibrated properly: they often registered CO₂ concentration values well below 400 ppm, values that seem highly unlikely when looking at the [Keeling Curve](https://keelingcurve.ucsd.edu/) of the last 5 years or so. Therefore, we had to leave out 3 rooms from our analyses. We decided to leave them out of this open dataset as well.

### Preprocessed data 

TO DO: change preprocessing description below.

Preprocessing of measurements from the measurement database was done using [get_preprocessed_b4b_data()](https://github.com/energietransitie/twomes-twutility-inverse-grey-box-analysis/blob/main/data/extractor.py). Preprocessing steps include:
- removal of duplicate measurements;
- calculation of derived properties as a combination of other properties, as indicated in the column `Calculation` in the table below (none, in this particular dataset);
- removal of absolute outliers, i.e., measurement values smaller than the value in the column `Min` or larger than the value in the column `Max` in the table below;
- removal of statistic outliers, i.e., measurement values with an absolute [z-score](https://en.wikipedia.org/wiki/Standard_score) higher than the value indicated in the `Sigma` column in the table below (none, for this particular dataset);
- removal of measurements for properties for ids where the variance is not above the threshold indicated in the `Minimum standard deviation` column in the table below; 
- interpolation of measurements to intervals of 15 minutes (no interpolation between measurements that were 90 minutes apart or more);
- All column values represent the average during the interval that starts at the timestamp indicated. 


| **Index/  Column** | **Name**    | **Type**    | **Unit** | **Description**                                      | **Calculation** | Min    | Max    | Sigma | Minimum standard deviation |
|--------------------|-------------|-------------|----------|------------------------------------------------------|-----------------|--------|--------|-------|----------------------------|
| index              | `id`        | `Int16`     |          | unique code of the room                              |                 | 900000 | 999999 |       |                            |
| index              | `timestamp` | `Timestamp` |          | start of the interpolated interval (time zone aware) |                 |        |        |       |                            |
| column             | `co2__ppm`  | `float32`   | ppm      |                                                      |                 | 5      |        |       | >0                         |

## Status
Dataset is: _collected_, _published as open data_

## License
This data is made available under the [CC BY 4.0](./LICENSE.md) by the [Research group Energy Transition, Windesheim University of Applied Sciences](https://windesheim.nl/energietransitie) 

## Credits

Data collection was a joint effort of:
* Henri ter Hofte · [@henriterhofte](https://github.com/henriterhofte) · Twitter [@HeNRGi](https://twitter.com/HeNRGi)
* Nick van Ravenzwaaij · [@n-vr](https://github.com/n-vr)
* Engbert Nijboer
 
Thanks go to those who are the ultimate source of this dataset:
* all anonymous subjects who volunteered to make their measurement data available
