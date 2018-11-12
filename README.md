# About jtimon
https://github.com/nileshsimaria/jtimon  
https://forums.juniper.net/t5/Automation/OpenConfig-and-gRPC-Junos-Telemetry-Interface/ta-p/316090  


# Install jtimon (docker container) 

## let's build a jtimon Docker image 

```
# git clone https://github.com/nileshsimaria/jtimon.git
# cd jtimon/
# make docker
```
## check the image
```
# docker images jtimon
REPOSITORY          TAG                 IMAGE ID            CREATED             SIZE
jtimon              latest              2e8967d4ea00        2 hours ago         16.4 MB
```
There is no container running
```
# docker ps | grep jtimon
```
# Run jtimon 
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
Alternatively, run this command 
```
docker run -it --rm jtimon --help
```

# create a jtimon configuration file
```
vi vmx1.json
```
```
more vmx1.json
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

# Pass to the container the jtimon configuration file 

lets run jtimon dockerized while passing the local directory to the container to access the configuration file.  


run jtimon with the configuration file ```vmx1.json``` and Print Telemetry data
 
```
./jtimon --config vmx1.json --print
```
Alternatively, run this command 

```
docker run -it --rm -v $PWD:/u jtimon --config vmx2.json --print
```
# Junos device details 
Here's the junos details for the device 172.30.52.155 (```vmx1.json``` jtimon configuration file)

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
```
lab@jedi-vmx-1-vcp> show agent sensors
```
