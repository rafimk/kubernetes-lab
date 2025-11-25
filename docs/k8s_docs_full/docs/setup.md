# 🏗️ Kubernetes Home Lab Setup Guide

This document provides a **complete setup guide** for building a Kubernetes home lab on Ubuntu using **K3s**, **kubectl**, and additional developer tools.

---

# 1️⃣ System Requirements

- Ubuntu Server 22.04 or 24.04  
- VM hosted on Proxmox/KVM/QEMU  
- Minimum: 2 vCPU, 4GB RAM, 20GB disk  
- Internet connectivity  
- sudo privileges  

---

# 2️⃣ Install Rancher Desktop (Optional)

Rancher Desktop offers a GUI + tooling support.

### Enable KVM
```bash
[ -r /dev/kvm ] && [ -w /dev/kvm ] || echo "insufficient privileges"
sudo usermod -a -G kvm "$USER"
```

### Install Rancher Desktop
```bash
curl -s https://download.opensuse.org/repositories/isv:/Rancher:/stable/deb/Release.key | gpg --dearmor | sudo dd status=none of=/usr/share/keyrings/isv-rancher.gpg

echo 'deb [signed-by=/usr/share/keyrings/isv-rancher.gpg] https://download.opensuse.org/repositories/isv:/Rancher:/stable/deb/ ./' | sudo tee /etc/apt/sources.list.d/rancher-desktop.list

sudo apt update
sudo apt install rancher-desktop
```

---

# 3️⃣ Install Homebrew (Linuxbrew)

```bash
/bin/bash -c "$(curl -fsSL https://raw.githubusercontent.com/Homebrew/install/HEAD/install.sh)"
echo 'eval "$(/home/linuxbrew/.linuxbrew/bin/brew shellenv)"' >> ~/.bashrc
source ~/.bashrc
```

Verify:
```bash
brew --version
```

---

# 4️⃣ Install Kubernetes Tools

```bash
brew install kubectl k9s
sudo apt install -y tmux vim
```

---

# 5️⃣ Install K3s (Lightweight Kubernetes)

Run:
```bash
curl -sfL https://get.k3s.io | sudo sh -
```

Check status:
```bash
sudo systemctl status k3s
```

🧩 Install kubectx + kubens Using Brew
```bash
brew install kubectx
```

This installs both:

kubectx → switch Kubernetes contexts

kubens → switch namespaces

```bash
kubens
kubens default
kubens mealie
```


Check nodes:
```bash
sudo k3s kubectl get nodes
sudo k3s kubectl get pods -A
```

---

# 6️⃣ Fix kubeconfig (Permission Issue Resolution)

K3s stores kubeconfig at:
```bash
/etc/rancher/k3s/k3s.yaml
```

Copy for user access:

```bash
mkdir -p ~/.kube
sudo cp /etc/rancher/k3s/k3s.yaml ~/.kube/config
sudo chown $USER:$USER ~/.kube/config
```

Set environment variable:
```bash
echo 'export KUBECONFIG=$HOME/.kube/config' >> ~/.bashrc
source ~/.bashrc
```

Test:
```bash
kubectl get nodes
```

---

# 7️⃣ Enhance Shell with kubectl Aliases + Autocomplete

Edit ~/.bashrc:
```bash
vim ~/.bashrc
```

Add:

```bash
# Load bash completion
if [ -f /etc/bash_completion ]; then
  . /etc/bash_completion
fi

# kubectl alias & completion
if command -v kubectl &> /dev/null; then
  alias k='kubectl'
  source <(kubectl completion bash)
  complete -o default -F __start_kubectl k
  complete -o default -F __start_kubectl kubectl
fi

# Shortcuts
alias kgb='kubectl get pods'
alias kc='kubectx'
alias kn='kubens'
```

Reload:
```bash
source ~/.bashrc
```

---

# 8️⃣ Setup Vim for YAML Editing

Create ~/.vimrc:
```bash
vim ~/.vimrc
```

Add:

```vim
set number
set tabstop=2
set shiftwidth=2
set expandtab
syntax on
```

---

# 9️⃣ Configure tmux (Vim-style navigation)

Create ~/.tmux.conf:
```tmux
setw -g mode-keys vi
set -g mouse on

bind -n M-h select-pane -L
bind -n M-j select-pane -D
bind -n M-k select-pane -U
bind -n M-l select-pane -R
```

Reload:
```bash
tmux source-file ~/.tmux.conf
```

---

# 🔟 Validate Cluster

```bash
kubectl get nodes
kubectl get pods -A
```

You now have a *fully functional* Kubernetes lab. 🎉

# 🔟 Install helm
```bash
brew install helm
```