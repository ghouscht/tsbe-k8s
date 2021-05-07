# Installation/Konfiguration eines Kubernetes Clusters

## docker
[Container runtimes | Docker](https://kubernetes.io/docs/setup/production-environment/container-runtimes/#docker)

Docker ist seit 1.20 eigentlich deprecated, funktioniert aber momentan noch ohne Probleme. Wir verwenden trotzdem Docker, da ihr bereits damit vertraut seit.

### Installation
<https://docs.docker.com/engine/install/ubuntu/>

Verwende folgende Befehle als Benutzer *root* um Docker zu installieren.
```bash
apt-get update
apt-get -y install \
    apt-transport-https \
    ca-certificates \
    curl \
    gnupg \
    lsb-release
curl -fsSL https://download.docker.com/linux/ubuntu/gpg | gpg --dearmor -o /usr/share/keyrings/docker-archive-keyring.gpg
echo \
    "deb [arch=amd64 signed-by=/usr/share/keyrings/docker-archive-keyring.gpg] https://download.docker.com/linux/ubuntu \
    $(lsb_release -cs) stable" | tee /etc/apt/sources.list.d/docker.list > /dev/null
apt-get update
apt-get -y install docker-ce docker-ce-cli containerd.io
```

### Konfiguration
<https://kubernetes.io/docs/setup/production-environment/container-runtimes/#docker>

Führe folgende Befehle als Benutzer *root* aus:
```bash
mkdir /etc/docker
cat <<EOF | sudo tee /etc/docker/daemon.json
{
  "exec-opts": ["native.cgroupdriver=systemd"],
  "log-driver": "json-file",
  "log-opts": {
    "max-size": "100m"
  },
  "storage-driver": "overlay2"
}
EOF
systemctl enable docker
systemctl daemon-reload
systemctl restart docker
```

## kubeadm
<https://kubernetes.io/docs/setup/production-environment/tools/kubeadm/install-kubeadm/>

### Vorbereitung
Führe folgende Befehle als Benutzer *root* aus:
```bash
modprobe br_netfilter
cat <<EOF | sudo tee /etc/modules-load.d/k8s.conf
br_netfilter
EOF

cat <<EOF | sudo tee /etc/sysctl.d/80-k8s.conf
net.bridge.bridge-nf-call-ip6tables = 1
net.bridge.bridge-nf-call-iptables = 1
net.ipv4.conf.all.rp_filter = 0
net.ipv4.conf.default.rp_filter = 0
EOF
sysctl --system
```

APT Repo hinzufügen und Pakete installieren (ebenfalls als *root* User):
```bash
apt-get update
apt-get install -y apt-transport-https ca-certificates curl
curl -fsSLo /usr/share/keyrings/kubernetes-archive-keyring.gpg https://packages.cloud.google.com/apt/doc/apt-key.gpg
echo "deb [signed-by=/usr/share/keyrings/kubernetes-archive-keyring.gpg] https://apt.kubernetes.io/ kubernetes-xenial main" | sudo tee /etc/apt/sources.list.d/kubernetes.list
apt-get update
apt-get install -y kubelet kubeadm kubectl
```

### Init vom Kubernetes Cluster
SWAP ausschalten:
```bash
swapoff -a
sed -i '/ swap / s/^/#/' /etc/fstab
```

Warum? Siehe: <https://github.com/kubernetes/kubernetes/issues/53533>

Speichere folgende Konfiguration im /root/kubeadm-config.yaml
```yaml
---
kind: ClusterConfiguration
apiVersion: kubeadm.k8s.io/v1beta2
kubernetesVersion: v1.21.0
---
kind: KubeletConfiguration
apiVersion: kubelet.config.k8s.io/v1beta1
cgroupDriver: systemd
```

Starte mit `kubeadm init --config /root/kubeadm-config.yaml` die Installation des Clusters, das dauert einige Minuten.

### root "berechtigen" / bash completion installieren
```bash
mkdir /root/.kube
cp /etc/kubernetes/admin.conf .kube/config
kubectl completion bash >/etc/bash_completion.d/kubectl
```

### Installation prüfen
Prüfe mit `kubectl get nodes` den Status des Master Servers:
```console
root@c4t-lpc01:~# kubectl get nodes
NAME                       STATUS     ROLES                  AGE   VERSION
c4t-lpc01.cloud4tsbe.lan   NotReady   control-plane,master   19m   v1.21.0
```

Warum ist dieser `NotReady`? Tipp: Verwende `kubectl describe node <name>` um das Problem zu identifizieren, findest du es?

### Installation Network plugin
Wir verwenden [Cilium](https://cilium.io/). Cilium ist in meinen Augent für neue Cluster way-to-go aufgrund der vielen Features insbesondere im Bereich Obserevability.

```bash
kubectl create -f https://raw.githubusercontent.com/cilium/cilium/1.9.6/install/kubernetes/quick-install.yaml
```

Prüfe nun noch einmal ob der Node `Ready` ist.

### vmadmin "berechtigen"
```bash
mkdir /home/vmadmin/.kube
cp /etc/kubernetes/admin.conf /home/vmadmin/.kube/config
chown vmadmin /home/vmadmin/.kube
chown vmadmin /home/vmadmin/.kube/config
```

Als User *vmadmin* testen:
```console
vmadmin@c4t-lpc01:~/Schreibtisch$ kubectl version
Client Version: version.Info{Major:"1", Minor:"21", GitVersion:"v1.21.0", GitCommit:"cb303e613a121a29364f75cc67d3d580833a7479", GitTreeState:"clean", BuildDate:"2021-04-08T16:31:21Z", GoVersion:"go1.16.1", Compiler:"gc", Platform:"linux/amd64"}
Server Version: version.Info{Major:"1", Minor:"21", GitVersion:"v1.21.0", GitCommit:"cb303e613a121a29364f75cc67d3d580833a7479", GitTreeState:"clean", BuildDate:"2021-04-08T16:25:06Z", GoVersion:"go1.16.1", Compiler:"gc", Platform:"linux/amd64"}

vmadmin@c4t-lpc01:~/Schreibtisch$ kubectl get node
NAME                       STATUS     ROLES                  AGE     VERSION
c4t-lpc01.cloud4tsbe.lan   NotReady   control-plane,master   3m38s   v1.21.0
```

## Join Token generieeren
Um den Cluster mit weiteren Nodes (worker) zu vergrössern, musst du ein sog. join Token generieeren, verwende dazu den folgenden Befehl:

```bash
kubeadm token create --print-join-command
``` 

Du erhälst einen *kubeadm join* Befehl, führe diesen auf dem System aus, das du dem Cluster hinzufügen möchtest (als Benutzer *root*).