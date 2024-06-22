# WordPress on Kubernetes with Monitoring

## Table of Contents
- [Introduction](#introduction)
- [Prerequisites](#prerequisites)
- [Project Structure](#project-structure)
- [Deployment Guide](#deployment-guide)
  - [Start Minikube](#start-minikube)
  - [PersistentVolumeClaim](#persistentvolumeclaim)
  - [Secrets Management](#secrets-management)
  - [MySQL Deployment and Service](#mysql-deployment-and-service)
  - [Nginx Deployment and Service](#nginx-deployment-and-service)
  - [WordPress Deployment and Service](#wordpress-deployment-and-service)
- [Monitoring Setup](#monitoring-setup)
  - [Prometheus Deployment](#prometheus-deployment)
  - [Grafana Deployment](#grafana-deployment)
  - [Service Monitors](#service-monitors)
  - [Dashboards](#dashboards)
- [Cleanup](#cleanup)
- [Output](#Output)
- [Best Practices](#best-practices)
- [Troubleshooting](#troubleshooting)
- [Contributing](#contributing)
- [License](#license)

## Introduction
This repository contains the code and configuration files to deploy a production-grade WordPress application on Kubernetes, along with a monitoring setup using Prometheus and Grafana. The deployment includes MySQL for the database, Nginx as a reverse proxy, and persistent storage for data durability. This setup uses Minikube for local Kubernetes cluster management.

## Prerequisites
- Minikube (v1.18+)
- Helm (v3+)
- kubectl (v1.18+)
- Docker (for building custom images)

## Project Structure
```
my-wordpress-k8s/
├── Chart.yaml
├── values.yaml
├── Dockerfiles/
│   ├── Dockerfile.wordpress
│   ├── Dockerfile.mysql
│   ├── Dockerfile.nginx
│   ├── nginx.conf
├── templates/
│   ├── mysql-deployment.yaml
│   ├── mysql-service.yaml
│   ├── nginx-deployment.yaml
│   ├── nginx-service.yaml
│   ├── wordpress-deployment.yaml
│   ├── wordpress-service.yaml
│   ├── pvc.yaml
│   ├── secrets.yaml
├── monitoring/
│   ├── dashboards/
│   │   ├── wordpress-dashboard.json
│   │   ├── nginx-dashboard.json
│   ├── grafana/
│   │   ├── values.yaml
│   ├── prometheus/
│   │   ├── values.yaml
│   ├── service-monitor/
│   │   ├── wordpress-service-monitor.yaml
│   │   ├── nginx-service-monitor.yaml
├── docker-compose.yaml
└── README.md
```

## Deployment Guide

### Start Minikube
Ensure Minikube is installed and start your Minikube cluster.

```bash
minikube start
```

To interact with the Minikube cluster, ensure you set your Docker environment to use Minikube's Docker daemon.

```bash
eval $(minikube docker-env)
```

### PersistentVolumeClaim
Deploy the PVC for WordPress and MySQL to ensure persistent storage.

```bash
kubectl apply -f templates/pvc.yaml
```

### Secrets Management
Ensure that sensitive information like database passwords are managed using Kubernetes Secrets.

```bash
kubectl apply -f templates/secrets.yaml
```

### MySQL Deployment and Service
Deploy the MySQL instance along with its service.

```bash
kubectl apply -f templates/mysql-deployment.yaml
kubectl apply -f templates/mysql-service.yaml
```

### Nginx Deployment and Service
Deploy Nginx configured with OpenResty and Lua support. Ensure `nginx.conf` is copied into the Docker image.

```bash
kubectl apply -f templates/nginx-deployment.yaml
kubectl apply -f templates/nginx-service.yaml
```

### WordPress Deployment and Service
Deploy WordPress with its respective service.

```bash
kubectl apply -f templates/wordpress-deployment.yaml
kubectl apply -f templates/wordpress-service.yaml
```

### Docker Compose for ChartMuseum
If using ChartMuseum for Helm chart repository, deploy it using Docker Compose.

```bash
docker-compose up -d
```

## Monitoring Setup

### Prometheus Deployment
Deploy Prometheus using the provided Helm chart.

```bash
helm repo add prometheus-community https://prometheus-community.github.io/helm-charts
helm install prometheus prometheus-community/prometheus -f monitoring/prometheus/values.yaml
kubectl apply -f monitoring/prometheus/prometheus.yaml
```

### Grafana Deployment
Deploy Grafana using the provided Helm chart.

```bash
helm repo add grafana https://grafana.github.io/helm-charts
helm install grafana grafana/grafana -f monitoring/grafana/values.yaml
kubectl apply -f monitoring/grafana/grafana.yaml
```

### Service Monitors
Deploy the Service Monitors for WordPress and Nginx to allow Prometheus to scrape their metrics.

```bash
kubectl apply -f monitoring/service-monitor/wordpress-service-monitor.yaml
kubectl apply -f monitoring/service-monitor/nginx-service-monitor.yaml
```

### Dashboards
Import the provided Grafana dashboards for WordPress and Nginx.

1. Open Grafana in your browser.
2. Go to Dashboards > Import.
3. Upload `monitoring/dashboards/wordpress-dashboard.json` and `monitoring/dashboards/nginx-dashboard.json`.

## Cleanup
To delete the deployed resources, use the following commands:

```bash
helm delete prometheus
helm delete grafana
kubectl delete -f templates/pvc.yaml
kubectl delete -f monitoring/prometheus/prometheus.yaml
kubectl delete -f monitoring/grafana/grafana.yaml
kubectl delete -f monitoring/service-monitor/wordpress-service-monitor.yaml
kubectl delete -f monitoring/service-monitor/nginx-service-monitor.yaml
```

To stop and delete the Minikube cluster:

```bash
minikube stop
minikube delete
```

## Output
### Grafana
![Alt-text](./Output/Screenshot%20from%202024-06-21%2021-52-01.png)

### Chartmuseum
![Atl-text](./Output/Screenshot%20from%202024-06-21%2022-50-58.png)



## Best Practices
- Use Kubernetes Secrets to manage sensitive data.
- Regularly backup PersistentVolume data.
- Monitor resource usage and set resource requests and limits.
- Use Ingress for external access to WordPress.

## Troubleshooting
- Ensure all pods are running with `kubectl get pods`.
- Check logs of individual pods with `kubectl logs <pod-name>`.
- Verify services and endpoints with `kubectl get svc`.
- For Minikube-specific issues, use `minikube logs`.

## Contributing
Contributions are welcome! Please submit a pull request or open an issue to discuss changes.
