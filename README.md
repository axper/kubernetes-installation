# How To Manually Install Kubernetes on CentOS 7

### Introduction

[Kubernetes](https://kubernetes.io/) is an open-source system for managing containerized applications across multiple hosts, providing basic mechanisms for deployment, maintenance, and scaling of applications.

In this guide, we will install Kubernetes on a CentOS 7 server.

We'll use the following configuration throughout this guide:

* We'll use the latest Kubernetes version `1.6.6`.
* A single Kubernetes master server will also act as a node.
* All master and node services will run inside Docker containers.
* Kubernetes cluster name will be `cluster.local`.
* Kubernetes API server HTTPS port will be `6443`.
* Local docker registry URL will be `docker-registry.example.com:5000`.
* Kubernetes internal cluster DNS IP will be `10.254.0.10`.

You can use your own custom configuration values instead of the ones used above.

When we're finished, we'll be able to:

* Add, remove and scale application containers.
* Add or remove nodes to horizontally scale our application.
* Install a microservices demo application called [Sock Shop](https://github.com/microservices-demo/microservices-demo).
* View logs of all Kubernetes components and our microservice application.
* Manage the cluster DNS server so the applications can talk to each other.
* Expose our application to the internet.
* Troubleshoot problems with Kubernetes.

## Prerequisites

Before you begin this guide you'll need the following:

- A CentOS 7 server with docker installed, which will act both as a master and a node. Set up the server by following this guide: [How To Install and Use Docker on CentOS 7](https://www.digitalocean.com/community/tutorials/how-to-install-and-use-docker-on-centos-7).
- (Optional) A CentOS 7 server with docker installed, which will act as a node for horizontally scaling our application. Note that our application will work just fine without an additional node, however we will eventually need to scale our application if the load increases and our userbase grows.
- (Optional) A private docker registry server, set it up following [these instructions](https://www.digitalocean.com/community/tutorials/how-to-set-up-a-private-docker-registry-on-ubuntu-14-04). A private docker registry is a secure way to deploy your application within your team.

## Step 1 — Setup Master: Install etcd

Etcd is a distributed key-value storage which is used by Kubernetes to distribute configurations for both Kubernetes and Flannel.

Install etcd using `yum`:

```super_user
yum update
yum install etcd
```

Configure etcd by editing the `/etc/etcd/etcd.conf` file:
```
ETCD_LISTEN_CLIENT_URLS="http://127.0.0.1:2379"
ETCD_ADVERTISE_CLIENT_URLS="http://127.0.0.1:2379"
```

Enable and start the etcd service:

```super_user
systemctl enable etcd.service
systemctl start etcd.service
```

Next we'll install Flannel.

## Step 2 — Setup Master: Install Flannel

Flannel is an address management agent for overlay networks. It is used as a communication agent between Kubernetes pods.

Install Flannel:

```super_user
yum install flannel
```

Enable and start the flanneld service:

```super_user
systemctl enable flanneld.service
systemctl start flanneld.service
```

Next we'll add the master service files.

## Step 3 — Setup Master: Create and Start Kubernetes Master Services

We'll use the official Docker images from CentOS to run Kubernetes master services.

Kubernetes master consists of 3 services:

1. The API Server
2. The Controller Manager
3. The Scheduler

Edit `/etc/systemd/system/kube-apiserver.service` and add the following:

```
[Unit]
Description=Kubernetes API Server
Documentation=https://github.com/GoogleCloudPlatform/kubernetes
After=docker.service
Requires=docker.service

[Service]
TimeoutStartSec=0
Restart=always
ExecStartPre=-/usr/bin/docker stop %n
ExecStartPre=-/usr/bin/docker rm %n
ExecStartPre=/usr/bin/docker pull registry.centos.org/centos/kubernetes-apiserver
ExecStart=/usr/bin/docker run --rm --net=host -p 443:443 -v /etc/kubernetes:/etc/kubernetes:z --name %n registry.centos.org/centos/kubernetes-apiserver

[Install]
WantedBy=multi-user.target
```

Edit `/etc/systemd/system/kube-controller-manager.service` and add the following:

```
[Unit]
Description=Kubernetes Controller Manager
Documentation=https://github.com/GoogleCloudPlatform/kubernetes
After=docker.service
Requires=docker.service

[Service]
TimeoutStartSec=0
Restart=always
ExecStartPre=-/usr/bin/docker stop %n
ExecStartPre=-/usr/bin/docker rm %n
ExecStartPre=/usr/bin/docker pull registry.centos.org/centos/kubernetes-controller-manager
ExecStart=/usr/bin/docker run --rm --net=host -v /etc/kubernetes:/etc/kubernetes:z --name %n registry.centos.org/centos/kubernetes-controller-manager

[Install]
WantedBy=multi-user.target
```

Edit `/etc/systemd/system/kube-scheduler.service` and add the following:

```
[Unit]
Description=Kubernetes Scheduler Plugin
Documentation=https://github.com/GoogleCloudPlatform/kubernetes
After=docker.service
Requires=docker.service

[Service]
TimeoutStartSec=0
Restart=always
ExecStartPre=-/usr/bin/docker stop %n
ExecStartPre=-/usr/bin/docker rm %n
ExecStartPre=/usr/bin/docker pull registry.centos.org/centos/kubernetes-scheduler
ExecStart=/usr/bin/docker run --rm --net=host -v /etc/kubernetes:/etc/kubernetes:z --name %n registry.centos.org/centos/kubernetes-scheduler

[Install]
WantedBy=multi-user.target
```

Next enable and start the master services:

```super_user
systemctl start kube-apiserver.service
systemctl start kube-controller-manager.service
systemctl start kube-scheduler.service
systemctl enable kube-apiserver.service
systemctl enable kube-controller-manager.service
systemctl enable kube-scheduler.service
```

Verify that all 3 services have started successfully:

```super_user
systemctl status kube-apiserver.service
systemctl status kube-controller-manager.service
systemctl status kube-scheduler.service
```

## Step 4 — Setup Master: Add Kubernetes Network Configuration To Flannel

## Step 5 — Setup Node: Add Private Registry Mirror to Docker Configuration

## Step 6 — Setup Node: Configure Flannel

## Step 7 — Setup Node: Create and Start Kubernetes Node Services

## Step 8 — Setup Node: Increase Maximum Virtual Memory for Docker

## Step 9 — Install Cluster DNS Server

## Step 10 — Install the Microservices Demo Application

## Troubleshooting Kubernetes

## Conclusion

In this guide we manually installed and scaled our microservices application.

There are other methods of installing Kubernetes such as using a specialized Linux distribution, however that leaves us little room for customization and the internal workings of Kubernetes remain hidden from the user.

See the [official Kubernetes tutorial](https://kubernetes.io/docs/tutorials/kubernetes-basics/) for further usage instructions.

See the [debugging instructions](https://github.com/kubernetes/kubernetes/wiki/Debugging-FAQ) for troubleshooting common problems with Kubernetes.

See the [`kubectl` cheatsheet](https://kubernetes.io/docs/user-guide/kubectl-cheatsheet/) for a quick run-down of possibile commands.
