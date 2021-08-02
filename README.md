# pepa_ping_mmsys21 


The dump file was generated by PostgreSQL 13.3. First we create the user and the database:
```
$ psql -U postgres -h <ip> -p <port>
CREATE ROLE mmsys21_user WITH LOGIN;
ALTER ROLE mmsys21_user WITH ENCRYPTED PASSWORD 'your_password';
CREATE DATABASE pepa_database;
GRANT ALL ON DATABASE pepa_database TO mmsys21_user;
\q
```
Download the file `pepa_database.sql.zip` and unzip it. To download the file, it is not enough to clone the repository. It must be downloaded directly from github.

Once the file has been downloaded and unzipped, and being in the directory in which the file has been extracted, run the following code to load the database:
```
$ psql pepa_database -U mmsys21_user -h <ip> -p <port> < pepa_database.sql
password: 'your_password'
```
To enter:
```
$ psql pepa_database -U mmsys21_user -h <ip> -p <port>
password: 'your_password'
```

To ensure that the database has been loaded correctly, it is recommended to perform the following queries and obtain the same result.
```
pepa_database=> SELECT COUNT(*) FROM pepa_ping;
 count  
---------
 3199176

pepa_database=> SELECT package_name, COUNT(*) FROM pepa_ping GROUP BY package_name ORDER BY count DESC LIMIT 5;
       package_name       | count  
--------------------------+--------
 com.google.android.ims   | 362790
 com.zhiliaoapp.musically | 223459
 com.instagram.android    | 210705
 com.whatsapp             | 184745
 com.google.android.gms   | 177428

pepa_database=> SELECT timestamp, rsrp, rsrq, cqi, rssnr, cell_type FROM cellular WHERE id=77212 ORDER BY timestamp LIMIT 10;
        timestamp        | rsrp | rsrq | cqi | rssnr | cell_type 
-------------------------+------+------+-----+-------+-----------
 2021-01-13 10:47:04.222 |  -79 |   -7 |  15 |    11 | LTE
 2021-01-13 10:47:04.396 |  -79 |   -7 |  15 |    11 | LTE
 2021-01-13 10:47:05.022 |  -79 |   -7 |  15 |    11 | LTE
 2021-01-13 10:47:07.115 |  -75 |   -6 |   9 |    12 | LTE
 2021-01-13 10:47:08.39  |  -73 |   -8 |  14 |     9 | LTE
 2021-01-13 10:47:09.309 |  -73 |   -8 |  14 |     9 | LTE
 2021-01-13 10:47:09.62  |  -73 |   -8 |  14 |     9 | LTE
 2021-01-13 10:47:09.917 |  -73 |   -8 |  14 |     9 | LTE
 2021-01-13 10:47:10.592 |  -72 |   -7 |  14 |    14 | LTE
 2021-01-13 10:47:10.878 |  -72 |   -7 |  14 |    14 | LTE

 pepa_database=> SELECT * FROM ram WHERE id=160944 ORDER BY timestamp desc limit 10;
   id   |        timestamp        | free_ram | total_ram 
--------+-------------------------+----------+-----------
 160944 | 2021-01-31 23:53:01.409 |   233336 |   3764192
 160944 | 2021-01-31 23:52:56.407 |   239424 |   3764192
 160944 | 2021-01-31 23:52:51.402 |   225268 |   3764192
 160944 | 2021-01-31 23:52:46.398 |   224832 |   3764192
 160944 | 2021-01-31 23:52:41.394 |   218044 |   3764192
 160944 | 2021-01-31 23:52:36.388 |   225256 |   3764192
 160944 | 2021-01-31 23:52:31.383 |   232708 |   3764192
 160944 | 2021-01-31 23:52:26.378 |   236052 |   3764192
 160944 | 2021-01-31 23:52:21.373 |   208872 |   3764192
 160944 | 2021-01-31 23:52:16.368 |   211652 |   3764192

```

# Dataset structure

## Network Flow Information
### pepa_ping

|field | description |  datatype |  example | notes|
|-|-|-|-|-|
|id | connection identifier (primary key) | integer | 25281857 | |
|dst_ip| Destination IP | character varying | 69.171.250.34  | |
|dst_port| port of destination| integer | 443 | |
|protocol| protocol used | character varying | tcp | |
|start_time| connection start time in seconds | double precision | 1610066505.914193 | |
|end_time| connection end time in seconds | double precision | 1610066530.791653 | |
|tx_bytes| bytes transmitted of the connection | double precision |  854| |
|rx_bytes| bytes received of the connection | double precision | 550 | |
|min_rtt| minimum rtt of the connection | double precision | 1866 |only if protocol==tcp |
|rtt| round trip time of the connection | double precision | 6799 |only if protocol==tcp |
|rtt_var| variance of rtt of the connection | double precision | 5216 |only if protocol==tcp |
|lost_packets| lost packets during the connection | double precision | 0 |only if protocol==tcp |
|package_name|contains the name of the package associated with the connection | character varying|com.facebook.services | |
|environmental_data | foreign key to environmental_data table| integer|61339| |

## Environmental Information
### environmental_data

