# Usefull Commands

```
sudo docker kill --signal=HUP kafka-alarm-producer
sudo docker exec -ti kafka-alarm-producer cat kafka-alarm-producer.log
sudo docker build -t kafka-alarm-producer . -f Dockerfile.kafka-alarm-producer 
sudo docker compose up -d
```  

```
sudo docker build -t kafka-telemetry-producer . -f Dockerfile.kafka-telemetry-producer 
sudo docker compose up -d
``` 

```
sudo docker kill --signal=HUP kafka-telemetry-producer
sudo docker exec -ti kafka-telemetry-producer cat kafka-telemetry-producer.log
```