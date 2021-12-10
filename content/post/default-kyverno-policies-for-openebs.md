---
title: "Default Kyverno Policies for OpenEBS"
date: 2021-09-19T11:05:09+03:00
slug: default-kyverno-policies-for-openebs
category: ["openebs, kyverno, policies, kubernetes"]
summary: Hey folks, In this blog we'll see how [OpenEBS](https://docs.openebs.io/) uses [Kyverno](https://kyverno.io/docs/) to enforce security and validate policies.As we know, PodSecurityPolicies(PSP) is being deprecated in Kubernetes 1.21 and will be removed in 1.25. So, the best alternative is Kyverno.
cover:
  image: "/img/kyverno/cover.jpg"
  alt: "img"
  caption: ""
  relative: "false"
showtoc: true
draft: false
---
Hey folks, In this blog we'll see how [OpenEBS](https://docs.openebs.io/) uses [Kyverno](https://kyverno.io/docs/) to enforce security and validate policies. 
As we know, PodSecurityPolicies(PSP) is being deprecated in Kubernetes 1.21 and will be removed in 1.25. So, the best alternative is Kyverno.

In case you're not aware of OpenEBS, PSP and Kyverno. Let me introduce you to these terms.

Let's begin!
### What is OpenEBS?

[OpenEBS](https://docs.openebs.io) is the leading open-source project for container-attached and container-native storage on Kubernetes and It adopts Container Attached Storage [(CAS)](https://www.cncf.io/blog/2020/09/22/container-attached-storage-is-cloud-native-storage-cas/) approach, where each workload is provided with a dedicated storage controller.

It implements granular storage policies and isolation that enable users to optimize storage for each specific workload which runs in userspace and does not have any Linux kernel module dependencies.


### What is PSP?

A [Pod Security Policy](https://kubernetes.io/docs/concepts/policy/pod-security-policy/) is a cluster-level resource that controls security-sensitive aspects of the pod specification.

PSP objects define a set of conditions that a pod must run with in order to be accepted into the system, as well as defaults for their related fields. PodSecurityPolicy is an optional admission controller that is enabled by default through the API, thus policies can be deployed without the PSP admission plugin enabled. This functions as a validating and mutating controller simultaneously.

### What is Kyverno?

[Kyverno](https://kyverno.io/) is an Open source policy engine designed specifically for Kubernetes. The word "Kyverno" is a Greek word for "Govern". It was originally developed by [Nirmata](https://nirmata.com/) and is now a [CNCF](https://www.cncf.io/sandbox-projects/) sandbox project.

It can validate, mutate, and generate configurations using admission controls and background scans. Kyverno policies are Kubernetes resources and do not require learning a new language, we can easily define policies declaratively in YAML.

Another, more popular alternative to Kyverno is OPA (Open Policy Agent) /Gatekeeper.

### What is OPA/Gatekeeper?

The [OPA(Open Policy Agent)](https://www.openpolicyagent.org/docs/latest/) is an open-source, general-purpose policy engine that enforces validation of objects during creation, updating, and deletion operations. OPA lets us enforce custom policies on Kubernetes objects without manually reconfiguring the Kubernetes API server.

[Gatekeeper](https://www.openpolicyagent.org/docs/latest/kubernetes-introduction/#what-is-opa-gatekeeper) is a customizable admission webhook for Kubernetes that dynamically enforces policies executed by the OPA.
OPA Gatekeeper is a specialized project providing first-class integration between OPA and Kubernetes. Because of the relationship between Open Policy Agent with Gatekeeper, the project is often written "OPA/Gatekeeper" to acknowledge these ties.

OPA/Gatekeeper uses its own declarative language called Rego, a query language. You define rules in Rego which, if invalid or returned a false expression, will trigger a constraint violation and blocks the ongoing process of creating/updating/deleting the resource.

>For more details, regarding Kyverno vs OPA/Gatekeeper you can read this [blog](https://neonmirrors.net/post/2021-02/kubernetes-policy-comparison-opa-gatekeeper-vs-kyverno/):)

### How do we use Kyverno for OpenEBS?

![alt text](https://dev-to-uploads.s3.amazonaws.com/uploads/articles/elqh27vz2bbb6e8yng42.png)

The Kyverno policies are based on the Kubernetes Pod Security Standards definitions. And, Pod Security Standard policies are organized in three groups:
- Privileged
- Baseline
- Restricted

Kyverno has already defined policies under two categories [Baseline](https://kyverno.io/policies/pod-security/#baseline) and [Restricted](https://kyverno.io/policies/pod-security/#restricted).

But, while applying these policies in OpenEBS, it blocks the creation of openebs-ndm because ndm requires privilege mode, it requires access to /dev, /proc and /sys directories for monitoring the attached devices and also to fetch the details of the attached device using various probes.

>NDM(Node Disk Manager) is an important component in the OpenEBS architecture. NDM treats block devices as resources that need to be monitored and managed just like other resources such as CPU, Memory and Network. It is a daemonset which runs on each node, detects attached block devices based on the filters and loads them as block devices custom resource into Kubernetes.

![alt text](https://dev-to-uploads.s3.amazonaws.com/uploads/articles/igzh5ybsd2u279dm0ot9.png)
  <figcaption>Fig.1 - screenshot of openebs-ndm error</figcaption>


For that reason, we need policies in the privileged category.

>"Privileged" mean here unrestricted policy, providing the widest possible level of permissions.

So, we have written eight privileged policies for OpenEBS.

| Control        | Policy        | 
| -------------  |:-------------:| 
| Capabilities   |[Allow Capabilities](https://github.com/openebs/charts/blob/master/charts/openebs/templates/kyverno/allow-capabilities.yaml)|
| Host Namespaces|[Allow Host Namespaces](https://github.com/openebs/charts/blob/master/charts/openebs/templates/kyverno/allow-host-namespaces.yaml)|   
| Host Ports     |[Allow Host Ports](https://github.com/openebs/charts/blob/master/charts/openebs/templates/kyverno/allow-host-ports.yaml)|
|Privilege Escalation   |[Allow Privilege Escalation](https://github.com/openebs/charts/blob/master/charts/openebs/templates/kyverno/allow-privilege-escalation.yaml)|
|Privileged Containers|[Allow Privilege Containers](https://github.com/openebs/charts/blob/master/charts/openebs/templates/kyverno/allow-privileged-containers.yaml)|   
| SELinux |[Allow SELinux](https://github.com/openebs/charts/blob/master/charts/openebs/templates/kyverno/allow-selinux.yaml)|
|/proc Mount Type|[Require Default Proc Mount](https://github.com/openebs/charts/blob/master/charts/openebs/templates/kyverno/allow-proc-mount.yaml)|   
|User groups |[Require User Groups](https://github.com/openebs/charts/blob/master/charts/openebs/templates/kyverno/require-user-groups.yaml)|

Let's look to our common points of policy specification:

```
apiVersion: kyverno.io/v1
kind: Policy
```
Here the kind is Policy which means policies will only apply to resources within the namespace in which they are defined. 

```
spec:
  validationFailureAction: enforce
  background: true
```
These lines represent that this policy is going to be of an “enforced” nature which means that it will block any incoming resources which will violate this policy.

```
match:
   resources:
        kinds:
           - Pod
```
These lines make sure that the policy only acts upon incoming requests of type Pod.


Now, steps to apply Kyverno policies in OpenEBS:

1.Install kyverno in your Kubernetes cluster.

```
kubectl create -f https://raw.githubusercontent.com/kyverno/kyverno/main/definitions/release/install.yaml
```
2.Apply Default [kyverno](https://github.com/openebs/charts/tree/master/charts/openebs/templates/kyverno) policies for OpenEBS using

```
kubectl apply -f <filename.yaml>
```
3.Check the default kyverno policies in the Kubernetes cluster using
```
kubectl get pol
```
<figure>
  <img src="">
  <figcaption>Fig.2 - screenshot of pol detail</figcaption>
</figure>

4.After that install OpenEBS 
```
kubectl apply -f https://openebs.github.io/charts/openebs-operator.yaml
```
Now, you'll see it's successfully installed and you can create any OpenEBS storage engine with kyverno policies.

With all this, kyverno policies for OpenEBS are ready to use!!:)

## Conclusion

Kyverno is really awesome! In this blog, we saw how easy it is to create policy with kyverno. As it is Kubernetes-native, it is straightforward to write, operate and no new language is required to learn.
There are a lot of amazing things about Kyverno for detailed information, please refer to the [official documentation](https://kyverno.io/docs/).

It's an Open-Source Project, wanna contribute?

Go and explore the [GitHub repository](https://github.com/kyverno) and make your contributions as well. You can also join the [Kyverno slack](https://app.slack.com/client/T09NY5SBT/CLGR9BJU9) channel to connect with the maintainers and other Kyverno community members.

That's all, hope you'll find this blog informative:)

> If you have any feedback or questions, feel free to reach out to me on [Twitter](https://twitter.com/NiveditaPrasa15) and I will be glad to help:)

Happy Reading!