|field | description |  datatype |  example | notes|
|-|-|-|-|-|
|id |environmental data identifier (primary key) | integer | 160944 | |
|timestamp| initial execution time associated with the environment| timestamp without time zone | 2021-01-31 23:52:01.176| |
|uuid| represent an identifier for the user| character varying | 09527143-f7de-4422-8b65-509824de1eb1 | |
|isp| represents the company that provides service | integer | 846845 | anonymized |


### connection

|field | description |  datatype |  example | notes|
|-|-|-|-|-|
|id | foreign key to environmental_data table | integer | 160944 | |
|timestamp| is the time in which the associated information is recorded| timestamp without time zone | 2021-01-31 23:52:01.331| |
|asn| denotes the number of the associated autonomous system| integer | 872080 | anonymized |
|connection_type| that indicates the type of connection (Mobile, Wifi, Unknown) |  character varying | WIFI | |

### cellular
|field | description |  datatype |  example | notes|
|-|-|-|-|-|
|id | foreign key to environmental_data table | integer | 77212 | |
|timestamp|  that corresponds to the moment in which that data was collected| timestamp without time zone | 2021-01-13 10:47:04.222| |
|rssi|Received Signal Strength Indicator | integer | -51 | |
|rsrp| Reference Signals Received Power| integer | -79 |only if cell_type==LTE |
|rscp| Received Signal Code Power | integer | -63|only if cell_type==WCDMA |
|level| abstract level value for the overall signal strength | integer | 3 | corresponding to an integer between 0 and 4 |
|cqi| Channel Quality Indicator | integer | 15 | only if cell_type==LTE|
|rsrq| Reference Signal Received Quality| integer | -7 | only if cell_type==LTE|
|rssnr| Reference Signal Signal To Noise Radio| integer | 11 | only if cell_type==LTE|
|bit_error_rate| bit error rate | integer | 5 | only if cell_type==WCDMA or cell_type==GMS|
|timing_advance|timing advance | integer | 3 | only if cell_type==LTE|
| antenna | antenna identifier | integer | 8084| anonymized, if antenna is unknown the antenna identifier is 0|
| cell_type | information regarding the type of cell in which it is connected | character varying(10) | LTE | |
| network_type | information regarding the type of network in which it is connected | character varying(10)| UMTS| |


   
### wifi

|field | description |  datatype |  example | notes|
|-|-|-|-|-|
|id | foreign key to environmental_data table | integer | 160944 | |
|timestamp| is the time of the data in the row corresponds| timestamp without time zone | 2021-01-31 23:52:31.398 | |
|frequency| WiFi frequency| double precision | 5745 | |
|rssi| Received Signal Strength Indicator of the Wifi | integer | -78 | |

### ram

|field | description |  datatype |  example | notes|
|-|-|-|-|-|
|id | foreign key to environmental_data table | integer | 160944 | |
|timestamp| denotes the time that RAM was measured| timestamp without time zone | 2021-01-31 23:52:01.334| |
|free_ram| denotes the memory ram available in the current timestamp| double precision | 226120 | |
|total_ram| denotes the total memory ram | double precision | 3764192 | |







## External Information

### nmap_service

|field | description |  datatype |  example | notes|
|-|-|-|-|-|
|ip|  IPv4 address | character varying(20) |  157.240.29.34 | |
|port| number of the port | integer | 443 | |
|service| service that is currently in the tuple (IP, port) obtained through nmap|  varying(30)| https| |
|protocol| protocol (udp, tcp) associated to the tuple|smallint | 6| |
|date| date the information was obtained| date | 2021-01-22||

### autonomous_system
|field | description |  datatype |  example | notes|
|-|-|-|-|-|
|ip| IPv4 address | character varying(15) | 157.240.29.34 | |
|as_number|autonomous system number associated to the IP. Foreign key to the table autonomous_system_info| integer| 32934||
date| date the information was obtained|date|2021-10-22|

### autonomous_system_info 
|field | description |  datatype |  example | notes|
|-|-|-|-|-|
|as_number| autonomous system number| integer |12956|| 
|name| name of the autonomous system associated|character varying(100)| AKAMAI||
|country| country of the autonomous system associated|character varying(5)|US||

### server_information
|field | description |  datatype |  example | notes|
|-|-|-|-|-|
|ip| IPv4 address | character varying(15) | 157.240.29.34 | |
|common_name| common_name in TLS/SSL certificate or dns_reverse associated to the IP| character varying(100)| *.facebook.com| |
|organization| organization in TLS/SSL certificate|character varying(100)| Facebook, Inc.| |
|source| corresponds to the form from which the information was extracted dns_reverse or SSL/TLS| character varying (20) | TLS ||
|date| date the information was obtained| date | 2021-01-22||


### domain_tag
|field | description |  datatype |  example | notes|
|-|-|-|-|-|
|common_name| common_name obtained in the server_information table| character varying(100) | *.facebook.com| |
|tag| categorization associated to the common_name| character varying(100) | Social Networking| |
|approved| 1 if the categorization was approved by the community or 0 otherwise| smallint| 1||



## Creative Commons License
[![](http://mirrors.creativecommons.org/presskit/buttons/88x31/svg/by.svg)](http://creativecommons.org/licenses/by/4.0/)

This work is licensed under a [Creative Commons Attribution 4.0 International License](http://creativecommons.org/licenses/by/4.0/).
