# mlops-argo-workflows

## MLOps.Community - Argo Workflows Meetup

This repo includes the code samples and how-to's that are presented in the MLOps.Community workshop at 20 Jul 2022.


- [Community π¬](https://mlops.community/)
- [Session π](https://mlops.community/watch/argo-workflows_eZ1GowAF3vHExq/)
- [Reach me on LinkedIn βοΈ](https://www.linkedin.com/in/kemaltugrulyesilbek/)

# Getting Started

When you are working on Argo Workflows, it is handy to keep things local. You can try and test your workflows and other 
Kubernetes related stuff on your local without having to wait. 

We will work with Argo on our local machines. I am using a MacBook, so most of the code/examples are for Unix based OS.
However, you will be able to substitute the Mac or Unix based commands and run the examples.

## 1. Install Required Tools

**Kind**

[Kind](https://kind.sigs.k8s.io/) is a tool for running local Kubernetes clusters using Docker container βnodesβ. It 
can easily spin up local Kubernetes cluster.

If you are on Mac, you can easily install it via: 

```shell
β°β$ brew install kind
```

For other OS, follow the steps on Kind's frontpage.

The Kind version I am using is:

```shell
β°β$ kind --version                                                                                                                               130 β΅
kind version 0.14.0
```

**Kubectl**

[Kubectl](https://kubernetes.io/docs/reference/kubectl/kubectl/) is the CLI tool for execute commands against 
Kubernetes. 

If you are on Mac, you can install it via:

```shell
β°β$ brew install kubectl
```

For other OS, follow the guidelines [here](https://kubernetes.io/docs/tasks/tools/#kubectl).

**Argo CLI**

We will use Argo CLI to manage our workflows.

If you are on Mac, you can easily install it via:

```shell
β°β$ brew install argoproj/tap/argo
```

For other OS, follow the guidelines on Argo's release page [here](https://github.com/argoproj/argo-workflows/releases/tag/v3.3.8).


## 2. Configure Kind

Use Kind to spin up your very own K8s cluster on your local. Start the cluster with following command using the file 
under `installation/kind_kube.yaml`. We are gonna use the name `argo` for our cluster:

```shell
β°β$ kind create cluster --name argo --config installation/kind_kube.yaml
Creating cluster "argo" ...
 β Ensuring node image (kindest/node:v1.24.0) πΌ
 β Preparing nodes π¦ π¦
 β Writing configuration π
 β Starting control-plane πΉοΈ
 β Installing CNI π
 β Installing StorageClass πΎ
 β Joining worker nodes π
Set kubectl context to "kind-argo"
You can now use your cluster with:

kubectl cluster-info --context kind-argo
```

Now check that you are using that cluster by switching to the right context. You should see using kubectl that you 
are in `kind-argo` context and cluster.

```shell
β°β$  kubectl config get-contexts
CURRENT   NAME                            CLUSTER                         AUTHINFO                        NAMESPACE
*         kind-argo                       kind-argo                       kind-argo
```

## 3. Configure the Cluster

We now create a namespace in our cluster. It is important for our examples that namespace name is `argo`:

```shell
β°β$ kubectl create namespace argo
namespace/argo created
```

Letβs check out to that namespace:

```shell
β°β$ kubens argo
Context "kind-argo" modified.
Active namespace is "argo".
```

_If you dont wanna use `kubens`, you will need to add `-n argo` to indicate namespace in your `kubectl` operations._

## 4. Install Argo in the Cluster

We need to install Argo ourselves. Let's install it from the ArgoProj repo:

```shell
β°β$ kubectl apply -n argo -f https://raw.githubusercontent.com/argoproj/argo-workflows/master/manifests/quick-start-postgres.yaml
customresourcedefinition.apiextensions.k8s.io/clusterworkflowtemplates.argoproj.io created
customresourcedefinition.apiextensions.k8s.io/cronworkflows.argoproj.io created
...
```

Now we must wait a bit for all the pods to be up and running... You can see the status of the pods via:

```shell
β°β$ watch kubectl get pods
...
```

When all the pods are up, initiate the port-forwarding so that we can access the Argo's UI:

```shell
β°β$ kubectl port-forward -n argo deployment/argo-server 2746:2746
Forwarding from 127.0.0.1:2746 -> 2746
```

## 5. Test It!

That is it. Letβs test if everything works now. We will submit the Hello-World example workflow from the ArgoProj which
basically runs the Hello-Whale example from Docker:

```shell
β°β$ argo submit https://raw.githubusercontent.com/argoproj/argo/master/examples/hello-world.yaml --watch
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
 β hello-world-hsstc  whalesay  hello-world-hsstc  4s
 
 β°β$ kubectl logs hello-world-hsstc main
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

Also, open up the Argo UI and see if it is up and running: [https://localhost:2746](https://localhost:2746).

Looking good! We are ready to roll! π€

## Trouble?

Did you make a mistake on the way? Are you done with the examples and want to free up the resources? No worries, you 
can easily bring down the cluster you created with a simple command, which will release all the resources and setup!

```shell
β°β$ kind delete clusters argo                                                                                                                     130 β΅
Deleted clusters: ["argo"]
```

# Running Examples

There are examples in numbered and ordered directories. You can run them by against your Argo deployment.

For Workflows:

```shell
argo submit <directory>/workflow.yaml
```


For CronWorkflows:

```shell
`argo cron create <directory>/workflow.yaml`
```
