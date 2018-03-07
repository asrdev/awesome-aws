# AWS CLI script for creating a Kubernetes Master and Slave Node

Using the AWS CLI tools, a Kubernetes cluster can be created using the following shell script and post-install command scripts.
The cluster is made up of one Kubernetes master node and any number of slave nodes or 'minions'. The following command shell and post-
install script invokes the AWS cli to create a single Kubernetes master:
## Master Node
```
#!/bin/sh
# Amend the settings below to suit your environment
aws ec2 run-instances --image-id ami-86110ce2 --count 1 --instance-type t2.medium --key-name PROJ --subnet-id subnet-e750fdaa --security-group-ids sg-d4253bbd --tag-specifications ResourceType=instance,Tags=[{Key=Name,Value=PROJ-HOST-K8S-MAS.VX}] --user-data file://install-k8s.txt
```
The following is the 'install-k8s.txt' file, comprising the post-install preparation of the server node:
```
#!/bin/sh
# Script to customise installation of EC2 K8s master instance
# Pull down repository for Docker/Kubernetes for Centos 7 and enable.
# Amend the following environment parameters to suit your own system.
export MINIONHN=minion02
export DOMAIN=civica.local
export MASTERHN=master01
cd /tmp
sudo yum -y install git
git clone https://github.com/billhartcivica/kubernetes-docker-rpm-repo.git
sudo cp ./kubernetes-docker-rpm-repo/virt7-docker-common-release.repo /etc/yum.repos.d/virt7-docker-common-release.repo
sudo yum -y update
# Install the Epel repository, the NTP daemon, Git tools and Nodejs. Startup the NTP daemon.
sudo yum -y install epel-release ntp nodejs
sudo systemctl start ntpd
sudo systemctl enable ntpd
# Install the Kompose, Docker, Kubernetes, Flannel and etcd components.
sudo yum -y install kompose
sudo yum -y install --enablerepo=virt7-docker-common-release kubernetes etcd flannel
# Amend the config files accordingly. Set the hostname on the server to 'master' and timezone to UK/London.
sudo sed -i -e 's/127.0.0.1/$MASTERHN.$DOMAIN/g' /etc/kubernetes/config
sudo sh -c 'echo "$MASTERHN.$DOMAIN" > /etc/hostname'
sudo hostnamectl set-hostname $MASTERHN
sudo timedatectl set-timezone Europe/London
# Edit the /etc/hosts file with the IP address/name of the local server.
ADDR=`ifconfig eth0 | grep inet | grep -v inet6 | awk '{print $2}'`
sudo sh -c 'echo "$ADDR      $MASTERHN $MASTERHN.$DOMAIN" >> /etc/hosts'
# Disable selinux and firewalls services.
sudo setenforce 0
sudo sed -i -e 's/SELINUX=enforcing/SELINUX=disabled/g' /etc/sysconfig/selinux
sudo systemctl disable iptables-services firewalld
sudo systemctl stop iptables-services firewalld
# Enable the Kubernetes components and start all of them.
sudo systemctl enable kube-apiserver kube-controller-manager kube-scheduler flanneld
sudo systemctl start kube-apiserver kube-controller-manager kube-scheduler flanneld
# Amend the etcd.conf file and apiserver files.
sudo sed -i -e 's/localhost/0.0.0.0/g' /etc/etcd/etcd.conf
sudo set -i -e 's/2380/2379/g' /etc/etcd/etcd.conf
sudo sed -i -e 's/bind-address=127.0.0.1/bind-address=0.0.0.0/g' /etc/kubernetes/apiserver
sudo sed -i -e 's/127.0.0.1/master02/g' /etc/kubernetes/apiserver
sudo sed -i -e 's/KUBE_ADMISSION_CONTROL/# KUBE_ADMISSION_CONTROL/g' /etc/kubernetes/apiserver
# Start etcd service and configure.
suod systemctl start etcd
sudo etcdctl mkdir /kube-centos/network
sudo etcdctl mk /kube-centos/network/config "{ \"Network\": \"172.30.0.0/16\", \"SubnetLen\": 24, \"Backend\": { \"Type\": \"vxlan\" } }"
#::: EDIT THE /etc/sysconfig/flanneld
sudo sed -i -e 's/127.0.0.1/master02/g' /etc/sysconfig/flanneld
sudo sed -i -e 's/atomic.io/kube-centos/g' /etc/sysconfig/flanneld
systemctl daemon-reload
sudo systemctl restart kube-apiserver kube-controller-manager kube-scheduler flanneld
```
## Slave Node
The following AWS CLI script and post-install script creates an example Kubernetes slave node.
```
#!/bin/sh
# Amend the settings below to suit your environment
aws ec2 run-instances --image-id ami-86110ce2 --count 1 --instance-type t2.medium --key-name PROJ --subnet-id subnet-e750fdaa --security-group-ids sg-d4253bbd --tag-specifications ResourceType=instance,Tags=[{Key=Name,Value=PROJ-HOST-K8S-MINX.VX}] --user-data file://install-k8s-min.txt
```
```
#!/bin/sh
# Script to customise installation of EC2 K8s instance
# Amend the following environment parameters to suit your own system.
export MINIONHN=minion02
export DOMAIN=civica.local
export MASTERHN=master01
sudo yum -y install ntp
sudo systemctl start ntpd
sudo systemctl enable ntpd
sudo yum -y update
sudo yum -y install git
cd /tmp
git clone https://github.com/billhartcivica/kubernetes-docker-rpm-repo.git
sudo cp ./kubernetes-docker-rpm-repo/virt7-docker-common-release.repo /etc/yum.repos.d/virt7-docker-common-release.repo
sudo yum -y install epel-release
sudo yum install -y python-pip
sudo pip install docker-compose
sudo yum upgrade python*
sudo yum -y install --enablerepo=virt7-docker-common-release kubernetes etcd flannel
sudo sed -i -e 's/127.0.0.1/$MINIONHN.$DOMAIN/g' /etc/kubernetes/config
sudo sh -c 'echo "$MINIONHN.$DOMAIN" > /etc/hostname'
sudo hostnamectl set-hostname $MINIONHN
sudo timedatectl set-timezone Europe/London
ADDR=`ifconfig eth0 | grep inet | grep -v inet6 | awk '{print $2}'`
sudo sh -c 'echo '$ADDR      $MINIONHN $MINIONHN.$DOMAIN' >> /etc/hosts'
sudo setenforce 0
sudo sed -i -e 's/SELINUX=enforcing/SELINUX=disabled/g' /etc/sysconfig/selinux
sudo sed -i -e 's/SELINUX=enforcing/SELINUX=disabled/g' /etc/selinux/config
sudo systemctl disable iptables-services firewalld
sudo systemctl stop iptables-services firewalld
sudo sed -i -e 's/127.0.0.1/0.0.0.0/g' /etc/kubernetes/kubelet
sudo sed -i -e 's/# KUBELET_PORT/KUBELET_PORT/g' /etc/kubernetes/kubelet
sudo set -i -e 's/hostname-override=127.0.0.1/hostname-override=$MINIONHN/g' /etc/kubernetes/kubelet
sudo sed -i -e 's/127.0.0.1:8080/$MASTERHN:8080/g' /etc/kubernetes/kubelet
sudo sed -i -e 's/127.0.0.1/$MASTERHN/g' /etc/sysconfig/flanneld
sudo sed -i -e 's/atomic.io/kube-centos/g' /etc/sysconfig/flanneld
sudo systemctl enable kube-proxy kubelet flanneld docker
# sudo systemctl start kube-proxy kubelet flanneld docker
sudo kubectl config set-cluster default-cluster --server=http://$MASTERHN:8080
sudo kubectl config set-context default-context --cluster=default-cluster --user=default-admin
sudo yum -y install nodejs
sudo kubectl config use-context default-context
```

