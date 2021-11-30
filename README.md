# Kubernetes

This installation is for testing proposes, do not install it as a production environment.

# What is Kubernetes?

Kubernetes is an open-source container-orchestration system for automating application deployment, scaling, and management. It was originally designed by Google, and is now maintained by the Cloud Native Computing Foundation. [Wikipedia](https://en.wikipedia.org/wiki/Kubernetes).

## My Lab

* VMware Workstation;
* CentOS 7;
* 1 Master Nodes;
* 2 Worker Nodes (Minions);
* Windows Server as my DNS Server.

**2 vCPU | 4GB Memory | 20GB Disk**

  ### Master
  
  * chlabk8sm01.chlab.local - 192.168.255.33/24
 
  ### Node

  * chlabk8sm01.chlab.local - 192.168.255.33/24
  * chlabk8sm02.chlab.local - 192.168.255.34/24

## Prerequisites 
Before you start creating your Kubernetes cluster, We need to prepare whole virtual machines (Master and Nodes).

* Name;
* IP/DNS/GATEWAY;
* Yum update;
* Disable SELINUX;
* Disable SWAP;
* Allow Firewall Rules;
* Create DNS (A).
* All commands will be run in root "sudo su -"

## Let's Go!

### Default Steps (All Servers)

**DISABLE SWAP**
```
swapoff -a 
```
**Edit the file /etc/fstab and comment the line below with #**
```
**#**/dev/mapper/centos-swap swap                    swap    defaults        0 0
```
save the file and back to the shell - If swap is not disabled, kubelet service will not start on the masters and nodes, for Platform9 Managed Kubernetes version 3.3 and above.

**Allow Firewall Rules**
```
firewall-cmd --permanent --add-port=2379-2380/tcp

firewall-cmd --permanent --add-port=6443/tcp

firewall-cmd --permanent --add-port=10250/tcp

firewall-cmd --permanent --add-port=10251/tcp

firewall-cmd --permanent --add-port=10252/tcp

firewall-cmd --permanent --add-port=10255/tcp

firewall-cmd --reload

modprobe br_netfilter

echo '1' > /proc/sys/net/bridge/bridge-nf-call-iptables
```
**Configure DNS**

I am using Windows Server as a DNS Server.

In master server edit the file /etc/resolv.conf
```
vi /etc/resolv.conf and type your DNS

Example:
search chlab.local #(It's my domain)
nameserver 192.168.255.10 #(It's my IP domain server)
```
**Disable SELINUX**
```
setenforce 0

sed -i --follow-symlinks 's/SELINUX=enforcing/SELINUX=disabled/g' /etc/sysconfig/selinux
```
After that, reboot your servers.

**Configure Repository**
```
cat <<EOF > /etc/yum.repos.d/kubernetes.repo
[kubernetes]
name=Kubernetes
baseurl=https://packages.cloud.google.com/yum/repos/kubernetes-el7-x86_64
enabled=1
gpgcheck=1
repo_gpgcheck=1
gpgkey=https://packages.cloud.google.com/yum/doc/yum-key.gpg
        https://packages.cloud.google.com/yum/doc/rpm-package-key.gpg	 
EOF
```
**Install Kubeadm**
```
yum install kubeadm -y
```
**Install Docker**
```
yum install docker -y
```
**Enable and Start Kubelet**
```
systemctl restart kubelet && systemctl enable kubelet
```
**Enable and Start Docker**
 ```
systemctl restart docker && systemctl enable docker
```

## MASTER STEPS
INITIALIZE KBERNETES MASTER

```
kubeadm init --pod-network-cidr=192.168.255.0/24
```
RUNNING THE INSTTRUCTIONS

As you can see in your screen, we need to run these commands:

```
mkdir -p $HOME/.kube
-i /etc/kubernetes/admin.conf $HOME/.kube/config
chown $(id -u):$(id -g) $HOME/.kube/config
```

**Deploy Pod Network**

```
kubectl apply -f https://docs.projectcalico.org/v3.7/manifests/calico.yaml
```

COPY THE LINE (This line will be different for my)

```
kubeadm join 192.168.255.33:6443 --token vj3ror.foc4ijc3uy3ep1mf \
    --discovery-token-ca-cert-hash sha256:96f45bdd8c9408d7bb84f5ff6e661b1b0906496a172a49e17a2cd5ed0d6f73e2
```

## NODE STEP
```

kubeadm join 192.168.255.33:6443 --token vj3ror.foc4ijc3uy3ep1mf \
    --discovery-token-ca-cert-hash sha256:96f45bdd8c9408d7bb84f5ff6e661b1b0906496a172a49e17a2cd5ed0d6f73e2
```
## Get Nodes

To list your nodes:

```
kubectl get nodes
```

## DONE

If you followed all tips and didn't skip anything you probably had success.
