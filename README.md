## <p align=center> Declarative Kubernetes deployment using k3s, FluxCD, SOPS/age, Ingress-NGINX, Lua plugin, MySQL, WordPress, Prometheus, Loki, Promtail. <br> <br> <br>  </p> 

#### Table of Contents

| **SL** | **Topic** |
| --- | --- |
| 01 | [Architecture](#01) |
| 02 | [Project Structure](#02) |
| 03 | [Setup Steps](#03) |
| 04 | [Secrets Management](#04)  |
| 05 | [FluxCD Deployment](#05) |
| 06 | [Ingress-NGINX Lua Plugin](#06)|
| 07 | [Monitoring Stack](#07)|
| 08 | [Validation](#08)|

<br>

### <a name="01">Architecture</a>

**Cluster**: Single-node k3s

**GitOps**: FluxCD bootstrapped to GitHub

**Secrets**: SOPS + age encryption

**Ingress**: ingress-nginx Helm chart with Lua plugin to redirect 404s

**Applications**: MySQL and WordPress

**Monitoring**: Prometheus, Grafana, Loki, Promtail

<br>

### <a name="02">Project Structure</a>

<img src= "https://i.postimg.cc/VsMXLjRL/project-structure.png" alt="project-structure"> <br>

<br>

### <a name="03">Setup Steps</a>

1. **Update & Install Packages**
``` bash
sudo apt update && sudo apt -y upgrade
sudo apt install -y curl git ufw vim wget gnupg apt-transport-https
sudo swapoff -a
sudo sed -i '/ swap / s/^/#/' /etc/fstab
```

2. **Install k3s**
``` bash
curl -sfL https://get.k3s.io | sh -
sudo kubectl get pods -A
mkdir -p $HOME/.kube
sudo cp /etc/rancher/k3s/k3s.yaml $HOME/.kube/config
sudo chown $(id -u):$(id -g) $HOME/.kube/config
export KUBECONFIG=$HOME/.kube/config
kubectl get nodes
```

3. **Install FluxCD**
``` bash
curl -s https://fluxcd.io/install.sh | sudo bash
flux --version
```

4. **Bootstrap FluxCD to GitHub**
``` bash
export GITHUB_TOKEN=git_personal_access_token
flux bootstrap github \
  --owner=Shadikul-Islam \
  --repository=devops-task-wedevs \
  --branch=master \
  --path=cluster \
  --personal
```

5. **Install SOPS & age**
``` bash
wget https://github.com/FiloSottile/age/releases/download/v1.2.1/age-v1.2.1-linux-amd64.tar.gz
tar -xvf age-v1.2.1-linux-amd64.tar.gz
sudo mv age/age age/age-keygen /usr/local/bin/
age-keygen --version
age --version
```
**Age Installation:**
``` bash
curl -LO https://github.com/getsops/sops/releases/download/v3.10.2/sops-v3.10.2.linux.amd64
sudo mv sops-v3.10.2.linux.amd64 /usr/local/bin/sops
sudo chmod +x /usr/local/bin/sops
sops --version
```

<br>

### <a name="04">Secrets Management</a>

**Encrypted secrets stored in cluster/secrets:**

```
mysql.enc.yaml        # MySQL credentials
wordpress.enc.yaml    # WordPress DB credentials
```


**Encrypt a secret:**

```sops --encrypt --age-file age/age.key mysql.yaml > cluster/secrets/mysql.enc.yaml```

Flux automatically decrypts and applies secrets during the reconciliation process.

<br>

### <a name="05">FluxCD Deployment</a>

Apps and infrastructure managed declaratively via Kustomizations.

reconcile commands:
``` bash
flux reconcile source git flux-system
flux reconcile kustomization flux-system
flux reconcile kustomization infrastructure
```

<br>

### <a name="06">Ingress-NGINX Lua Plugin</a>

**URL:** http://app.local/notfound

Custom Lua plugin redirects all 404 paths to https://google.com.

<br>

### <a name="07">Monitoring Stack</a>

**Prometheus**: Metrics collection

**Grafana**: Dashboards & visualization

**Loki**: Centralized logging

**Promtail**: Pod log collection

**URL**: http://app-monitor.local

The Grafana dashboard JSON file is in cluster/infrastructure/monitoring/grafana-dashboard.

<br>

### <a name="08">Validation</a>

Check pods and services:
``` bash
kubectl get pods -A
kubectl get svc -A
```

**WordPress URL**: http://app.local

Ingress 404 redirect: Any invalid path → redirects to Google
