
# Homarr Deployment on k3s (Ubuntu Server)

This guide documents the exact steps used to deploy **Homarr** on a k3s Kubernetes cluster running on Ubuntu, and expose it externally using Traefik Ingress.

---

## 🧱 1. Create Namespace

```bash
kubectl create namespace homarr
```

---

## 🔐 2. Create Required Secret (`db-encryption`)

Homarr requires an encryption secret.

```bash
kubectl -n homarr create secret generic db-encryption   --from-literal=db-encryption-key='CHANGE_ME_SUPER_SECRET_32CHARS'
```

You can generate a strong key with:

```bash
openssl rand -hex 32
```

---

## 📦 3. Install Homarr via Helm (OCI – Recommended)

```bash
helm install homarr   oci://ghcr.io/homarr-labs/charts/homarr   --namespace homarr   --set env.TZ="Asia/Dubai"
```

Check installation:

```bash
kubectl get pods -n homarr
kubectl get svc -n homarr
```

Expected service:

```
NAME     TYPE        CLUSTER-IP       PORT(S)
homarr   ClusterIP   10.43.xxx.xxx    7575/TCP
```

---

## 🌐 4. Expose Homarr via Traefik Ingress

Create file: **homarr-ingress.yaml**

```yaml
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  name: homarr-ingress
  namespace: homarr
spec:
  ingressClassName: traefik
  rules:
    - host: homarr.local
      http:
        paths:
          - path: /
            pathType: Prefix
            backend:
              service:
                name: homarr
                port:
                  number: 7575
```

Apply it:

```bash
kubectl apply -f homarr-ingress.yaml
```

Verify:

```bash
kubectl get ingress -n homarr
```

Expected:

```
NAME             CLASS     HOSTS          ADDRESS   PORTS
homarr-ingress   traefik   homarr.local   <none>    80
```

---

## 🖥️ 5. Access Homarr from Your PC

Find k3s node IP:

```bash
kubectl get nodes -o wide
```

Example: `192.168.1.50`

Edit hosts file (Windows):

```
C:\Windows\System32\drivers\etc\hosts
```

Add:

```
192.168.1.50   homarr.local
```

Open in browser:

```
http://homarr.local
```

---

## 🧹 6. Useful Commands

Restart Homarr:

```bash
helm upgrade homarr   oci://ghcr.io/homarr-labs/charts/homarr   -n homarr
```

Remove Homarr:

```bash
helm uninstall homarr -n homarr
kubectl delete namespace homarr
```

Check logs:

```bash
kubectl logs -n homarr deploy/homarr
```

---

## ✅ Status

You now have:

- k3s cluster running Homarr
- Secure DB encryption secret
- Working Traefik Ingress
- External access via `homarr.local`

This file can now be added to your lab repository.

