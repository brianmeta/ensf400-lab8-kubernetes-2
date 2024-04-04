**Name**: Brian Meta

**UCID**: 30142390

**Course**: ENSF 400

**Section**: L01, B01

# Assignment 3

In this README I will outline the steps that were taken to show the correct output of the assignment.

> Notes: All commands for each step were run in the default working directory provided by the code space.
>
> This is exactly how my terminal looked:  
> @brianmeta ➜ /workspaces/ensf400-lab8-kubernetes-2 (main) $

## **_Step 1_** - Setup

Immediately after entering the code space, I first started the minikube with the command:

```bash
$ minikube start
😄 minikube v1.32.0 on Ubuntu 20.04 (docker/amd64)
✨ Using the docker driver based on existing profile
👍 Starting control plane node minikube in cluster minikube
...
```

I then enabled ingress add-on using the command:

```bash
$ minikube addons enable ingress
💡  ingress is an addon maintained by Kubernetes. For any concerns contact minikube on GitHub.
You can view the list of minikube maintainers at: https://github.com/kubernetes/minikube/blob/master/OWNERS
    ▪ Using image registry.k8s.io/ingress-nginx/controller:v1.9.4
    ▪ Using image registry.k8s.io/ingress-nginx/kube-webhook-certgen:v20231011-8b53cabe0
    ▪ Using image registry.k8s.io/ingress-nginx/kube-webhook-certgen:v20231011-8b53cabe0
🔎  Verifying ingress addon...
🌟  The 'ingress' addon is enabled
```

## **_Step 2_** - Create App Resources

Now that the code space has been setup, I started to create the resources for each app. To avoid errors when creating these resources, I first created the deployment, then service, and lastly ingress.

Hence, all the resources for app-1 were created by executing the commands below in the same order:

1. ```bash
   $ kubectl create -f app-1-dep.yaml
   deployment.apps/app-1 created
   ```

1. ```bash
   $ kubectl create -f app-1-svc.yaml
   service/app-1-svc created
   ```

1. ```bash
   $ kubectl create -f app-1-ingress.yaml
   ingress.networking.k8s.io/app-1-ingress created
   ```

I then did the same for app-2 in the same order:

1. ```bash
   $ kubectl create -f app-2-dep.yaml
   deployment.apps/app-2 created
   ```

1. ```bash
   $ kubectl create -f app-2-svc.yaml
   service/app-2-svc created
   ```

1. ```bash
   $ kubectl create -f app-2-ingress.yaml
   ingress.networking.k8s.io/app-2-ingress created
   ```

## **_Step 3_** - Create Nginx LoadBalancers

After creating the app resources, I created the configmap (which will be used by the Nginx loadbalancer pods) using the command:

```bash
$ kubectl create -f nginx-configmap.yaml
configmap/nginx-configmap created
```

Afterwards, I created the Nginx deployment resource using the command:

```bash
$ kubectl create -f nginx-dep.yaml
deployment.apps/nginx-dep created
```

I then used the command below to check that all 5 pods are running:

```bash
$ kubectl get po
NAME                         READY   STATUS    RESTARTS   AGE
nginx-dep-86b95db777-6gvg6   1/1     Running   0          19s
nginx-dep-86b95db777-gj6gr   1/1     Running   0          19s
nginx-dep-86b95db777-nchsh   1/1     Running   0          19s
nginx-dep-86b95db777-x8v88   1/1     Running   0          19s
nginx-dep-86b95db777-z8c8z   1/1     Running   0          19s
```

Finally, I created the service and ingress resources using the following commands (in the same order):

```bash
$ kubectl create -f nginx-svc.yaml
service/nginx-svc created
```

```bash
$ kubectl create -f nginx-ingress.yaml
ingress.networking.k8s.io/nginx-ingress created
```

## **_Step 4_** - Results

Now that all of the resources have been created, I used the command below to check if I configured the applications correctly.

```bash
   $ curl http://$(minikube ip)/
```

### FIXME: SCREENSHOT RESULTS GO HERE

FIXME Note: Talk about how you see way more app1 then app2 since app2 is a canary with 30% traffic.

## **_Step 5_** - Clean Up

After completing the assignment, I deleted all the resources using the commands below in the same order:

```bash
$ kubectl delete -f nginx-ingress.yaml
ingress.networking.k8s.io "nginx-ingress" deleted

$ kubectl delete -f nginx-svc.yaml
service "nginx-svc" deleted

$ kubectl delete -f nginx-dep.yaml
deployment.apps "nginx-dep" deleted

$ kubectl delete -f nginx-configmap.yaml
configmap "nginx-configmap" deleted

$ kubectl delete -f app-2-ingress.yaml
ingress.networking.k8s.io "app-2-ingress" deleted

$ kubectl delete -f app-2-svc.yaml
service "app-2-svc" deleted

$ kubectl delete -f app-2-dep.yaml
deployment.apps "app-2" deleted

$ kubectl delete -f app-1-ingress.yaml
ingress.networking.k8s.io "app-1-ingress" deleted

$ kubectl delete -f app-1-svc.yaml
service "app-1-svc" deleted

$ kubectl delete -f app-1-dep.yaml
deployment.apps "app-1" deleted
```

> Note: All resources were created and destroyed based on the resources role/dependencies
