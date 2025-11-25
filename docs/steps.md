-- https://www.skool.com/kubernetes-devops-could-3122/classroom

## git token :  
### Step : 1 Install Rancher desktop
Ref : https://docs.rancherdesktop.io/getting-started/installation/#linux
```
[ -r /dev/kvm ] && [ -w /dev/kvm ] || echo 'insufficient privileges'
sudo usermod -a -G kvm "$USER"

curl -s https://download.opensuse.org/repositories/isv:/Rancher:/stable/deb/Release.key | gpg --dearmor | sudo dd status=none of=/usr/share/keyrings/isv-rancher-stable-archive-keyring.gpg
echo 'deb [signed-by=/usr/share/keyrings/isv-rancher-stable-archive-keyring.gpg] https://download.opensuse.org/repositories/isv:/Rancher:/stable/deb/ ./' | sudo dd status=none of=/etc/apt/sources.list.d/isv-rancher-stable.list
sudo apt update
sudo apt install rancher-desktop
```

---

### Step : 2 Install Homebrew
Ref : https://brew.sh/

```
/bin/bash -c "$(curl -fsSL https://raw.githubusercontent.com/Homebrew/install/HEAD/install.sh)"
```

For bash (your case — Ubuntu Server):

Add these two lines to your ~/.bashrc:
```
vim ~/.bashrc
echo 'eval "$(/home/linuxbrew/.linuxbrew/bin/brew shellenv)"' >> ~/.bashrc
source ~/.bashrc
:wq
```

---

### Step : 3 Kubernetees

```
brew install kubectl k9s
sudo apt install tmux vim
curl -sfL https://get.k3s.io | sudo sh -
sudo k3s kubectl get nodes
sudo k3s kubectl get pods -A
```

Now copy config file to your home directory then you can call kubectl directly without k3s command

```
mkdir -p ~/.kube

sudo cp /etc/rancher/k3s/k3s.yaml ~/.kube/config

sudo chown $USER:$USER ~/.kube/config

k9s
    eixt :q
```

✅ Step 1 — Open your .bashrc
```
vim ~/.bashrc

# kubectl / k3s helpers

# Load bash completion (if available)
if [ -f /etc/bash_completion ]; then
  . /etc/bash_completion
fi

# kubectl + completion
if command -v kubectl &> /dev/null; then
  # Main completion
  source <(kubectl completion bash)

  # Short alias
  alias k='kubectl'

  # Enable completion for both kubectl and k
  complete -o default -F __start_kubectl kubectl
  complete -o default -F __start_kubectl k
fi

# Handy shortcuts
alias kgb='kubectl get pods'

# These need kubectx / kubens installed (optional)
alias kc='kubectx'
alias kn='kubens'

# Context switch aliases – adjust or remove if not using these names
alias kcs='kubectl config use-context admin@homelab-staging'
alias kcp='kubectl config use-context admin@homelab-production'

```

---

create vim configuration file
```
vim .vimrc

```

kubernetess command

```
k run nginx-home --image=nginx
k run httpd-home --image=httpd

k get pods

k get pods -o wide

k describe pod nginx-home

k get pod nginx-home -o yaml

k run nginx --image=nginx --dry-run=client -o yaml > nginx.yaml

```