# kafka-jmx-grafana-docker

Docker-compose file for Confluent Kafka with configuration mounted as properties files. Brings up Kafka and components with JMX metrics exposed and visualized using Prometheus and Grafana

## Start

```
docker-compose up -d
```

## Usage

The docker-compose file brings up 3 node kafka cluster with security enabled. Each service in the compose file has its properties/configurations mounted as a volume from a directory with the same name as the service.

Check the kafka server.properties for more details about the Kafka setup.

### Health

Check if all components are up and running using

```bash
docker-compose ps -a
# Ensure there are no Exited services and all containers have the status `Up`
```


### Client

To use a kafka client, exec into the `kfkclient` container which contains the Kafka CLI and other tools necessary for troubleshooting Kafka. THe `kfkclient` container also contains a properties file mounted to `/opt/client`, which can be used to define the client properties for communicating with Kafka.

```
docker exec -it kfkclient bash
```

### Logs

Check the logs of the respective service by its container name.

```bash
docker logs <container_name> # docker logs kafka1
```

### Restarting services

To restart a particular service - 

```bash
docker-compose restart <service_name> # docker-compose restart kafka1
# OR
docker-compose up -d --force-recreate <service_name> # docker-compose up -d --force-recreate kafka1
```

# Scenario 11

> **Before starting ensure that there are no other versions of the sandbox running**
> Run `docker-compose down -v` before starting

1. Start the scenario with `docker-compose up -d`
2. Wait for all services to be up and healthy `docker-compose ps`
3. Wait for the topics to be created. Check the control center(localhost:9021) to see if the topic `inventory_changes` is created

## Problem Statement

The client is unable to produce/consume from the topic `inventory_changes` using the command - 

```
kafka-console-producer --bootstrap-server kafka1:19092 --producer.config /opt/client/client.properties --topic inventory_changes

kafka-console-consumer --bootstrap-server kafka1:19092 --consumer.config /opt/client/client.properties --from-beginning --topic inventory_changes
```

The client is using SASL/PLAIN over PLAINTEXT with the user `bob`


## Solution
After bringing up the container check logs SSL handshake is failing lets check the truststore , keystore . The certificates is expired

Note - Refer scenario6 for the steps of generating certificates , keystore and truststore.

![alt text](<./assets/Screenshot 2024-10-29 at 4.01.27 AM.png>)

Comment or remove user:root , entrypoint and cap_add . The script scripts/kafka1.sh is droping triffic to port 19092 becase of script in entrypoint

```
#!/bin/bash

yum install iptables -y

iptables -A INPUT -p tcp --dport 19092 -j DROP

su - appuser

$@
```
![alt text](<./assets/Screenshot 2024-10-29 at 10.10.56 AM.png>)

Afer removing those run:-

```
docker-compose up -d
```

And start publishing/consuming the data to the topic inventory_changes

![alt text](<Screenshot 2024-10-29 at 10.17.11 AM.png>)