
Stand up the Virtual Machines if they are not already running

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

Use this same pattern to install the kube-apiserver. This component creates a REST and cli interface for the kubelet, the main kubernetes executable which we will install next. 

```
wget https://storage.googleapis.com/kubernetes-release/release/v1.1.2/bin/linux/amd64/kube-apiserver
chmod +x kube-apiserver
sudo cp kube-apiserver /usr/bin
sudo cp kube-apiserver.service /etc/systemd/system
```

Next trigger the kube-apiserver server with the sytemctl service

```
sudo systemctl start kube-apiserver
sudo systemctl enable kube-apiserver
```

Test the kube-apiserver to make sure its alive

```
sudo systemctl status kube-apiserver
curl http://localhost:8080/api/v1/nodes
```

Now lets install the kubelet. The Kubelet schedules containers and functions as the main executable of Kubernetes. It must run on each worker node to schedule docker containers. We are including it here in the master to utlize it as a worker as well. 

```
wget https://storage.googleapis.com/kubernetes-release/release/v1.1.2/bin/linux/amd64/kubelet
chmod +x kubelet
sudo cp kubelet /usr/bin
sudo cp kubelet.service /etc/systemd/system
```

Test the kubelet to make sure its alive

```
sudo systemctl status kubelet
```

Finally, lets download the CLI component kubectl so we can query the cluster

```
wget https://storage.googleapis.com/kubernetes-release/release/v1.1.2/bin/linux/amd64/kubectl
chmod +x kubeclt
```

Query the cluster for the number of nodes

```
kubectl get nodes
```

If everything went according to plan, then you should see something that resembles the following. 

```
NAME      LABELS                           STATUS    AGE
kmaster   kubernetes.io/hostname=kmaster   Ready     2d
```

Now lets download and install the kube-scheduler. The scheduler assigns which pod to place the workload. 

```
wget https://storage.googleapis.com/kubernetes-release/release/v1.1.2/bin/linux/amd64/kube-scheduler
chmod +x kube-scheduler
sudo cp kube-scheduler /usr/bin
sudo cp kube-scheduler.service /etc/systemd/system
```

Trigger the scheduler to start with each restart

```
sudo systemctl start kube-scheduler
sudo systemctl enable kube-scheduler
```

Test the kube-scheduler to make sure its alive. 

```
sudo systemctl status kube-scheduler
```

Now post a simple nginx job without a specified job in it's yaml using kubectl. This will force the kube-scheduler to find a node for our workload. 

```
kubectl create --filename nginx-scheduled.yaml
```

You should see both pods assigned if everything is working (2/2). Alternatively the pods will be stuck waiting and will not be assigned (0/2)

```
nginx-scheduled  2/2       Running   0          35m
```

Next is the kube-controller-manager. This pod creates the services and ensures the replication pods confirm to the desired behavior. Without it, our guestbook example fails to schedule pods that correspond to our services. To test this, try the guestbook example as is.

```
git clone https://github.com/kubernetes/kubernetes.git
cd kubernetes
kubectl create -f examples/guestbook/all-in-one/guestbook-all-in-one.yaml
```

This will fail to create the pods, and only the services will be visible. To verify, run the following with kubectl:

```
kubectl get services
kubectl get pods
```

Lets enable the download and enable the controller manager to fix this example.

```
wget https://storage.googleapis.com/kubernetes-release/release/v1.1.2/bin/linux/amd64/kube-controller-manager
chmod +x kube-controller-manager
sudo cp kube-controller-manager /usr/bin
sudo cp kube-controller-manager.service /etc/systemd/system
sudo systemctl start kube-controller-manager
sudo systemctl enable kube-controller-manager
```

Now when the guestbook example is triggered, the pods should be created as expected

```
kubectl create -f examples/guestbook/all-in-one/guestbook-all-in-one.yaml
```

Finally, the last component, kube-proxy, connects services with their respective pods. Without this, you can curl a service and will not reach its respective pods. Try that now

```
kubectl get services
curl 10.0.226.213 (ip of frontend service)
```

You should receive no response. Lets install the kube-proxy and try this again

```
wget https://storage.googleapis.com/kubernetes-release/release/v1.1.2/bin/linux/amd64/kube-proxy
chmod +x kube-proxy
sudo cp kube-proxy /usr/bin
sudo cp kube-proxy.service /etc/systemd/system
sudo systemctl start kube-proxy
sudo systemctl enable kube-proxy
```

And now curl the front end service

```
kubectl get services
curl 10.0.226.213 (ip of frontend service)
```

This should provide a basic kubernetes master. 
