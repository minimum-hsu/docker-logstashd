# docker-logstashd

## Build

```
$sudo docker build --force-rm=true -t logstashd .
```

## Run

### Basic Run

```
$sudo docker run -d -p 5043:5043 --name logstashd logstashd
```

### Advanced Run

#### Prepare a volume

```
$mkdir certs
```

#### Prepare SSL certificate  

##### Existing domain validated SSL certificate

Rename as logstash.key and logstash.crt, and put them into _certs_ directory.

##### Find out IP address and generate self-signed certificate

+ Find out IP address if you do not know it.  
```
$ifconfig eth0 | grep 'inet addr' | cut -d: -f2 | awk '{print $1}'
```
+ Open the OpenSSL configuration file to write  
```
$vim ./certs/openssl.cnf
```
+ Find the _[ v3\_ca ]_ section in this configuration file, and rewrite `subjectAltName = IP: 127.0.0.1` as `subjectAltName = IP: YOUR_IP_ADDRESS` where _YOUR_IP_ADDRESS_ is the IP you found or specified.
+ Generate certificate. (ex. IP is 127.0.0.1)
```
$openssl req -x509 -nodes -days 3650 -newkey rsa:2048 \
-keyout ./certs/logstash.key -out ./certs/logstash.crt \
-subj "/CN=127.0.0.1" \
-extensions v3_req -config /certs/openssl.cnf
```

#### Run the docker image

```
$sudo docker run -d -v ./certs:/certs -p 5043:5043 --name logstashd logstashd
```

## Check

```
$sudo docker exec logstashd service logstash status
```
