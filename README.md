# Evaluation Toolset

## Project Description
Toolset prepared for the `EU` `INEGMA` project. Its purpose is preparing set of tools meant to help the process of
evaluation of disaster response and firefighting exercises. This particular toolset helps in gathering evaluation data
during an ongoing exercises, filtering and presenting them (some in real time).

#### Toolset consist of:
 - Google Docs
 - Google Sheets
 - Google Forms
 - Google Slides 
 - Observation support tool (OST) (https://github.com/DRIVER-EU/ost)
 - Kafka (https://kafka.apache.org/)
 - Elasticsearch
 - Kibana

This exact project contains our personal use case of this toolset showing how we expect this toolset to be used 
(`Demonstrator`, as we call it).

![Demonstator data flow diagram.png](images%2FDemonstator%20data%20flow%20diagram.png)

Above image presents data flow in our demonstrator. First of all, data comes from OST and other sources. Another source
can be any application which can send any data relevant for the evaluation to kafka (for example an application which 
periodically sends status of some machines used in the exercise). All data about exercise are gathered in Kafka and sent
to Elasticsearch as they come in. Then they can be viewed, filtered, sorted and made into statistics in Kibana which
lets user access data from Elasticsearch trough graphical interface.

Demonstrator has also additional application which serves to simulate other sources of data (other than OST). It 
basically takes scenario prepared in xsl file format and sends data from this scenerio to kafka at time configured 
in the given scenario.

## Getting Started
### Requirements
- ubuntu server 20.04 LTS or 22.04 LTS or newer
- docker 23.x.x+
- docker compose plugin v.2.2.3+
### Prerequisites
Before starting demonstrator, enter root project directory in terminal and type:
```shell
mkdir certificates && cd certificates && openssl req -newkey rsa:2048 -nodes -keyout key.pem -x509 -days 365 -out certificate.pem -subj "/C=PL/ST=itti/L=itti/O=itti/OU=itti/CN=itti"
```
To create new self-signed certificate to be used by OST. To use different certificate replace two generated files with
files named the same.

Change default passwords for ELK stack in .env file.

### Running Demonstrator
To run demonstrator enter projects root directory in terminal and type:
```shell
docker compose up -d
```
This command may need root privileges.
After everything sets up, at those addresses corresponding applications should be available:
- http://localhost:5601 - kibana
- http://localhost:8187 - scenario simulator
- https://localhost - OST

OST's administrator credentials by default are:
- login: admin
- password: admin
It is recommended to change password after first deployment through the OST application.

### Configuration
~~Default configuration of kafka's elasticsearch sink connectors can be found in connect/config/. To change it change or
add commands in connect/config/add_connector_config.sh and rebuild the docker image with docker file in connect/.
Then substitute the image in projects docker-compose.yml. Information about how to configure kafka connect with 
elasticsearch sink connector can be found under https://www.confluent.io/blog/kafka-elasticsearch-connector-tutorial/.
There is also screen cap of the site in sites/ directory.~~

~~For the elasticsearch to show the dates as dates and not long numbers index templates need to be made. For every kafka
topic with a date there is a need of index template to change the mapping of the date field to "Date nanoseconds"
format. To add a new index template open kibana in your browser, go to Management -> Index Management -> Index Templates
and create new template. Make index pattern same as the topic in kafka and then in mappings add new field mapping with 
name same as the field with date from kafka and with "Date nanoseconds" field type.~~

~~For example for the OST answers to show date right template needs to look like this:
![index template1.png](images%2Findex%20template1.png)
![index template2.png](images%2Findex%20template2.png)~~

Demonstrator now uses Logstash to listen to kafka on configured topics. It also changes fields with dates in
milliseconds from long format to elasticsearch date format before it pushes them to elasticsearch, so they don't need to
be changed trough index templates anymore. If there are going to be new topics in kafka which require different filters
in Logstash, they can be added in different pipeline. They need their own .conf file in logstash/pipeline/ which needs
to be added in pipelines.yml configuration file like in documentation:
https://www.elastic.co/guide/en/logstash/current/multiple-pipelines.html.

### Administration
To stop demonstrator enter projects root directory in terminal and type:
```shell
docker compose down
```

To check status of each application in demonstrator enter projects root directory in terminal and type:
```shell
docker compose ps
```

To check logs of specific application in demonstrator enter projects root directory in terminal and type:
```shell
docker compose logs [service name]
```
where [service name] is the name of the service in docker-compose.yml which corresponds to the specific application.

The ELK (elasticsearch, Logstash, Kibana) docker configuration was adopted from: 
https://github.com/deviantony/docker-elk which contains a lot of useful documentation.
