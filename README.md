# docker-logstashd

This docker image is assumed that the Elasticseach server is _localhost:9200_.

## Build

Enter the following command in the repo directory.
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
$mkdir ~/certs
```

#### Prepare SSL certificate  

##### Existing domain validated SSL certificate

Rename as logstash.key and logstash.crt, and put them into _~/certs_ directory.

##### Find out IP address and generate self-signed certificate

+ Find out IP address if you do not know it.  
```
$ifconfig eth0 | grep 'inet addr' | cut -d: -f2 | awk '{print $1}'
```
+ Copy the OpenSSL configuration file.
```
$cp ./certs/openssl.cnf ~/certs/openssl.cnf
```
+ Open the OpenSSL configuration file to write  
```
$vim ~/certs/openssl.cnf
```
+ Find the _[ v3\_ca ]_ section in this configuration file, and rewrite `subjectAltName = IP: 127.0.0.1` as `subjectAltName = IP: YOUR_IP_ADDRESS` where _YOUR\_IP\_ADDRESS_ is the IP you found or specified.
+ Generate certificate. (ex. IP is 127.0.0.1)
```
$openssl req -x509 -nodes -days 3650 -newkey rsa:2048 \
-keyout ~/certs/logstash.key -out ~/certs/logstash.crt \
-subj "/CN=127.0.0.1" \
-extensions v3_req -config ~/certs/openssl.cnf
```

#### Run the docker image

```
$sudo docker run -d -v ~/certs:/certs -p 5043:5043 --name logstashd logstashd
```

## Check

```
$sudo docker exec logstashd service logstash status
```

## Note

### Specify Elasticsearch server

+ Open _./conf.d/lumberjack.conf_ in the repo directory to write  
```
$vim ./conf.d/lumberjack.conf
```
+ Find the _elasticsearch_ block in this configuration file, and rewrite `host => "localhost"` as `host => "YOUR_SERVER"` where _YOUR\_SERVER_ is the domain or IP you specified. [Please refer to the official documents for other configurations.](https://www.elastic.co/guide/en/logstash/current/plugins-inputs-elasticsearch.html)
+ Build the image.

### Use [minimum-hsu/docker-elasticsearchd](https://github.com/minimum-hsu/docker-elasticsearchd)

+ Open _./conf.d/lumberjack.conf_ in the repo directory to write  
```
$vim ./conf.d/lumberjack.conf
```
+ Find the _elasticsearch_ block in this configuration file, and rewrite `host => "localhost"` as `host => "es"`.
+ Build the image.
+ Add option `--link elasticsearchd:es` in run command, such as  
```
$sudo docker run -d -v ~/certs:/certs -p 5043:5043 --link elasticsearchd:es --name logstashd logstashd
```
