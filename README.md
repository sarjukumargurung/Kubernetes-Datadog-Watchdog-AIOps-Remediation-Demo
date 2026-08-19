## Kubernetes-Datadog-Watchdog-AIOps-Remediation-Demo

## Demo - Datadog Watchdog AIOps Remediation Demo

Datadog Watchdog automatically detects anomalies, performs root-cause analysis, and triggers automated remediation workflows to restore MySQL health inside a Kubernetes-in-Docker (KinD) local cluster on macOS. 

When MySQL experiences performance degradation (such as high connection spikes or locks), Watchdog isolates the infrastructure layer from the application layer to find the root fault instantly.

## Local Architecture & Demo Prerequisites

To replicate this Datadog AIOps environment locally on a Mac, you need a local Kubernetes cluster capable of running containerized databases alongside the monitoring agent.

### Local Cluster Layer: Run KinD (Kubernetes in Docker) 

### Database Workload: 
Deploy a standard MySQL Kubernetes Deployment or Helm chart.

### Observability Agent: 
Install the Datadog Agent Helm Chart with ```datadog.apm.enabled and datadog.logs.enabled``` turned on

### Remediation Engine: 
Connect Datadog Webhooks to local automation tools like Kubernetes operators.

```
[ MySQL Pod (KinD) ] ──(Metrics/Logs)──> [ Datadog Agent ] ──> [ Datadog Cloud (Watchdog AI) ]
                                                                             │
[ Auto-Healed Pod ] <──(K8s Scale/Restart)── [ Webhook Workflow ] <──────────┘ (Anomaly Detected)
```

## 1. Step-by-Step Local Deployment Guide

```bash
# Install kind using Homebrew if needed
brew install kind

# Create a cluster with port mappings
kind create cluster --name dd-mysql-demo
```

## 2. Deploy MySQL with Monitoring Labels
Apply a basic manifest to run MySQL. Ensure you add the Datadog Autodiscovery annotations so the agent immediately picks up the MySQL database metrics:

```
apiVersion: apps/v1
kind: Deployment
metadata:
  name: mysql-demo
spec:
  replicas: 1
  selector:
    matchLabels:
      app: mysql
  template:
    metadata:
      labels:
        app: mysql
      annotations:
        # Datadog Autodiscovery configuration for MySQL
        ://datadoghq.com: '["mysql"]'
        ://datadoghq.com: '[{}]'
        ://datadoghq.com: '[{"server": "localhost", "user": "datadog", "pass": "your_secure_password"}]'
    spec:
      containers:
      - name: mysql
        image: mysql:8.0
        env:
        - name: MYSQL_ROOT_PASSWORD
          value: "xxx"
        ports:
        - containerPort: 3306
```
## 3. Install Datadog Agent via Helm
```
helm repo add datadog https://datadoghq.com
helm repo update

helm install datadog-agent datadog/datadog \
  --set datadog.apiKey="YOUR_DATADOG_API_KEY" \
  --set datadog.appKey="YOUR_DATADOG_APP_KEY" \
  --set datadog.clusterName="kind-dd-mysql-demo"

```

## 4. Configure the Automated Action
1) Navigate to your Datadog Dashboards / Monitors settings in the cloud console.

2) Under Watchdog, locate your anomalous MySQL metrics.

3) Use Datadog Workflow Automation to build a visual blueprint.

4) Add a standard HTTP block or a specialized Kubernetes step to hit your local proxy endpoint, instructing the cluster to execute a remediation action (e.g., kubectl rollout restart deployment/mysql-demo).

