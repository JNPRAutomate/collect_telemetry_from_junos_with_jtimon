# About this repository

We will use jtimon to collect openconfig telemetry from junos devices.  
The data collected will be printed.  
The data collected will be also stored in influxdb.  
We will use influxdb cli and a python library to query influxb data.   

# Junos device details 

## Junos version
```
lab@jedi-vmx-1-vcp> show version | match telemetry
JUNOS na telemetry [17.2R1-S2.1-C1]
```
```
lab@jedi-vmx-1-vcp> show version | match openconfig
JUNOS Openconfig [0.0.0.9]
```
## Junos configuration 
```
lab@jedi-vmx-1-vcp> show configuration system services extension-service | display set
set system services extension-service request-response grpc clear-text port 50051
set system services extension-service request-response grpc skip-authentication
set system services extension-service notification allow-clients address 0.0.0.0/0
```
```
lab@jedi-vmx-1-vcp> show configuration system services netconf | display set
set system services netconf ssh
```
## Display information about sensors
to Display information about sensors, run this command: 
```
lab@jedi-vmx-1-vcp> show agent sensors
```

# jtimon

## About jtimon

https://github.com/nileshsimaria/jtimon  
https://forums.juniper.net/t5/Automation/OpenConfig-and-gRPC-Junos-Telemetry-Interface/ta-p/316090  

## Install jtimon (docker container) 

### requirements
Install Docker

### Build a jtimon Docker image 

```
# git clone https://github.com/nileshsimaria/jtimon.git
# cd jtimon/
# make docker
```
### check the image
```
# docker images jtimon
REPOSITORY          TAG                 IMAGE ID            CREATED             SIZE
jtimon              latest              2e8967d4ea00        2 hours ago         16.4 MB
```
### List running containers 
There is no container running
```
# docker ps | grep jtimon
```
## Run jtimon 
```
# ./jtimon --help
Usage of /usr/local/bin/jtimon:
      --alias-file string          File containing aliasing information
      --api                        Receive HTTP commands when running
      --compression string         Enable HTTP/2 compression (gzip, deflate)
      --config strings             Config file name(s)
      --config-file-list string    List of Config files
      --explore-config             Explore full config of JTIMON and exit
      --grpc-headers               Add grpc headers in DB
      --gtrace                     Collect GRPC traces
      --json                       Convert telemetry packet into JSON
      --latency-profile            Profile latencies. Place them in TSDB
      --log-mux-stdout             All logs to stdout
      --max-run int                Max run time in seconds
      --no-per-packet-goroutines   Spawn per packet go routines
      --pprof                      Profile JTIMON
      --pprof-port int32           Profile port (default 6060)
      --prefix-check               Report missing __prefix__ in telemetry packet
      --print                      Print Telemetry data
      --prometheus                 Stats for prometheus monitoring system
      --prometheus-port int32      Prometheus port (default 8090)
      --stats-handler              Use GRPC statshandler
      --version                    Print version and build-time of the binary and exit

```
Alternatively, run this command. These 2 commands are equivalents. 
```
# docker run -it --rm jtimon --help
```

## create a jtimon configuration file
```
# vi vmx1.json
```
```
# more vmx1.json
{
    "host": "172.30.52.155",
    "port": 50051,
    "user": "lab",
    "password": "m0naco",
    "cid": "my-client-id",
    "grpc" : {
        "ws" : 524288
    },
    "paths": [{
        "path": "/interfaces",
        "freq": 2000
    }, {
        "path": "/junos/system/linecard/cpu/memory/",
        "freq": 1000
    }, {
        "path": "/bgp",
        "freq": 10000
    }, {
        "path": "/components",
        "freq": 10000
    }]
}
```

## Pass the jtimon configuration file to the container

lets run jtimon dockerized while passing the local directory to the container to access the configuration file.  

run jtimon with the configuration file ```vmx1.json``` and Print Telemetry data
 
```
# ./jtimon --config vmx1.json --print
```
Alternatively, run this command. These 2 commands are equivalents. 

```
# docker run -it --rm -v $PWD:/u jtimon --config vmx1.json --print
```
# influxdb 

InfluxDB is an open source time series database written in GO. 

## install influxbd 

download influxdb
```
# wget https://dl.influxdata.com/influxdb/releases/influxdb_1.7.0_amd64.deb
```
Install influxdb
```
# sudo dpkg -i influxdb_1.7.0_amd64.deb
```
start influxdb
```
# service influxdb start
```
Verify
```
# service influxdb status
```


## jtimon and influxbd
create a jti configuration file that uses influxdb
```
# more vmx2.json
{
    "host": "172.30.52.156",
    "port": 50051,
    "user": "lab",
    "password": "m0naco",
    "cid": "my-client-id2",
    "grpc" : {
        "ws" : 524288
    },
    "paths": [{
        "path": "/interfaces",
        "freq": 2000
    }, {
        "path": "/junos/system/linecard/cpu/memory/",
        "freq": 1000
    }, {
        "path": "/bgp",
        "freq": 10000
    }, {
        "path": "/components",
        "freq": 10000
    }],
    "influx": {
        "server": "172.30.52.37",
        "port": 8086,
        "dbname": "db",
        "user": "influx",
        "password": "influxdb",
        "recreate": true,
        "measurement": "m"
    }
}

```

start an influxdb cli session and create a user
```
# influx
Connected to http://localhost:8086 version 1.7.0
InfluxDB shell version: 1.7.0
Enter an InfluxQL query
>  CREATE USER "influx" WITH PASSWORD 'influxdb'
> show users
user   admin
----   -----
influx false
> exit
# 
```
run jtimon with this configuration file
```
# ./jtimon --config vmx2.json --print
```

## query data from Influxdb using CLI 
start an influxdb cli session
```
# influx
Connected to http://localhost:8086 version 1.7.0
InfluxDB shell version: 1.7.0
Enter an InfluxQL query
```
To list the databases, run this command.
```
> show databases
name: databases
name
----
_internal
db
```
To use the database ```db```, run this command

```
> use db
Using database db
```
Run this command to list measurements
```
> show measurements
name: measurements
name
----
m
```
```
> select * from m order by desc limit 10
```
exit influxdb cli
```
> exit
# 
```

## query data from Influxdb using python

install the influxdb python library.
This python library is a client for interacting with InfluxDB.
```
# pip install influxdb
```
Verify
```
# pip list | grep influx
```

run this command to start a python interactive session  
```
# python
```
connect to InfluxDB using Python.
```
>>> from influxdb import InfluxDBClient
>>> influx_client = InfluxDBClient('localhost',8086)
```
list the databases
```
>>> influx_client.query('show databases')
ResultSet({'(u'databases', None)': [{u'name': u'_internal'}, {u'name': u'db'}]})
>>> gp=influx_client.query('show databases').get_points()
>>> for item in gp:
...    print item
...
{u'name': u'_internal'}
{u'name': u'db'}
```
list measurements for a database
```
>>> influx_client.query('show measurements', database='db')
ResultSet({'(u'measurements', None)': [{u'name': u'm'}]})
```
query data from a particular measurement and database
```
>>> gp = influx_client.query('select * from "m"  order by desc limit 5 ', database='db').get_points()
>>> for item in gp:
...    print item
...
```
