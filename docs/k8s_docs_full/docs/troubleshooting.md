# 🩺 Troubleshooting Guide for Kubernetes Home Lab

This page covers the most common issues encountered in a k3s-based Kubernetes environment and how to fix them.

---

# ❗ 1. "Unable to read /etc/rancher/k3s/k3s.yaml"

### Cause  
`kubectl` cannot access the kubeconfig file due to root permissions.

### Fix
```bash
mkdir -p ~/.kube
sudo cp /etc/rancher/k3s/k3s.yaml ~/.kube/config
sudo chown $USER:$USER ~/.kube/config
export KUBECONFIG=~/.kube/config
```

---

# ❗ 2. kubectl: command not found

Install kubectl:
```bash
brew install kubectl
```

or:
```bash
sudo apt install kubectl
```

---

# ❗ 3. k9s cannot connect to cluster

Check:
```bash
echo $KUBECONFIG
```

Should be:
```
/home/<user>/.kube/config
```

---

# ❗ 4. GitHub Authentication Fails (PAT Required)

Use a **Personal Access Token**.

Paste token using:
```
CTRL + SHIFT + V
```

---

# ❗ 5. Cannot paste in SSH console

- Windows Terminal: **CTRL + SHIFT + V**
- Proxmox Console: Use clipboard icon
- PuTTY: Right-click to paste

---

# ❗ 6. k3s not starting / stuck

Check service status:
```bash
sudo journalctl -u k3s -f
sudo systemctl restart k3s
```

---

# ❗ 7. Node Not Ready

Possible issues:
- low memory
- containerd crash
- disk space

Check:
```bash
free -h
df -h
sudo systemctl status k3s
```

---

# ❗ 8. kube-apiserver connection refused

Restart:
```bash
sudo systemctl restart k3s
```

Check ports:
```bash
sudo netstat -tulpn | grep 6443
```

---

If you need a deeper diagnostic checklist, let me know. 🔍
