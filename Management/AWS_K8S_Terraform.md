Terraform can create and tear down infrastructure on demand, so it can prove very useful in deploying a Kubernetes cluster and allowing
it to exist only for the time required. The following script can be run by saving it as "filename.tf" - ensuring the filename suffix is 'tf'
and then running 'terraform apply'. You will need to adjust the script parameters accordingly to suit your environment. This will create
and run a Kubernetes master node.
The parameters you will need to amend are:
* access_key: This is the AWS access key you use to connect to AWS.
* secret_key: The secret key used with the access key.
* region: Your AWS region.
* key_name: The SSH key you will need to connect to the server.
* Name: Under tags. This is the name you give your K8s master.
* private_key: The PEM file corresponding to the 'key_name' paramter.
* MASTERHN and DOMAIN: The environment parameters used to set the hostname and domain for the host.
```
provider "aws" {
  access_key = "XXXXXXXXXXXXXXXXXXXX"
  secret_key = "xxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxx"
  region     = "eu-west-2"
}

resource "aws_instance" "example" {
  ami           = "ami-86110ce2"
  instance_type = "t2.micro"
  key_name = "MYKEY"
  root_block_device {
    volume_type="gp2"
    volume_size="30"
    delete_on_termination = "true"
    }
  tags {
    Name = "k8stest1"
    project = "egar"
  }

  connection {
    type = "ssh"
    user = "centos"
    private_key = "${file("MYKEY.pem")}"
  }

  provisioner "remote-exec" {
    inline = [
      "export MINIONHN=kminion02",
      "export DOMAIN=civica.local",
      "export MASTERHN=kmaster01",
      "cd /tmp",
      "sudo yum -y install git",
      "git clone https://github.com/billhartcivica/kubernetes-docker-rpm-repo.git",
      "sudo cp ./kubernetes-docker-rpm-repo/virt7-docker-common-release.repo /etc/yum.repos.d/virt7-docker-common-release.repo",
      "sudo cp ./kubernetes-docker-rpm-repo/etcdnet.sh /tmp",
      "sudo yum -y update",
      "sudo yum -y install epel-release ntp nodejs",
      "sudo systemctl start ntpd",
      "sudo systemctl enable ntpd",
      "sudo yum -y install kompose",
      "sudo yum -y install --enablerepo=virt7-docker-common-release kubernetes etcd flannel",
      "sudo sed -i -e 's/127.0.0.1/$MASTERHN.$DOMAIN/g' /etc/kubernetes/config",
      "sudo sh -c 'echo $MASTERHN.$DOMAIN > /etc/hostname'",
      "sudo hostnamectl set-hostname $MASTERHN",
      "sudo timedatectl set-timezone Europe/London",
      "export ADDR=`/usr/sbin/ifconfig eth0 | grep inet | grep -v inet6 | awk '{print $2}'`",
      "sudo chmod 666 /etc/hosts",
      "sudo echo \"$ADDR      $MASTERHN $MASTERHN.$DOMAIN\" >> /etc/hosts",
      "sudo chmod 644 /etc/hosts",
      "sudo setenforce 0",
      "sudo sed -i -e 's/SELINUX=enforcing/SELINUX=disabled/g' /etc/sysconfig/selinux",
      "sudo systemctl disable iptables-services firewalld",
      "sudo systemctl stop iptables-services firewalld",
      "sudo systemctl enable kube-apiserver kube-controller-manager kube-scheduler flanneld",
      "sudo systemctl start kube-apiserver kube-controller-manager kube-scheduler flanneld",
      "sudo sed -i -e 's/localhost/0.0.0.0/g' /etc/etcd/etcd.conf",
      "sudo set -i -e 's/2380/2379/g' /etc/etcd/etcd.conf",
      "sudo sed -i -e 's/bind-address=127.0.0.1/bind-address=0.0.0.0/g' /etc/kubernetes/apiserver",
      "sudo sed -i -e 's/127.0.0.1/0.0.0.0/g' /etc/kubernetes/apiserver",
      "sudo systemctl start etcd",
      "sudo chmod 755 /tmp/etcdnet.sh",
      "sudo /tmp/etcdnet.sh",
      "sudo sed -i -e 's/127.0.0.1/0.0.0.0/g' /etc/sysconfig/flanneld",
      "sudo sed -i -e 's/atomic.io/kube-centos/g' /etc/sysconfig/flanneld",
      "sudo systemctl daemon-reload",
      "sudo systemctl restart kube-apiserver kube-controller-manager kube-scheduler flanneld",
      "sudo kubectl config set-cluster default-cluster --server=http://\"$MASTERHN\":8080",
      "sudo kubectl config set-context default-context --cluster=default-cluster --user=default-admin",
      "sudo yum -y install nodejs",
      "sudo kubectl config use-context default-context",
      ]
  }
}

resource "aws_eip" "ip" {
  instance = "${aws_instance.example.id}"
}
```
