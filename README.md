# Kubernetes for Dummies workshop

This workshop sets up a Kubernetes cluster locally on Vagrant. It walks through the steps to do so. Furthermore, it assumes an extremely low level of expertise by the audience. 

Create the intial VMs with Vagrant. This will create two local Virtual Machines. A Mac laptop with Virtualbox and Vagrant were used to build this workshop. 

Kubernetes is a cluster container orchestrator. This means that it tells docker containers where to go and what to do on a bunch of computers. Cluster schedulers typically have a master and a bunch of workers, also called nodes. 

To begin with, start with creating the master [here](https://github.com/PeterLamar/kubernetes-workshop/blob/master/master.md)
