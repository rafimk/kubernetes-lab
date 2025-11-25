# 🧪 Kubernetes Practice & Hands‑On Labs

This file contains guided exercises to help you practice Kubernetes concepts in your home lab.

---

# 🟦 1. Create Your First Pod

```bash
k run nginx-home --image=nginx
```

Verify:
```bash
k get pods
```

---

# 🟧 2. Create a Deployment

```bash
k create deployment web --image=nginx
k get deployments
```

---

# 🟩 3. Scale Deployment

```bash
k scale deployment web --replicas=5
k get pods -o wide
```

---

# 🟪 4. Expose Deployment (ClusterIP)

```bash
k expose deployment web --port=80
k get svc
```

---

# 🟨 5. Export Pod YAML (Dry Run)

```bash
k run demo --image=httpd --dry-run=client -o yaml > demo.yaml
```

---

# 🟥 6. Apply YAML

```bash
k apply -f demo.yaml
```

---

# 🟫 7. Describe Resources

```bash
k describe pod demo
k describe deployment web
```

---

# 🔵 8. Clean Up

```bash
k delete pod demo
k delete deployment web
```

Execute inside the POD
```bash
 k exec -it nginx-docs -- /bin/bash
```

Find POD version
```bash
cat /etc/os-release
```

```bash
apt-get update
apt-get install -y htop
htop
```

```bash
k create deploy test --image=httpd --replicas=3
```

```bash
k get deployments.apps
k descibe deployments.app test
watch -n 1 "kubectl get pods"
```

# Cretae namespace 
```bash
mkdir mealie
cd mealie
k create ns mealie --dry-run=client -o yaml > namespace.yaml
k config set-context --current --namespace=mealie
```

# Change namespace 
```bash
k config set-context --current --namespace=mealie
```
or

```bash
bn mealie
```

```bash
ssh -L 9000:localhost:9000 rafi@ubundu.tail910419.ts.net
```
---

```bash
k exec -it nginx-storage -c nginx -- bash
k exec -it nginx-storage -c busybox -- sh
```


# Install homarr with Helm
```bash
helm repo add homarr-labs https://homarr-labs.github.io/charts/
helm repo update

openssl rand -hex 32

kubectl create secret generic db-encryption \
  --from-literal=db-encryption-key='378e1b758c67a6e72ac98cc37f7e86815eb4e13def45e00de402d7e878cd785a' \
  --namespace homarr
  
  helm install homarr homarr-labs/homarr \
  --namespace homarr \
  --set env.TZ="Asia/Dubai"

```

# 🧠 Want More?
I can add:

- Ingress practice  
- ConfigMap / Secret labs  
- StatefulSets  
- Volumes & PVC labs  
- Helm charts  
- CI/CD practice (GitOps + ArgoCD)

Just tell me! 🚀
