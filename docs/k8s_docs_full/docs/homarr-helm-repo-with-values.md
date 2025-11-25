
# Homarr Deployment on k3s (Ubuntu Server) via Helm Repo

This guide documents the exact steps to deploy **Homarr** on a k3s Kubernetes cluster running on Ubuntu, using the **Homarr Helm chart repo** and a custom `values.yaml` file, and expose it via Traefik Ingress.

---

## 🧱 1. Create Namespace

```bash
kubectl create namespace homarr
```

If the namespace already exists, Kubernetes will report `AlreadyExists` and you can continue.

---

## 🔐 2. Create Required Secret (`db-encryption`)

Homarr expects an encryption secret. We’ll create it manually in the `homarr` namespace.

```bash
kubectl -n homarr create secret generic db-encryption   --from-literal=db-encryption-key='CHANGE_ME_SUPER_SECRET_32CHARS'
```

You can generate a strong key with:

```bash
openssl rand -hex 32
```

Replace `CHANGE_ME_SUPER_SECRET_32CHARS` with the generated value.

Verify:

```bash
kubectl -n homarr get secret db-encryption
```

---

## 📦 3. Add Homarr Helm Repo

```bash
helm repo add homarr-labs https://homarr-labs.github.io/charts/
helm repo update
```

Check that the chart is visible:

```bash
helm search repo homarr-labs
```

You should see an entry like `homarr-labs/homarr`.

---

## ⚙️ 4. Create `values.yaml` for Homarr

Create a file named **`values.yaml`** in your repo (for example under `k8s/homarr/values.yaml`):

```bash
vim values.yaml
```

Suggested content:

```yaml
# values.yaml

env:
  TZ: "Asia/Dubai"

persistence:
  homarrDatabase:
    enabled: true
    storageClassName: "local-path"  # k3s default storage class
    size: "1Gi"

service:
  enabled: true
  type: ClusterIP
  port: 7575

ingress:
  enabled: true
  ingressClassName: "traefik"
  hosts:
    - host: homarr.local
      paths:
        - path: /
          pathType: Prefix
  tls: []  # can be configured later if you add HTTPS
```

- `TZ` → sets Homarr’s timezone.
- `persistence.homarrDatabase` → ensures the database is stored on a PVC.
- `service.type: ClusterIP` → internal service; external access is via Ingress.
- `ingress.ingressClassName: traefik` → uses k3s’ default Traefik ingress controller.
- `host: homarr.local` → you will map this hostname to your node IP in your hosts file.

Commit this `values.yaml` into your lab repo.

---

## 🚀 5. Install Homarr Using Helm + values file

From the folder where your `values.yaml` lives (or provide the full path):

```bash
helm install homarr   homarr-labs/homarr   --namespace homarr   -f values.yaml
```

If you ever need to re-apply config, use `helm upgrade`:

```bash
helm upgrade homarr   homarr-labs/homarr   --namespace homarr   -f values.yaml
```

---

## 🔎 6. Verify Deployment

Check the pods:

```bash
kubectl get pods -n homarr
```

Expected:

```text
NAME                      READY   STATUS    RESTARTS   AGE
homarr-xxxxxxxxxx-xxxxx   1/1     Running   0          Xm
```

Check the service:

```bash
kubectl get svc -n homarr
```

Expected:

```text
NAME     TYPE        CLUSTER-IP      EXTERNAL-IP   PORT(S)     AGE
homarr   ClusterIP   10.43.xxx.xxx   <none>        7575/TCP    Xm
```

Check the ingress created by the chart (because we enabled it in `values.yaml`):

```bash
kubectl get ingress -n homarr
```

Expected:

```text
NAME     CLASS     HOSTS          ADDRESS   PORTS   AGE
homarr   traefik   homarr.local   <none>    80      Xm
```

> Note: The ingress name may simply be `homarr` (created by the chart). That’s fine as long as `HOSTS` includes `homarr.local`.

---

## 🖥️ 7. Access Homarr from Your PC

1. Get your k3s node IP:

   ```bash
   kubectl get nodes -o wide
   ```

   Example:

   ```text
   NAME     STATUS   ROLES                  AGE   VERSION   INTERNAL-IP
   ubundu   Ready    control-plane,master   XXd   v1.xx.x   192.168.1.50
   ```

2. On your **Windows machine**, edit:

   ```text
   C:\Windows\System32\drivers\etc\hosts
   ```

   Add a line:

   ```text
   192.168.1.50   homarr.local
   ```

   (Replace `192.168.1.50` with your actual node IP or Tailscale IP.)

3. In your browser, open:

   ```text
   http://homarr.local
   ```

You should see the Homarr UI.

---

## 🧹 8. Useful Helm & Kubectl Commands

### Upgrade Homarr with new values

```bash
helm upgrade homarr   homarr-labs/homarr   --namespace homarr   -f values.yaml
```

### Show current values for the release

```bash
helm get values homarr -n homarr
```

### Check logs

```bash
kubectl logs -n homarr deploy/homarr
```

### List all resources in the namespace

```bash
kubectl get all -n homarr
```

### Uninstall Homarr (keep namespace)

```bash
helm uninstall homarr -n homarr
```

### Remove namespace completely

```bash
kubectl delete namespace homarr
```

---

## ✅ Summary

You now have:

- Homarr deployed via **Helm repo** (`homarr-labs/homarr`)
- A custom **`values.yaml`** controlling:
  - Timezone
  - Persistent database volume
  - Service and Traefik Ingress
- External access via `http://homarr.local` through Traefik

This markdown file is ready to be committed to your lab repo as documentation for your Homarr setup.
