**Name**: Brian Meta

**UCID**: 30142390

**Course**: ENSF 400

**Section**: L01 and B01

# Assignment 3

In this README I will outline the steps that were taken to show the correct output of the assignment.

> ## Assignment 3 Notes:
>
> - The assignment was updated to have app-1-ingress.yaml and app-2-ingress.yaml redirect traffic to "/" since students were having trouble, but I managed to complete the assignment by redirecting traffic to "/app", which was originally requested. Thus, I decided not to modify my solution, keeping the redirect path as "/app".
>
> - In nginx-configmap.yaml, I updated the upstream value from app-1 to app-1-svc (I did the same for app-2). The reason for this is because, we were not explicitly told how to name the service, so I decided to maintain the same naming scheme as nginx for clarity and consistency.
>
> - All commands for each step were run in the default working directory provided by the code space. This is exactly how my terminal looked:  
>   @brianmeta ➜ /workspaces/ensf400-lab8-kubernetes-2 (main) $

## _Step 1 - Setup_

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

## _Step 2 - Create App Resources_

Now that the code space has been setup, I started to create the resources for each app. To avoid errors when creating these resources, I first created the deployment, then service, and lastly ingress.

Hence, all the resources for app-1 were created by executing the commands below in the same order:

1. ```bash
   $ kubectl create -f app-1-dep.yaml
   deployment.apps/app-1-dep created
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
   deployment.apps/app-2-dep created
   ```

1. ```bash
   $ kubectl create -f app-2-svc.yaml
   service/app-2-svc created
   ```

1. ```bash
   $ kubectl create -f app-2-ingress.yaml
   ingress.networking.k8s.io/app-2-ingress created
   ```

## _Step 3 - Create Nginx LoadBalancers_

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

I then used the command below to check that all 5 pods (and the 2 app pods) are running:

```bash
$ kubectl get po
NAME                         READY   STATUS    RESTARTS   AGE
app-1-dep-86f67f4f87-c8djc   1/1     Running   0          4m22s
app-2-dep-7f686c4d8d-hph6h   1/1     Running   0          2m38s
nginx-dep-6c6fdcb9d6-688l8   1/1     Running   0          20s
nginx-dep-6c6fdcb9d6-kwtbs   1/1     Running   0          20s
nginx-dep-6c6fdcb9d6-ng77k   1/1     Running   0          20s
nginx-dep-6c6fdcb9d6-p99vs   1/1     Running   0          20s
nginx-dep-6c6fdcb9d6-zdfzv   1/1     Running   0          20s
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

## _Step 4 - Results_

Now that all of the resources have been created, I used the command below to check if I configured the applications correctly.

```bash
   $ curl http://$(minikube ip)/
```

![curl screenshot results](/media/curl_screenshot.png)

Below is a zoomed in version of the screenshot.

![curl screenshot results zoomed in](/media/curl_screenshot_zoomedin.png)

I executed the curl command 10 times, and after counting the number of responses from each app, I got 7 responses from app-1-dep and 3 responses from app-2-dep. This was expected because app-2-dep is a canary deployment that handles 30% of the traffic. This also means that the main deployment app-1-dep is handling 70% of the traffic, per the assignments instructions.

## _Step 5 - Clean Up_

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
deployment.apps "app-2-dep" deleted

$ kubectl delete -f app-1-ingress.yaml
ingress.networking.k8s.io "app-1-ingress" deleted

$ kubectl delete -f app-1-svc.yaml
service "app-1-svc" deleted

$ kubectl delete -f app-1-dep.yaml
deployment.apps "app-1-dep" deleted
```

> Note: All resources were created and destroyed based on the resources' role/dependencies
