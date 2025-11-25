
# kube-prometheus-stack on k3s (Ubuntu) via Helm Repo + values.yaml

This document describes how to deploy **kube-prometheus-stack** on a k3s Kubernetes cluster (Ubuntu server), using the **Prometheus Community Helm repo** and a custom `values.yaml`.  
It also configures:

- Persistent storage for **Prometheus** and **Grafana**
- Access to **Grafana** via Traefik Ingress at `http://grafana.local`

---

## 1️⃣ Create `monitoring` Namespace

```bash
kubectl create namespace monitoring
```

If the namespace already exists you’ll see `AlreadyExists` – that’s fine.

---

## 2️⃣ Add Prometheus Community Helm Repo

```bash
helm repo add prometheus-community https://prometheus-community.github.io/helm-charts
helm repo update
```

Verify kube-prometheus-stack is available:

```bash
helm search repo prometheus-community/kube-prometheus-stack
```

You should see a result with the current chart version.

---

## 3️⃣ Create `values.yaml` for kube-prometheus-stack

Create a file named **`values.yaml`** (for example under `k8s/kube-prometheus-stack/values.yaml` in your lab repo):

```bash
vim values.yaml
```

Recommended content for a small k3s homelab:

```yaml
# values.yaml for kube-prometheus-stack on k3s

# Optional: give a simpler name to resources
fullnameOverride: kube-prometheus-stack

# -------------------------------
# Prometheus configuration
# -------------------------------
prometheus:
  prometheusSpec:
    retention: 15d
    storageSpec:
      volumeClaimTemplate:
        spec:
          storageClassName: local-path   # k3s default StorageClass
          accessModes: ["ReadWriteOnce"]
          resources:
            requests:
              storage: 10Gi

# -------------------------------
# Alertmanager configuration
# -------------------------------
alertmanager:
  alertmanagerSpec:
    storage:
      volumeClaimTemplate:
        spec:
          storageClassName: local-path
          accessModes: ["ReadWriteOnce"]
          resources:
            requests:
              storage: 5Gi

# -------------------------------
# Grafana configuration
# -------------------------------
grafana:
  adminUser: admin
  adminPassword: "changeme"   # change this in real setups

  service:
    type: ClusterIP
    port: 80

  ingress:
    enabled: true
    ingressClassName: "traefik"
    hosts:
      - grafana.local
    path: /
    pathType: Prefix
    tls: []   # you can configure TLS later if you want HTTPS

  persistence:
    enabled: true
    storageClassName: "local-path"
    accessModes:
      - ReadWriteOnce
    size: 5Gi

  # (optional) set your timezone for dashboards
  env:
    TZ: "Asia/Dubai"

# -------------------------------
# General settings
# -------------------------------
kubeApiServer:
  enabled: true

kubeControllerManager:
  enabled: true

kubeScheduler:
  enabled: true

kubeProxy:
  enabled: true

kubeEtcd:
  enabled: true

kubeStateMetrics:
  enabled: true

nodeExporter:
  enabled: true
```

> Notes:
> - `storageClassName: local-path` uses k3s’ built‑in local path provisioner for persistent volumes.
> - Prometheus data: **10Gi**  
> - Alertmanager data: **5Gi**  
> - Grafana data: **5Gi**, with PVC so dashboards/settings survive pod restarts.
> - Grafana is exposed via **Traefik Ingress** at `http://grafana.local`.

Commit this `values.yaml` into your lab repo.

---

## 4️⃣ Install kube-prometheus-stack with Helm

From the folder where `values.yaml` lives:

```bash
helm install kube-prometheus-stack   prometheus-community/kube-prometheus-stack   --namespace monitoring   -f values.yaml
```

If you ever need to re-apply config, use `helm upgrade` instead of `install`:

```bash
helm upgrade kube-prometheus-stack   prometheus-community/kube-prometheus-stack   --namespace monitoring   -f values.yaml
```

---

## 5️⃣ Verify the Deployment

### Pods

```bash
kubectl get pods -n monitoring
```

You should eventually see pods like:

```text
NAME                                                         READY   STATUS    RESTARTS   AGE
kube-prometheus-stack-grafana-xxxxx                          3/3     Running   0          Xm
kube-prometheus-stack-kube-state-metrics-xxxxx               1/1     Running   0          Xm
kube-prometheus-stack-operator-xxxxx                         1/1     Running   0          Xm
prometheus-kube-prometheus-stack-prometheus-0                2/2     Running   0          Xm
alertmanager-kube-prometheus-stack-alertmanager-0            2/2     Running   0          Xm
kube-prometheus-stack-prometheus-node-exporter-xxxxx         1/1     Running   0          Xm
...
```

