# mlops-argo-workflows

## MLOps.Community - Argo Workflows Meetup

This repo includes the code samples and how-to's that are presented in the MLOps.Community workshop at 20 Jul 2022.


- [Community 💬](https://mlops.community/)
- [Session 🎙](https://mlops.community/watch/argo-workflows_eZ1GowAF3vHExq/)
- [Reach me on LinkedIn ✉️](https://www.linkedin.com/in/kemaltugrulyesilbek/)

## Getting Started

When you are working on Argo Workflows, it is handy to keep things local. You can try and test your workflows and other 
Kubernetes related stuff on your local without having to wait. 

We will work with Argo on our local machines. I am using a MacBook, so most of the code/examples are for Unix based OS.
However, you will be able to substitute the Mac or Unix based commands and run the examples.

## 1. Install Kind

[Kind](https://kind.sigs.k8s.io/) is a tool for running local Kubernetes clusters using Docker container “nodes”. It 
can easily spin up local Kubernetes cluster. It is (I think) lighter than minikube and definitely easier to use.

If you are on Mac, you can easily install it via: 

```shell
╰─$ brew install kind
```

For other OS, follow the steps on Kind's frontpage.

The Kind version I am using is:

```shell
╰─$ kind --version                                                                                                                               130 ↵
kind version 0.14.0
```

## 2. Configure Kind

Use Kind to spin up your very own K8s cluster on your local. Start the cluster with following command using the file 
under `installation/kind_kube.yaml`. We are gonna use the name `argo` for our cluster:

```shell
╰─$ kind create cluster --name argo --config installation/kind_kube.yaml
Creating cluster "argo" ...
 ✓ Ensuring node image (kindest/node:v1.24.0) 🖼
 ✓ Preparing nodes 📦 📦
 ✓ Writing configuration 📜
 ✓ Starting control-plane 🕹️
 ✓ Installing CNI 🔌
 ✓ Installing StorageClass 💾
 ✓ Joining worker nodes 🚜
Set kubectl context to "kind-argo"
You can now use your cluster with:

kubectl cluster-info --context kind-argo
```

Now check that you are using that cluster by switching to the right context. You should see using kubectl that you 
are in `kind-argo` context and cluster.

```shell
╰─$  kubectl config get-contexts
CURRENT   NAME                            CLUSTER                         AUTHINFO                        NAMESPACE
*         kind-argo                       kind-argo                       kind-argo
```

## 3. Configure the Cluster

We now create a namespace in our cluster. It is important for our examples that namespace name is `argo`:

```shell
╰─$ kubectl create namespace argo
namespace/argo created
```

Let’s check out to that namespace:

```shell
╰─$ kubens argo
Context "kind-argo" modified.
Active namespace is "argo".
```

## 4. Install Argo in the Cluster

Argo does not come with the K8s package. Therefore we need to install it ourselves. This can be a bit tricky due to 
version matching between Argo and K8s. After trial and error, I have found that the following manifest work with my 
Kind, and K8s versions.

Here are my versions:

```shell
$ kubectl version
Client Version: version.Info{Major:"1", Minor:"18", GitVersion:"v1.18.8", GitCommit:"9f2892aab98fe339f3bd70e3c470144299398ace", GitTreeState:"clean", BuildDate:"2020-08-13T16:12:48Z", GoVersion:"go1.13.15", Compiler:"gc", Platform:"darwin/amd64"}
Server Version: version.Info{Major:"1", Minor:"24", GitVersion:"v1.24.0", GitCommit:"4ce5a8954017644c5420bae81d72b09b735c21f0", GitTreeState:"clean", BuildDate:"2022-05-19T15:39:43Z", GoVersion:"go1.18.1", Compiler:"gc", Platform:"linux/amd64"}
╰─$ kind --version
kind version 0.14.0
```

You can use the Argo manifest under `installation/argo_install.yaml` that works with those versions.  To do that, execute:

```shell
╰─$ kubectl apply -n argo -f installation/argo_install.yaml
```

Lastly, we will create a role rebinding:

```shell
kubectl create rolebinding default-admin --clusterrole=admin --serviceaccount=argo:default -n argo
rolebinding.rbac.authorization.k8s.io/default-admin created
```

_Pro tip: We don't need to use `-n argo` in each step because we are already operating in that namespace._


## 5. Test It!

That is it. Let’s test if everything works now. We will submit the Hello-World example workflow from the ArgoProj which
basically runs the Hello-Whale example from Docker:

```shell
╰─$ argo submit https://raw.githubusercontent.com/argoproj/argo/master/examples/hello-world.yaml --watch
Name:                hello-world-hsstc
Namespace:           argo
ServiceAccount:      default
Status:              Succeeded
Conditions:
 PodRunning          False
 Completed           True
Created:             Fri Jul 15 14:52:43 +0200 (10 seconds ago)
Started:             Fri Jul 15 14:52:43 +0200 (10 seconds ago)
Finished:            Fri Jul 15 14:52:53 +0200 (now)
Duration:            10 seconds
Progress:            1/1
ResourcesDuration:   3s*(100Mi memory),3s*(1 cpu)

STEP                  TEMPLATE  PODNAME            DURATION  MESSAGE
 ✔ hello-world-hsstc  whalesay  hello-world-hsstc  4s
 
 ╰─$ kubectl logs hello-world-hsstc main
 _____________
< hello world >
 -------------
    \
     \
      \
                    ##        .
              ## ## ##       ==
           ## ## ## ##      ===
       /""""""""""""""""___/ ===
  ~~~ {~~ ~~~~ ~~~ ~~~~ ~~ ~ /  ===- ~~~
       \______ o          __/
        \    \        __/
          \____\______/
```

Looking good! We are ready to roll! 🤘

## Trouble?

Did you make a mistake on the way? Are you done with the examples and want to free up the resources? No worries, you 
can easily bring down the cluster you created with a simple command, which will release all the resources and setup!

```shell
╰─$ kind delete clusters argo                                                                                                                     130 ↵
Deleted clusters: ["argo"]
```
