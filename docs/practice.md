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

---

# 🧠 Want More?
I can add:

- Ingress practice  
- ConfigMap / Secret labs  
- StatefulSets  
- Volumes & PVC labs  
- Helm charts  
- CI/CD practice (GitOps + ArgoCD)

Just tell me! 🚀