### Services

```bash
kubectl get svc -n monitoring
```

You should see at least:

```text
NAME                                         TYPE        CLUSTER-IP      EXTERNAL-IP   PORT(S)             AGE
kube-prometheus-stack-grafana               ClusterIP   10.43.x.x       <none>        80/TCP              Xm
kube-prometheus-stack-kube-prometheus       ClusterIP   10.43.x.x       <none>        9090/TCP            Xm
kube-prometheus-stack-alertmanager          ClusterIP   10.43.x.x       <none>        9093/TCP            Xm
...
```

### Ingress (Grafana)

```bash
kubectl get ingress -n monitoring
```

Expected:

```text
NAME                    CLASS     HOSTS          ADDRESS   PORTS   AGE
kube-prometheus-stack   traefik   grafana.local  <none>    80      Xm
```

(Exact name may vary if you remove `fullnameOverride`, but `HOSTS` should include `grafana.local`.)

---

## 6️⃣ Access Grafana via Traefik Ingress

1. Get your k3s node IP:

   ```bash
   kubectl get nodes -o wide
   ```

   Example:

   ```text
   NAME     STATUS   ROLES                  AGE   VERSION   INTERNAL-IP
   ubundu   Ready    control-plane,master   10d   v1.xx.x   192.168.1.50
   ```

2. On your **Windows machine**, edit the hosts file:

   ```text
   C:\Windows\System32\drivers\etc\hosts
   ```

   Add:

   ```text
   192.168.1.50   grafana.local
   ```

   (Replace `192.168.1.50` with your actual node IP or Tailscale IP.)

3. Open Grafana in your browser:

   ```text
   http://grafana.local
   ```

4. Login with the credentials configured in `values.yaml`:

   - **User:** `admin`
   - **Password:** `changeme` (or whatever you set)

Change the password on first login in a real environment.

---

## 7️⃣ Check Prometheus & Alertmanager (optional quick access)

You can port-forward if you want to access Prometheus / Alertmanager UIs quickly without ingress:

### Prometheus

```bash
kubectl port-forward -n monitoring svc/kube-prometheus-stack-prometheus 9090:9090
```

Open: <http://localhost:9090>

### Alertmanager

```bash
kubectl port-forward -n monitoring svc/kube-prometheus-stack-alertmanager 9093:9093
```

Open: <http://localhost:9093>

---

## 8️⃣ Useful Helm & Kubectl Commands

### Show currently applied values

```bash
helm get values kube-prometheus-stack -n monitoring
```

### Upgrade with a modified `values.yaml`

```bash
helm upgrade kube-prometheus-stack   prometheus-community/kube-prometheus-stack   --namespace monitoring   -f values.yaml
```

### List all resources in `monitoring` namespace

```bash
kubectl get all -n monitoring
```

### Uninstall kube-prometheus-stack (keep namespace)

```bash
helm uninstall kube-prometheus-stack -n monitoring
```

### Remove namespace completely

```bash
kubectl delete namespace monitoring
```

(You may also need to manually delete the CRDs created by kube-prometheus-stack if you want a totally clean removal.)

---

## ✅ Summary

With this setup you now have:

- **kube-prometheus-stack** deployed via **Helm repo** (`prometheus-community/kube-prometheus-stack`)
- A custom **`values.yaml`** providing:
  - Persistent storage for Prometheus & Alertmanager
  - Persistent storage for Grafana (PVC on `local-path`)
  - Traefik Ingress exposing Grafana at `http://grafana.local`
- Everything documented so you can push this file to your homelab repo and recreate the stack easily.

Get Grafana 'admin' user password by running:

  kubectl --namespace monitoring get secrets kube-prometheus-stack-grafana -o jsonpath="{.data.admin-password}" | base64 -d ; echo

Access Grafana local instance:

  export POD_NAME=$(kubectl --namespace monitoring get pod -l "app.kubernetes.io/name=grafana,app.kubernetes.io/instance=kube-prometheus-stack" -oname)
  kubectl --namespace monitoring port-forward $POD_NAME 3000

Get your grafana admin user password by running:

  kubectl get secret --namespace monitoring -l app.kubernetes.io/component=admin-secret -o jsonpath="{.items[0].data.admin-password}" | base64 --decode ; echo


Visit https://github.com/prometheus-operator/kube-prometheus for instructions on how to create & configure Alertmanager and Prometheus instances using the Operator.
