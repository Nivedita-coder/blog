---
title: "Steps for installing K3s on Ubuntu."
date: 2021-08-28T11:05:09+03:00
slug: steps-for-installing-K3s-on-ubuntu.
category: ["ubuntu"]
summary: In the kubernetes world, there are so many distros available today -  kubeadm, Minikube and many more. But, what's K3s?
description: In the kubernetes world, there are so many distros available today -  kubeadm, Minikube and many more. But, what's K3s?
---

Let's Begin!

### What's K3s?

K3s is a fully CNCF(Cloud Native Computing Foundation) certified, compliant Kubernetes distribution by SUSE(formally Rancher Labs) that is easy to use and focused on lightness. Also, the deployment of applications with this lightweight kubernetes is faster. K3s has a built foundation on a single binary which is less than 100MB in size.

Advantage of K3s.
* Small Size
* Fast Deployment
* Lightweight
* Continuous Integration
* Simplified and Secure
* Perfect for IoT and Edge Computing.

If you want to learn more about K3s, just visit the [K3s documentation](https://rancher.com/docs/k3s/latest/en/).

Let's jump into the installation steps.

Step 1:
```
curl -sfL https://get.k3s.io | sh -
```
After installation of k3s if you run `kubectl get node`, you will get an error for that reason step 2 is important.

Step 2:

To make kubectl work run these commands.
```
mkdir -p $HOME/.kube
sudo cp /etc/rancher/k3s/k3s.* $HOME/.kube/config
sudo chown $(id -u):$(id -g) $HOME/.kube/config
```
Now, it's ready!
Go and check the node:)

It's pretty easy to install K3s, just in few seconds.

That's all, hope you have enjoyed it.Happy Reading!