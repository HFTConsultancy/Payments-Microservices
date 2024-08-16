# Kafka Setup and Microservices Deployment Guide

This guide walks you through setting up Kafka with Helm, deploying several microservices, and using Kafka UI for monitoring.

## Prerequisites

- Kubernetes cluster
- Helm installed
- Kubectl installed

## Step 1: Install Kafka

Add the Bitnami Helm repository and install Kafka with Zookeeper.

```shell
helm repo add bitnami https://charts.bitnami.com/bitnami

helm install kafka-local bitnami/kafka \
--set persistence.enabled=false,zookeeper.persistence.enabled=false
```

## Step 2: Create Kafka Client Pod

Run a Kafka client pod to interact with your Kafka instance.

```shell
kubectl run kafka-local-client \
    --restart='Never' \
    --image docker.io/bitnami/kafka:3.3.1-debian-11-r19 \
    --namespace orders-microservice \
    --command \
    -- sleep infinity
```

## Step 3: Access Kafka Client

Access the Kafka client pod to execute Kafka commands.

```shell
kubectl exec --tty -i kafka-local-client --namespace orders-microservice -- bash
```

## Step 4: Deploy Microservices

Deploy the following microservices in your Kubernetes cluster.

### Order Service

```shell
kubectl run order-svc --rm --tty -i \
    --image worldbosskafka/orders:v1.0.0 \
    --restart Never \
    --namespace orders-microservice \
    --command \
    -- python3 -u ./order_svc.py
```

### Transaction Service

```shell
kubectl run transaction-svc --rm --tty -i \
    --image worldbosskafka/transactions:v1.0.0 \
    --restart Never \
    --namespace orders-microservice \
    --command \
    -- python3 -u ./transaction.py
```

### Notification Service

```shell
kubectl run notification-svc --rm --tty -i \
    --image worldbosskafka/notification:v1.0.0 \
    --restart Never \
    --namespace orders-microservice \
    --command \
    -- python3 -u ./notification.py
```

### Analytics Service

```shell
kubectl run analytics-svc --rm --tty -i \
    --image worldbosskafka/analytics:v1.0.0 \
    --restart Never \
    --namespace orders-microservice \
    --command \
    -- python3 -u ./analytics.py
```

## Step 5: Install Kafka UI

Add the Kafka UI Helm repository and install the Kafka UI for monitoring your Kafka clusters.

```shell
helm repo add kafka-ui https://provectus.github.io/kafka-ui

helm install kafka-ui kafka-ui/kafka-ui \
--set envs.config.KAFKA_CLUSTERS_0_NAME=kafka-local \
--set envs.config.KAFKA_CLUSTERS_0_BOOTSTRAPSERVERS=kafka-local.orders-microservice.svc.cluster.local:9092
```

## Step 6: Access Kafka UI

Port forward the Kafka UI service to your local machine to access it in your web browser.

```shell
kubectl port-forward svc/kafka-ui 8080:8080
```

Now you can access the Kafka UI by navigating to `http://localhost:8080` in your web browser.

---

This guide sets up Kafka with a few basic microservices and a web-based UI for monitoring. Adjust the configurations as needed for your environment.
