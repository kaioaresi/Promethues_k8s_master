# Monitorando Kubernetes com Prometheus
---
# Topcis

- Prometheus architecture
- Run Prometheus on Kubernetes
- Application Monitoring
- PromQL
- Alerting
---

# What is Prometheus ?

- It's a metrics-based monitoring system
- Developed by `SoundClound` in 2012
- Written in `go`
- Cloud Native Computing Foundation project, make parte in 2016
- Can trigger alerts if some condition is observed to be true

# Features

- Multi-dimensional data model with time series data identified by metric name and key/value pairs
- PromQL, a flexible query language to leverage this dimensionality
- No reliance on distributed storage; single server nodes are autonomous
- Time series collection happens via a pull model over HTTP
- Pushing time series is supported via an intermediary gateway
- Targets are discovered via service discovery or static configuration
- Multiple modes of graphing and dashboarding support

# Components

The Prometheus ecosystem consists of multiple components, many of which are optional:

- The main Prometheus server which scrapes and stores time series data
- Client libraries for instrumenting application code
- A push gateway for supporting short-lived jobs
- Special-purpose exporters for services like HAProxy, StatsD, Graphite, etc.
- An alertmanager to handle alerts
- Various support tools

# Architecture

![architecture prometheus](https://prometheus.io/assets/architecture.png)

---
# Setting up a Kubernetes cluster (master)

1 - Disable swap
```
swapoff -a
```
2 - Edit: `/etc/fstab`
```
vi /etc/fstab
```

3 - Comment out swap
```
#/root/swap swap swap sw 0 0
```

4 - Add the Kubernetes repo
```
cat << EOF > /etc/yum.repos.d/kubernetes.repo
[kubernetes]
name=Kubernetes
baseurl=https://packages.cloud.google.com/yum/repos/kubernetes-el7-x86_64
enabled=1
gpgcheck=1
repo_gpgcheck=1
gpgkey=https://packages.cloud.google.com/yum/doc/yum-key.gpg https://packages.cloud.google.com/yum/doc/rpm-package-key.gpg
exclude=kube*
EOF
```

5 - Disable SELinux
```
setenforce 0
```

6 - Permanently disable SELinux:
```
vi /etc/selinux/config
```

7 - Change enforcing to disabled
```
SELINUX=disabled
```
8 - Install Kubernetes 1.11.3
```
yum install -y kubelet-1.11.3 kubeadm-1.11.3 kubectl-1.11.3 kubernetes-cni-0.6.0 --disableexcludes=kubernetes
```

9 - Start and enable the Kubernetes service
```
systemctl start kubelet && systemctl enable kubelet
```

10 - Create the k8s.conf file:

```
cat << EOF >  /etc/sysctl.d/k8s.conf
net.bridge.bridge-nf-call-ip6tables = 1
net.bridge.bridge-nf-call-iptables = 1
EOF
sysctl --system
```

11 - Create kube-config.yml:
```
vi kube-config.yml
```

12 - Add the following to kube-config.yml:
```
apiVersion: kubeadm.k8s.io/v1alpha1
kind:
kubernetesVersion: "v1.11.3"
networking:
  podSubnet: 10.244.0.0/16
apiServerExtraArgs:
  service-node-port-range: 8000-31274
```

13 - Initialize Kubernetes
```
kubeadm init --config kube-config.yml
```

14 - Copy admin.conf to your home directory

```
mkdir -p $HOME/.kube
cp -i /etc/kubernetes/admin.conf $HOME/.kube/config
chown $(id -u):$(id -g) $HOME/.kube/config
```

15 - Install flannel
```
kubectl apply -f https://raw.githubusercontent.com/coreos/flannel/v0.9.1/Documentation/kube-flannel.yml
```

16 - Patch flannel
```
vi /etc/kubernetes/manifests/kube-controller-manager.yaml
```

Add the following to kube-controller-manager.yaml:
```
--allocate-node-cidrs=true
--cluster-cidr=10.244.0.0/16
```

Then reolad kubelete
```
systemctl restart kubelet
```

## Setting up the Kubernetes Worker

Now that the setup for the Kubernetes master is complete, we will begin the process of configuring the worker node. The following actions will be executed on the Kubernetes worker.

1 - Disable swap
```
swapoff -a
```

2 - Edit: /etc/fstab
```
vi /etc/fstab
```

3 - Comment out swap
```
#/root/swap swap swap sw 0 0
```

4 - Add the Kubernetes repo
```
cat << EOF > /etc/yum.repos.d/kubernetes.repo
[kubernetes]
name=Kubernetes
baseurl=https://packages.cloud.google.com/yum/repos/kubernetes-el7-x86_64
enabled=1
gpgcheck=1
repo_gpgcheck=1
gpgkey=https://packages.cloud.google.com/yum/doc/yum-key.gpg https://packages.cloud.google.com/yum/doc/rpm-package-key.gpg
exclude=kube*
EOF
```

5 - Disable SELinux
```
setenforce 0
```

6 - Permanently disable SELinux:
```
vi /etc/selinux/config
```

7 - Change enforcing to disabled
```
SELINUX=disabled
```

8 - Install Kubernetes 1.11.3
```
yum install -y kubelet-1.11.3 kubeadm-1.11.3 kubectl-1.11.3 kubernetes-cni-0.6.0 --disableexcludes=kubernetes
```

9 - Start and enable the Kubernetes service
```
systemctl start kubelet && systemctl enable kubelet
```

10 - Create the k8s.conf file:
```
cat << EOF >  /etc/sysctl.d/k8s.conf
net.bridge.bridge-nf-call-ip6tables = 1
net.bridge.bridge-nf-call-iptables = 1
EOF
sysctl --system
```

11 - Use the join token to add the Worker Node to the cluster:
```
kubeadm join < MASTER_IP >:6443 --token < TOKEN > --discovery-token-ca-cert-hash sha256:< HASH >
```

12 - On the master node, test to see if the cluster was created properly.

13 - Get a listing of the nodes:
```
kubectl get nodes
```


---
# References

- [Docs. Prometheus](https://prometheus.io/docs/introduction/overview/)
- [Cloud Native](https://www.cncf.io/)
