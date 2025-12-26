# KAFKA_NOOB

A personal Kafka playground for beginners to learn and practice Kafka infrastructure deployments using **Strimzi** on **Kubernetes (Minikube)**. This repository is intended for experimenting with Kafka, KRaft mode (no Zookeeper), Kafka UI, Kafka Connect, Schema Registry, and observability via Prometheus and Grafana — all on a local machine.

---

## Table of Contents

1. [Why this repo](#why-this-repo)
2. [Prerequisites](#prerequisites)
3. [Minikube Setup](#minikube-setup)
4. [Installing Strimzi Operator](#installing-strimzi-operator)
5. [Kafka Cluster Deployment](#kafka-cluster-deployment)
6. [Kafka Users and Topics](#kafka-users-and-topics)
7. [Kafka UI Deployment](#kafka-ui-deployment)
8. [Schema Registry Deployment](#schema-registry-deployment)
9. [Kafka Connect Deployment](#kafka-connect-deployment)
10. [Prometheus & Grafana (Observability)](#prometheus--grafana-observability)
11. [Deployment Order and Verification](#deployment-order-and-verification)
12. [Notes on Authentication & Security](#notes-on-authentication--security)

---

## Why this repo

`KAFKA_NOOB` is designed as a **safe, personal sandbox** for experimenting with Kafka infrastructure without production risk. Key objectives:

* Deploy Kafka cluster in **KRaft mode** (no Zookeeper).
* Learn Kafka **users, topics, and authentication (SCRAM)**.
* Integrate **Kafka UI, Schema Registry, and Kafka Connect**.
* Test **observability** via Prometheus and Grafana.
* Easy-to-run **Minikube setup** for laptops.

---

## Prerequisites

* Mac / Linux / Windows with **16GB RAM recommended**
* [kubectl](https://kubernetes.io/docs/tasks/tools/)
* [Minikube](https://minikube.sigs.k8s.io/docs/start/)
* [Helm](https://helm.sh/docs/intro/install/)
* Docker (for Minikube `--driver=docker`)

Recommended Minikube resources: 4 CPU, 8GB RAM, 50GB disk.

---

## Minikube Setup

Start Minikube:

```bash
minikube start --memory=8192 --cpus=4 --disk-size=50g --driver=docker
```

Verify nodes:

```bash
kubectl get nodes
```

---

## Installing Strimzi Operator

* Add Helm repo and update:

```bash
helm repo add strimzi https://strimzi.io/charts/
helm repo update
```

* Create namespace and install operator:

```bash
kubectl create namespace kafka-noob
helm install strimzi strimzi/strimzi-kafka-operator -n kafka-noob
```

* Verify operator pods:

```bash
kubectl get pods -n kafka-noob
```

---

## Kafka Cluster Deployment

* Deploy Kafka in **KRaft mode** with single broker/controller pod.
* Strimzi automatically creates the broker pod, combining **controller + broker roles**.
* Verify cluster:

```bash
kubectl get kafka -n kafka-noob
kubectl get strimzi -n kafka-noob
```

---

## Kafka Users and Topics

* **KafkaUser** CRD creates users with SCRAM-SHA-512 authentication and ACLs.
* Topics are created via **KafkaTopic** CRD.
* Verify:

```bash
kubectl get kafkatopic -n kafka-noob
kubectl get kafkauser -n kafka-noob
```

* Check user secret:

```bash
kubectl get secret -n kafka-noob
```

---

## Kafka UI Deployment

* Provides a browser-based interface to inspect clusters, topics, and consumers.
* Access via NodePort using Minikube service command:

```bash
minikube service kafka-ui -n kafka-noob
```

* Kafka UI connects using SCRAM authentication to Kafka cluster.

---

## Schema Registry Deployment

* Stores and manages **Avro schemas**.
* Connects to Kafka cluster via bootstrap server.
* Uses the `_schemas` topic internally.
* Verify:

```bash
kubectl get pods -n kafka-noob
kubectl logs -f <schema-registry-pod> -n kafka-noob
```

---

## Kafka Connect Deployment

* Provides a **framework for integrating Kafka with external systems**.
* Can run **single-worker** mode for local experimentation.
* Connects to Kafka using SCRAM credentials.
* Verify:

```bash
kubectl get pods -n kafka-noob
kubectl logs <connect-pod> -n kafka-noob
```

---

## Prometheus & Grafana (Observability)

⚠️ **Work in Progress** — This section is not yet fully tested and verified.

* Prometheus scrapes Kafka metrics (JMX via Strimzi).
* Grafana visualizes these metrics via dashboards.
* Access Grafana via NodePort or port-forward.
* Prometheus is added as a data source inside Grafana UI.

**Status:** Testing and refinement ongoing. Configuration may need adjustments.

---

## Deployment Order and Verification

1. **Minikube** → verify nodes.
2. **Strimzi Operator** → verify operator pod.
3. **Kafka Cluster (KRaft)** → verify broker/controller pods.
4. **Kafka Users & Topics** → verify secrets, topics.
5. **Kafka UI** → verify UI shows cluster, topics.
6. **Schema Registry** → verify logs and `_schemas` topic.
7. **Kafka Connect** → verify logs and internal topics.
8. **Prometheus + Grafana** → verify metrics endpoints.

---

## Notes on Authentication & Security

* Using **SCRAM-SHA-512** for authentication locally (`SASL_PLAINTEXT`).
* `KafkaUser` automatically generates K8s secrets.
* Kafka UI, Schema Registry, and Kafka Connect reference these secrets for authentication.
* `authorization.type: simple` defines ACLs per topic, group, and cluster.

---

This README provides a **complete beginner-friendly workflow** to deploy and test Kafka and its ecosystem on Minikube using Strimzi, with security and observability practices in place.

---

## Acknowledgments

This project was inspired and built with reference to the excellent examples from the [Strimzi Kafka Operator repository](https://github.com/strimzi/strimzi-kafka-operator/tree/main/examples). A special thanks to the Strimzi community for providing clear, well-documented examples that make learning Kafka on Kubernetes much easier!
