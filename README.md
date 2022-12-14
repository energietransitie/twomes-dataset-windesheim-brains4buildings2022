# Brains4Buildings Windesheim dataset 
A dataset with CO₂ concentration, ventilation rate and and occupancy data of 6 office spaces at Windesheim University of Applied Sciences collected for the Brains4Buildings project for a few feews during the autumn of 2022.

**Note**: [Git LFS](https://git-lfs.github.com/) is required to clone big CSV files

## Table of contents
* [General info](#general-info)
* [Subject Recruitment](#recruitment)
* [Inclusion criteria](#inclusion-criteria)
* [Data](#data) 
* [Status](#status)
* [License](#license)
* [Credits](#credits)

## General info

For the [Brains4Buildings project](https://brains4buildings.org/), we collected data about CO₂ concentration, ventilation rate and and occupancy data. This data was collected to answer two research quenstions:

A. To what extent can we reliably derive ventilation rates in a room from:
 * Measured time series data about the occupancy in that room, and
 * Measured data about the CO₂ concentration in a room?

B.	To what extent can we reliably derive occupancy in a room based on:
 * Measured data about the ventilation rate in a room, and 
 * Measured data about the CO₂ concentration in a room?

## Recruitment 

Subjects were recruited via a recruitment e-mail targeted at people known to work in office rooms that satisfied the [inclusion criteria](#inclusion-criteria) for office rooms. Subjects could volunteer to participate and give informed consent by filling out an [online recruitment survey]() ([this survey is also available in XML-format]()).

## Inclusion criteria

Inclusion criteria for office rooms were:
* office rooms at Windesheim were only eligible if we could obtain PIR-based occupancy data and CO₂ concentration data via the existing Building Management System (BMS);
* 75% or more of the occupants of an office room must consent to their presence being tracked.  

Inclusion criteria for subjects were:
* subjects must work at Windesheim University of Applied Sciences in one of the eligible office rooms, typically for more than one hour per week;
* subjects must have a smartphone;
* subjects must give the static Bluetooth MAC address of their smartphone to the researchers for the purpose of the research;
* subjects must be willing to turn / leave on their Bluetooth on their smartphone when they were at Windesheim.


We only installed a [Twomes M5Stack CoreInk + SCD41](https://github.com/energietransitie/twomes-scd41-presence-firmware) measurement device in an office room if more than 75% of the known number of regular office room occupants provided informed consent. We never tracked presence via Bluetooth without informed consent; the measurement devices we use are technically incapable of tracking Bluetoth based presence without a static Bluetooth MAC-addresss.

## Data management

TO DO: Replace this text with (links to) data management plan, privacy policy and (if applicable), DPIA.

## Data

In the sections below, the data pre-processing and data formats used in the data files will be described.

### Measurement devices and other data sources 

We used the following measurement device types and data sources to collect data.

| Source/Device type name | Description                                | Open source repository of devcie                             |
| ----------------------- | ------------------------------------------ | ------------------------------------------------------------ |
| `CO2-meter-SCD4x`       | M5Stack CoreInk + SCD41 measurement device | [twomes-scd41-presence-firmware](https://github.com/energietransitie/twomes-scd41-presence-firmware) |
| `bms`                   | building management system                 |                                                              |
| `xovis`                 | Xovis PC2SE 3Dsensor                       |                                                              |
| `human_observer`        | data collected by human observers          |                                                              |

### Date and time information

All timestamps collected by the `CO2-meter-SCD4x`  devices were measured in [Unix time](https://en.wikipedia.org/wiki/Unix_time) format, using device clocks regularly synchronized via NTP with the correct UTC time. Setting the local device clock to the proper UTC time via NTP was one of the first steps performed by the measurement devices after they were connected to the internet via the home Wi-Fi network of a subject. Each measurement device synchronized its device clock via NTP every 6 hours. Uploads of measurement data (which could contain more than one measurement) were timestamped both by the measurement device according to the local device clock and by the server. We did not yet check for deviations between the last device timestamp of a measurement upload and the upload timestamp at the server.

Timestamps of `bms` and `xovis` sources were assumed to be in the [Europe/Amsterdam](https://en.wikipedia.org/wiki/Time_in_the_Netherlands) timezone. 

TO DO: describe how we verified this.

Timestamps collected by human observers were also registered in the the [Europe/Amsterdam](https://en.wikipedia.org/wiki/Time_in_the_Netherlands) timezone.

Timestamps were converted to a timezone-aware `pandas.Timestamp` value, in the [Europe/Amsterdam](https://en.wikipedia.org/wiki/Time_in_the_Netherlands) timezone. In the csv files we use [ISO 8601 format with time offset](https://en.wikipedia.org/wiki/ISO_8601): `YYYY-MM-DDThh:mm:ss±hhmm`.


### Raw measurements 
 Raw masurements will be available in the folder [/raw-measurements/](/raw-measurements/) in three formats:

 - [twomes_raw_measurements.parquet](/raw-measurements/twomes_raw_measurements.parquet): a single [parquet](https://parquet.apache.org/) file with data for all subject ids;
 - nnnnnn_raw_measurements.parquet: [parquet](https://parquet.apache.org/) files, one for each subject id;
 - nnnnnn_raw_measurements.zip: [zip](https://en.wikipedia.org/wiki/ZIP_(file_format))ped [csv](https://en.wikipedia.org/wiki/Comma-separated_values) files, one for each subject id;

All measurement data is structured according to the table below. By importing the parquet variant using [pandas.read_parquet()](https://pandas.pydata.org/pandas-docs/stable/reference/api/pandas.read_parquet.html), you automatically get a DataFrame wih the recommended indices and data types. 

Alternatively, you can also read the zipped csv files, but this typically takes much longer. You can use the code below to endup with a DataFrame with the recommended indices and data types:

```
TODO: insert pandas.read_csv() code here
```

| **Index/Column** | **Name** | **Type**     | **Description**                                                                                                                                       |
| ---------- | --------------- | ------------ | ----------------------------------------------------------------------------------------------------------------------------------------------------- |
| index       | `id`            | `category`   | unique code of the home                                                                                                                               |
| index       | `device_name`   | `category`   | unique name of the measurement device                                                                                                                 |
| index       | `source`        | `category`   | [device type name](###measurement-devices) of the measurement device                                                                                  |
| index        | `timestamp`     | `Timestamp` | start of the interval (timezone aware) |
| index       | `property`      | `category`   | property name of the measurement                                                                                                                      |
| column      | `value`         | `object`     | value of the measurement                                                                                                                              |
| column      | `unit`          | `category`   | unit of the measurement value                                                                                                                         |

### Raw propertes 
 In the folder [/raw-properties/](/raw-properties/) we will make various measured properties available in an 'unstacked' format with each property in its own column and an appropriate datatype. Similar to measurements, we will make data available in three formats:

 - [twomes_raw_properties.parquet](/raw-properties/twomes_raw_measurements.parquet): a single [parquet](https://parquet.apache.org/) file with data for all subject ids;
 - nnnnnn_raw_properties.parquet: [parquet](https://parquet.apache.org/) files, one for each subject id;
 - nnnnnn_raw_properties.zip: 23 [zip](https://en.wikipedia.org/wiki/ZIP_(file_format))ped [csv](https://en.wikipedia.org/wiki/Comma-separated_values) files, one for each subject id;

All property data is structured according to the table below. By importing the parquet variant using [pandas.read_parquet()](https://pandas.pydata.org/pandas-docs/stable/reference/api/pandas.read_parquet.html), you automatically get a DataFrame wih the recommended indices and data types. 

Alternatively, you can also read the zipped csv files, but this typically takes much longer. You can use the code below to endup with a DataFrame with the recommended indices and data types:

```
TODO: insert pandas.read_csv() code here
```

| **Index/Column** | **Name**                             | **Type**    | **Description**                                              |
| ---------------- | ------------------------------------ | ----------- | ------------------------------------------------------------ |
| index            | `id`                                 | `category`  | unique code of the home                                      |
| index            | `source`                             | `category`  | [device type name](###measurement-devices) of the measurement device |
| index            | `timestamp`                          | `Timestamp` | start of the interval (timezone aware)                       |
| column           | property_1; see property table below | data_type_1 | measured value of this property                              |
| column           | property2                            | data_type_2 | measured value of this property                              |
| ...              | ...                                  | ...         | ...                                                          |
| column           | property_n                           | data_type_n | measured value of this property                              |

### Measured Properties 

Below is a table that lists all properties that were measured, the data type in the [raw-properties](#raw-poperties) DataFrame, the measurement unit, the measurement interval, the source device and sensor that measured it, as well as the the property name and value format as retrieved from the Twomes database.

| Property          | Type      | Unit | Measurement interval \[h:mm:ss\] | Description                                                | Source            | Sensor                                                       | Database property  | [Database format](https://en.wikipedia.org/wiki/Printf_format_string) |
| ----------------- | --------- | ---- | -------------------------------- | ---------------------------------------------------------- | ----------------- | ------------------------------------------------------------ | ------------------ | ------------------------------------------------------------ |
| `co2__ppm`        | `float32` | ppm  | 0:10:00                          | CO₂ concentration                                          | `CO2-meter-SCD4x` | [SCD41](https://sensirion.com/products/catalog/SCD41/)       | `CO2concentration` | %u                                                           |
| `temp_in__degC`   | `float32` | ˚C   | 0:10:00                          | air temperature                                            | `CO2-meter-SCD4x` | [SCD41](https://sensirion.com/products/catalog/SCD41/)       | `roomTemp`         | %.1f                                                         |
| `rel_humidity__0` | `float32` | -    | 0:10:00                          | relative humidity                                          | `CO2-meter-SCD4x` | [SCD41](https://sensirion.com/products/catalog/SCD41/)       | `relativeHumidity` | %.1f                                                         |
| `occupancy__p`    | `Int8`    | -    | 0:10:00                          | number of smartphones responding to Bluetooth name request | `CO2-meter-SCD4x` | ESP32                                                        | `countPresence`    | %u                                                           |
| `heartbeat`       | `Int16`   | -    | 0:10:00                          | measurement system heartbeat                               | `CO2-meter-SCD4x` | ESP32                                                        | `heartbeat`        | %u                                                           |
| `co2__ppm`        | `float32` | ppm  | 0:01:00/1:00:00                  | CO₂ concentration                                          | `bms`             |                                                              |                    |                                                              |
| `temp_in__degC`   | `float32` | ˚C   | 0:01:00/1:00:00                  | air temperature                                            | `bms`             |                                                              |                    |                                                              |
| `rel_humidity__0` | `float32` | -    | 0:01:00/1:00:00                  | relative humidity                                          | `bms`             |                                                              |                    |                                                              |
| `occupancy__bool` | `Int8`    | 0/1  | 0:01:00/1:00:00                  |                                                            | `bms`             | PIR                                                          |                    |                                                              |
| `occupancy__p`    | `Int8`    | 0    | 0:05:00/0:15:00                  | nuber of persons in room                                   | `xovis`           | [PC2SE](https://www.xovis.com/technology/sensor/pc2se-sensor) |                    |                                                              |                                                   |

### Preprocessed data 

TO DO: change preprocessing description below.

Preprocessing of measurements from the measurement database was done using [get_preprocessed_homes_data()](https://github.com/energietransitie/twomes-twutility-inverse-grey-box-analysis/blob/main/data/extractor.py). Preprocessing steps include:
- removal of duplicate measurements;
- calculation of derived properties as a combination of other properties, as indicated in the column `Calculation` in the table below;
- removal of absolute outliers, i.e measurement values smaller than the value in the column `Min` or larger than the value in the column `Max` in the table below;
- removal of statistic outliers, i.e. measuremnt values with an absolute [z-score](https://en.wikipedia.org/wiki/Standard_score) higer than the value indicated in the `Sigma` column in he table below;
- interpolation of measurements to intervals of 15 minutes (no interpolation between measurements that were 60 minutes apart or more);
- All column values represent the average during the interval that starts at the timestamp indicated. 

TO DO: Change the markdown table below.

| **Index/  Column** | **Name**      | **Type**    | **Unit**        | **Description**                                     | **Calculation** |    Min |    Max | Sigma |
| ------------------ | ------------- | ----------- | --------------- | --------------------------------------------------- | --------------- | -----: | -----: | ----: |
| index              | `id`          | `Int16`     |                 | unique code of the room                             |                 | 000000 | 999999 |       |
| index              | `timestamp`   | `Timestamp` |                 | start of the interpolated interval (timezone aware) |                 |        |        |       |


## Status
Dataset is: _collected_, _anonimization-in-progress_

## License
This data is made available under the [CC BY 4.0](./LICENSE.md) by the [Research group Energy Transition, Windesheim University of Applied Sciences](https://windesheim.nl/energietransitie) 

## Credits

Data collection was a joint effort of:
* Henri ter Hofte · [@henriterhofte](https://github.com/henriterhofte) · Twitter [@HeNRGi](https://twitter.com/HeNRGi)
* Nick van Ravenzwaaij · [@n-vr](https://github.com/n-vr)
* Engbert Nijboer
 
Thanks go to those who are the ultimate source of this dataset:
* all anonymous subjects who volunteered to make their measurement data available
