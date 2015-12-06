# Kubernetes for Dummies workshop

This workshop sets up a Kubernetes cluster locally on Vagrant. It walks through the steps to do so. Furthermore, it assumes an extremely low level of expertise by the audience. 

Create the intial VMs with Vagrant. This will create two local Virtual Machines. A Mac laptop with Virtualbox and Vagrant were used to build this workshop. 

```
Vagrant up
```

The first step is to connect to the master 

```
vagrant ssh master
```

Next, install Docker, more info [here](https://docs.docker.com/engine/installation/fedora)

```
sudo yum install docker
sudo systemctl start docker
sudo systemctl enable docker
```

Install etcd. The first step is to download etcd and copying the executable to /usr/bin. More info [here](https://github.com/coreos/etcd/releases)

```
curl -L  https://github.com/coreos/etcd/releases/download/v2.2.2/etcd-v2.2.2-linux-amd64.tar.gz -o etcd-v2.2.2-linux-amd64.tar.gz
tar xzvf etcd-v2.2.2-linux-amd64.tar.gz
cd etcd-v2.2.2-linux-amd64
sudo cp etcd etcdctl /usr/bin
```

The second step is to schedule systemctl to start and enable the etcd binary on future restarts
```
sudo cp etcd.service /etc/systemd/system
sudo systemctl start etcd
sudo systemctl enable etcd
```
