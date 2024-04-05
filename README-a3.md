**Name**: Brian Meta

**UCID**: 30142390

**Course**: ENSF 400

**Section**: L01 and B01

# Assignment 3

In this README I will outline the steps that were taken to show the correct output of the assignment.

> ## Assignment 3 Notes:
>
> - Every line that starts with '$' is a command and the output is shown under the command.
>
> - In nginx-configmap.yaml, I updated the upstream value from app-1 to app-1-svc (I did the same for app-2). The reason for this is because the assignment did not explicitly state how to name the service, so I decided to maintain the same naming scheme as the nginx files for clarity and consistency.
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

Now that the code space has been setup, I started to create the necessary resources for each app. In this case, I created the deployment and then service to avoid running into errors. Since all ingresses (including nginx) use the path /, the ingress resource for each app will be created after testing the loadbalancer; otherwise there will be an error.

Hence, all the resources for app-1 was created by executing the commands below in the same order:

1. ```bash
   $ kubectl create -f app-1-dep.yaml
   deployment.apps/app-1-dep created
   ```

1. ```bash
   $ kubectl create -f app-1-svc.yaml
   service/app-1-svc created
   ```

I then did the same for app-2:

1. ```bash
   $ kubectl create -f app-2-dep.yaml
   deployment.apps/app-2-dep created
   ```

1. ```bash
   $ kubectl create -f app-2-svc.yaml
   service/app-2-svc created
   ```

## _Step 3 - Create Nginx LoadBalancers_

After creating the resources for each app, I created the configmap (which will be used by the Nginx loadbalancer pods) using the command:

```bash
$ kubectl create -f nginx-configmap.yaml
configmap/nginx-configmap created
```

Then I created the Nginx deployment resource using the command:

```bash
$ kubectl create -f nginx-dep.yaml
deployment.apps/nginx-dep created
```

I used the command below to check that all 5 nginx pods (and the 2 app pods) are running:

```bash
$ kubectl get po
NAME                         READY   STATUS    RESTARTS   AGE
app-1-dep-86f67f4f87-7w8rq   1/1     Running   0          39s
app-2-dep-7f686c4d8d-mntpd   1/1     Running   0          25s
nginx-dep-6c6fdcb9d6-77lws   1/1     Running   0          6s
nginx-dep-6c6fdcb9d6-896mq   1/1     Running   0          6s
nginx-dep-6c6fdcb9d6-pd9j5   1/1     Running   0          6s
nginx-dep-6c6fdcb9d6-rnkp2   1/1     Running   0          6s
nginx-dep-6c6fdcb9d6-wvp4x   1/1     Running   0          6s
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

Now that the resources have been created, I used the command below to check if I configured the nginx pods correctly.

```bash
$ curl http://$(minikube ip)/
```

The results can be seen below.

![loadbalancer screenshot](/media/loadbalancer.png)

⬊ ⬋

![loadbalancer screenshot zoomed in](/media/loadbalancer_zoomed_in.png)

Since no algorithm for the loadbalancer is specified in the value of the configmap, the load balancer will assume the RR (Round Robin) algorithm. This means that each pod should alternate between app-1 and app-2 when sending traffic. This also implies that the apps should each be handling close to 50% of the traffic. The reason why we see app-1 or app-2 appear multiple times in a row is because different pods are being used during each curl. For example, lets say pod1, pod2 and pod3 are each being used as loadbalancers. There are 3 distinct possible instances (all other instance are a combination of these 3):

---

1.  If the traffic constantly gets sent to one of the pods, then we will see the apps alternating between each other.

    app-1-dep ⮕ app-2-dep ⮕ app-1-dep

2.  If one pod is in a different state then another and the pods are being alternated between each other, then the apps will also alternate with some instances of repetition.

    (Assuming it is between the same 2 pods)

    app-1-dep ⮕ app-2-dep ⮕ app-2-dep ⮕ app-1-dep

3.  If 2 or more pods are in the same state and each are chosen sequentially, then the same app will be seen repeatedly.

    app-1-dep ⮕ app-1-dep ⮕ app-1-dep ⮕ app-2-dep ⮕ app-2-dep ⮕ app-2-dep

Of course, with more pods in play, we cannot predict the sequence or reason that caused the order in which each app was called.

---

Nonetheless, _no matter how the apps are given traffic, each will get close to half of the total traffic_. From the screenshots, **app-1** is called **8 (47.06%)** times and **app-2** is called **9 (52.94%)** times, out of **17 curls**.

After confirming that the Nginx pods work correctly, I wanted to check that the canary deployment configuration also works. Thus, I started by deleting all the Nginx resources using the commands below in the same order:

```bash
$ kubectl delete -f nginx-ingress.yaml
ingress.networking.k8s.io "nginx-ingress" deleted

$ kubectl delete -f nginx-svc.yaml
service "nginx-svc" deleted

$ kubectl delete -f nginx-dep.yaml
deployment.apps "nginx-dep" deleted

$ kubectl delete -f nginx-configmap.yaml
configmap "nginx-configmap" deleted
```

Then I created the ingress for each app using the commands below, starting with app-1-ingress:

```bash
$ kubectl create -f app-1-ingress.yaml
ingress.networking.k8s.io/app-1-ingress created
```

```bash
$ kubectl create -f app-2-ingress.yaml
ingress.networking.k8s.io/app-2-ingress created
```

To check that the canary deployment was configured correctly, I used the same [curl command](#step-4---results) as before.

The results can be seen below.

![canary deployment screenshot](/media/canary_deployment.png)

⬊ ⬋

![canary deployment screenshot zoomed in](/media/canary_deployment_zoomed_in.png)

I executed the **curl command 22 times**, and after counting the number of responses from each app, I got **15 responses from app-1-dep** and **7 responses from app-2-dep**. This was expected because app-2-dep was a canary deployment set to handle 30% of the traffic. This also means that the main deployment app-1-dep should be handling 70% of the traffic. Depending on the number of curls/requests, the traffic distribution precentages mentioned earlier may not be exact, but they will get closer as more requests are made. This can be seen from the screenshot above where **app-1-dep handles 15/22 (68.18%) of the requests** and **app-2-dep handles 7/22 (31.82%) requests**.

## _Step 5 - Clean Up_

After completing the assignment, I deleted the rest of the resources using the commands below in the same order:

```bash
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

> Note: All resources were created and destroyed based on their role/dependencies
