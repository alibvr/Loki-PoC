# Loki-PoC
## Setting Up a Cluster with Grafana and Grafana Loki

### Prerequisites
- Kubernetes cluster
- Helm installed

### Step 0: Set Up the Cluster and Prerequisites

Before you begin, ensure you have a Kubernetes cluster up and running. You can use a managed Kubernetes service like GKE, EKS, or AKS, or set up a local cluster using Minikube or Kind.

#### Set Up Minikube with Two Worker Nodes
Follow these steps to set up a Minikube cluster with two worker nodes:

1. Start Minikube with two worker nodes:
    ```sh
    minikube start --nodes=3
    ```

2. Verify the nodes:
    ```sh
    kubectl get nodes
    ```

#### Install Helm
If Helm is not already installed, follow these steps to install it:

1. Download the Helm binary:
    ```sh
    curl https://raw.githubusercontent.com/helm/helm/main/scripts/get-helm-3 | bash
    ```

2. Verify the installation:
    ```sh
    helm version
    ```

Once the cluster and Helm are set up, you can proceed with the installation of Grafana and Grafana Loki.

### Step 1: Install Grafana
```sh
helm repo add grafana https://grafana.github.io/helm-charts
helm repo update
helm install grafana grafana/grafana
```

### Step 2: Install Grafana Loki
```sh
helm repo add grafana https://grafana.github.io/helm-charts
helm repo update
helm install loki grafana/loki-stack --set loki.size=medium
```

### Step 3: Configure Retention
Ensure the correct retention is set up according to your cluster's NFRs by modifying the `values.yaml` file:
```yaml
loki:
    config:
        table_manager:
            retention_deletes_enabled: true
            retention_period: 168h # Example: 7 days
```

### Step 4: Configure Multiple Tenants
Modify the `values.yaml` file to configure multiple tenants:
```yaml
loki:
    config:
        auth_enabled: true
        multi_tenant_mode: true
        limits_config:
            enforce_metric_name: false
            reject_old_samples: true
            reject_old_samples_max_age: 168h # Example: 7 days
        schema_config:
            configs:
                - from: 2020-10-24
                    store: boltdb-shipper
                    object_store: s3
                    schema: v11
                    index:
                        prefix: index_
                        period: 24h
        storage_config:
            aws:
                s3: s3://<bucket-name>
        tenants:
            - name: application
            - name: infra
```

### Step 5: Apply the Configuration
```sh
helm upgrade loki grafana/loki-stack -f values.yaml
```

### Conclusion
You have successfully set up a cluster with Grafana and Grafana Loki, configured retention, and set up multiple tenants for dedicated application and infrastructure logging.