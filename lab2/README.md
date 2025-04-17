# **Lab Guide: Data Transformation & Automation**

Integrating Kafka for real-time network event streaming and transforming data into insightful visualizations and alerts with Grafana and Prometheus. We'll be using Kafka events related to telemetry and alarms from anomaly events coming from advanced analytics as an example.  
At the end of this lab, you should be able to transform data from any sort of source or subscription into useful information to gain better insight into your network state.

---

## Infrastructure Setup

Following are the steps to set up the environment.

Get into the workspace folder for this part of the lab:

```bash
# change into Part 2 directory
cd /workspaces/autocon3-ws-a3/lab2
```

---

## **Kafka-Based Alert Simulation & Export to Prometheus**

This part of the lab focuses on simulating **realistic network alerts** using Kafka producers and exposing them as **Prometheus metrics** using a lightweight Python exporter. The goal is to create a reproducible alert simulation and monitoring environment using Docker.

---

### **1. Build Kafka Producer and Exporter Images**

#### Kafka Alarm Producer
```bash
sudo docker build -t kafka-alarm-producer . -f Dockerfile.kafka-alarm-producer
```

---

### **2. Start the Kafka + Exporter Stack**

Start the infrastructure using Docker Compose:
```bash
sudo docker compose up -d
```

This will launch:
- Zookeeper
- Kafka broker
- Kafka Alarm Producer
- Kafka Alert Exporter (Prometheus endpoint)
- Prometheus
- Alertmanager

---

### **3. Verify Kafka is Running**

You can check Kafka connectivity using:
```bash
python3 kafka-check.py --bootstrap-servers kafka:9092
```

---

## **Simulating Alerts**

### **4. Kafka Alarm Producer**

The `kafka-alarm-producer.py` dynamically generates alert messages using JSON templates and sends them to Kafka topics:
- `rt-anomalies` (baseline deviations)
- `nsp-act-action-event` (threshold violations)

To restart or observe the producer:
```bash
sudo docker kill --signal=HUP kafka-alarm-producer
sudo docker exec -ti kafka-alarm-producer cat kafka-alarm-producer.log
```

---

### **5. Prometheus Exporter**

The `kafka-alert-exporter.py` consumes messages and translates them into Prometheus metrics.

- Accessible at: [http://localhost:8001/metrics](http://localhost:8001/metrics)
- Defined in `alarms.yml`

Sample metric definition:
```yaml
alarms:
  - topic: 'rt-anomalies'
    partition: 0
    counters:
      anomaly_detected:
        name: 'anomaly_detected'
        description: 'Anomaly detected in real-time telemetry'
        labels: ['neId', 'neName', 'counter']
```

To view logs:
```bash
sudo docker exec -ti kafka-alert-exporter tail -f /app/kafka-alert-exporter.log
```

---

## **Monitoring and Validation**

### **6. View Prometheus**

Access Prometheus UI at:  
👉 `http://localhost:9095`

---

### **7. Run Kafka Consumer (Optional Debug Tool)**

```bash
python3 kafka-msgs.py --bootstrap-servers kafka:9092
```

This prints Kafka messages in real-time from the defined topics.

---

## **Manual Message Testing**

For manual testing:
```bash
python3 kafka-test-producer.py --bootstrap-servers kafka:9092 --message "This is a test message"
```

---

## **Customization**

- Modify templates in `kafka-alarm-sim-templates/`
- Adjust timing and categories in `kafka-alarm-producer-config.yml`
- Update alert processing logic or metrics in `alarms.yml`

---

## **Cleaning Up**

Stop and remove the containers:
```bash
sudo docker compose down
```

Or kill individual components:
```bash
sudo docker kill kafka-alarm-producer kafka-alert-exporter prometheus alertmanager
```

---

### **Summary**

- **Kafka producers** dynamically simulate telemetry and alarm events.
- **Kafka exporter** translates alerts into Prometheus metrics.
- **Prometheus & Alertmanager** provide visualization and alerting.
- Easy to adapt for different message formats or new data sources.

