# CloudRaft Metrics-App Deployment Documentation.
## 1) Set up KIND cluster:
- Created a KIND cluster with kind-config.yaml to expose ports 80 and 443 and this yaml file is present at /cloudraft/kind/kind-config.yaml path of this repo.       
``` 
$ kind create cluster --config kind-config.yaml --name kind-cluster 
$ kind get clusters
kind-c1

$ kubectl cluster-info
Kubernetes control plane is running at https://127.0.0.1:64292
CoreDNS is running at https://127.0.0.1:64292/api/v1/namespaces/kube-system/services/kube-dns:dns/proxy

To further debug and diagnose cluster problems, use 'kubectl cluster-info dump'.
```
<br />  <br />  
## 2) Set up ingress nginx controller in Kubernetes:
- Configure ingress nginx controller to expose Cluster services.
```
$ kubectl apply-f https://raw.githubusercontent.com/kubernetes/ingress-nginx/nginx-0.27.0/deploy/static/mandatory.yaml
$ kubectl get all -n ingress-nginx
NAME                                            READY   STATUS      RESTARTS      AGE
pod/ingress-nginx-admission-create-8z6js        0/1     Completed   0             23h
pod/ingress-nginx-admission-patch-dnx59         0/1     Completed   0             23h
pod/ingress-nginx-controller-597df84f89-crpr4   1/1     Running     1 (31m ago)   23h

NAME                                         TYPE        CLUSTER-IP      EXTERNAL-IP   PORT(S)                      AGE
service/ingress-nginx-controller             NodePort    10.96.18.33     <none>        80:30210/TCP,443:30380/TCP   23h
service/ingress-nginx-controller-admission   ClusterIP   10.96.138.220   <none>        443/TCP                      23h

NAME                                       READY   UP-TO-DATE   AVAILABLE   AGE
deployment.apps/ingress-nginx-controller   1/1     1            1           23h

NAME                                                  DESIRED   CURRENT   READY   AGE
replicaset.apps/ingress-nginx-controller-597df84f89   1         1         1       23h

NAME                                       STATUS     COMPLETIONS   DURATION   AGE
job.batch/ingress-nginx-admission-create   Complete   1/1           12s        23h
job.batch/ingress-nginx-admission-patch    Complete   1/1           13s        23h

```

<br />  <br /> 
## 3) Created Helm chart:
- Helm Chart: cloudraft with image ghcr.io/cloudraftio/metrics-app:1.0 for deployment.
- Pushed to Git repository: https://github.com/HimanshuVK1/cloudraft with path location /cloudraft/helm-chart/ in this repo.
- Configured port 8080 with ClusterIP service then mapped with ingress nginx config, created secret with PASSWORD=MYPASSWORD and volume mount with evn under the deployment template.
- Set up an ingress resource to expose the app externally at http://localhost/counter  

<br />  <br /> 
## 4) Setup ArgoCD:
- Deploy ArgoCD under K8s Cluster directly and get login password.
```
$ kubectl apply -n argocd -f https://raw.githubusercontent.com/argoproj/argo-cd/stable/manifests/install.yaml
$ kubectl get secret argocd-initial-admin-secret -n argocd -o jsonpath='{.data.password}' | base64 -d
$ kubectl get all -n argocd
NAME                                                   READY   STATUS    RESTARTS      AGE
pod/argocd-application-controller-0                    1/1     Running   1 (35m ago)   23h
pod/argocd-applicationset-controller-cc68b7b7b-k4cnd   1/1     Running   1 (35m ago)   23h
pod/argocd-dex-server-555b55c97d-5z2pl                 1/1     Running   1 (35m ago)   23h
pod/argocd-notifications-controller-65655df9d5-tp9mh   1/1     Running   1 (35m ago)   23h
pod/argocd-redis-764b74c9b9-z8n22                      1/1     Running   1 (35m ago)   23h
pod/argocd-repo-server-7dcbcd967b-5djc5                1/1     Running   1 (35m ago)   23h
pod/argocd-server-5b9cc8b776-259h8                     1/1     Running   1 (35m ago)   23h

NAME                                              TYPE           CLUSTER-IP      EXTERNAL-IP   PORT(S)                      AGE
service/argocd-applicationset-controller          ClusterIP      10.96.176.158   <none>        7000/TCP,8080/TCP            23h
service/argocd-dex-server                         ClusterIP      10.96.121.134   <none>        5556/TCP,5557/TCP,5558/TCP   23h
service/argocd-metrics                            ClusterIP      10.96.229.228   <none>        8082/TCP                     23h
service/argocd-notifications-controller-metrics   ClusterIP      10.96.11.133    <none>        9001/TCP                     23h
service/argocd-redis                              ClusterIP      10.96.198.145   <none>        6379/TCP                     23h
service/argocd-repo-server                        ClusterIP      10.96.91.54     <none>        8081/TCP,8084/TCP            23h
service/argocd-server                             LoadBalancer   10.96.231.240   <pending>     80:31038/TCP,443:31356/TCP   23h
service/argocd-server-metrics                     ClusterIP      10.96.167.40    <none>        8083/TCP                     23h

NAME                                               READY   UP-TO-DATE   AVAILABLE   AGE
deployment.apps/argocd-applicationset-controller   1/1     1            1           23h
deployment.apps/argocd-dex-server                  1/1     1            1           23h
deployment.apps/argocd-notifications-controller    1/1     1            1           23h
deployment.apps/argocd-redis                       1/1     1            1           23h
deployment.apps/argocd-repo-server                 1/1     1            1           23h
deployment.apps/argocd-server                      1/1     1            1           23h

NAME                                                         DESIRED   CURRENT   READY   AGE
replicaset.apps/argocd-applicationset-controller-cc68b7b7b   1         1         1       23h
replicaset.apps/argocd-dex-server-555b55c97d                 1         1         1       23h
replicaset.apps/argocd-notifications-controller-65655df9d5   1         1         1       23h
replicaset.apps/argocd-redis-764b74c9b9                      1         1         1       23h
replicaset.apps/argocd-repo-server-7dcbcd967b                1         1         1       23h
replicaset.apps/argocd-server-5b9cc8b776                     1         1         1       23h

NAME                                             READY   AGE
statefulset.apps/argocd-application-controller   1/1     23h
```
- Install argocd cli client for windows with chocolate and logged in via CLI.
- Created argocd application with Kubernetes manifest file to deploy the Helm chart which is present at /cloudraft/argocd/cloudraft-metrics-app.yaml path of this repo.
```
$ kubectl get all -n cloudraft
NAME                                         READY   STATUS    RESTARTS   AGE
pod/cloudraft-metrics-app-66df9d7fb9-24757   1/1     Running   0          17m

NAME                        TYPE        CLUSTER-IP      EXTERNAL-IP   PORT(S)    AGE
service/cloudraft-service   ClusterIP   10.96.152.152   <none>        8080/TCP   13h

NAME                                    READY   UP-TO-DATE   AVAILABLE   AGE
deployment.apps/cloudraft-metrics-app   1/1     1            1           13h

NAME                                               DESIRED   CURRENT   READY   AGE
replicaset.apps/cloudraft-metrics-app-56f7bf7749   0         0         0       10h
replicaset.apps/cloudraft-metrics-app-66df9d7fb9   1         1         1       9h
replicaset.apps/cloudraft-metrics-app-8494f75b97   0         0         0       10h
replicaset.apps/cloudraft-metrics-app-d464d6c94    0         0         0       13h
```
![image](https://github.com/user-attachments/assets/729db414-8de4-435f-aed8-40508989f7e8)

- After Deployment with argocd via Kubernetes manifest file check status of ArgoCD application deployment with argocd-cli.
```
$ argocd app list
NAME              CLUSTER                         NAMESPACE  PROJECT  STATUS  HEALTH   SYNCPOLICY  CONDITIONS  REPO                                      PATH        TARGET
argocd/cloudraft  https://kubernetes.default.svc  cloudraft  default  Synced  Healthy  Auto-Prune  <none>      https://github.com/HimanshuVK1/cloudraft  helm-chart  main

$ argocd app get cloudraft
Name:               argocd/cloudraft
Project:            default
Server:             https://kubernetes.default.svc
Namespace:          cloudraft
URL:                https://localhost:8080/applications/cloudraft
Source:
- Repo:             https://github.com/HimanshuVK1/cloudraft
  Target:           main
  Path:             helm-chart
SyncWindow:         Sync Allowed
Sync Policy:        Automated (Prune)
Sync Status:        Synced to main (4a491f2)
Health Status:      Healthy

GROUP              KIND        NAMESPACE  NAME                      STATUS  HEALTH   HOOK  MESSAGE
                   Secret      cloudraft  cloudraft-metrics-secret  Synced                 secret/cloudraft-metrics-secret unchanged
                   ConfigMap   cloudraft  cloudraft-metrics-py      Synced                 configmap/cloudraft-metrics-py unchanged
                   Service     cloudraft  cloudraft-service         Synced  Healthy        service/cloudraft-service unchanged
apps               Deployment  cloudraft  cloudraft-metrics-app     Synced  Healthy        deployment.apps/cloudraft-metrics-app configured
networking.k8s.io  Ingress     cloudraft  cloudraft-ingress         Synced  Healthy        ingress.networking.k8s.io/cloudraft-ingress unchanged
```

<br />  <br /> 
## 5) Check response, validated and find Root Cause Analysis for Metrics-App:
- Checked http://localhost/counter url by hitting it multiple times.  
  As result output shows on every even request it responding with "Internal Server Error".
```
1st curl localhost/counter = Counter value: 1  
2nd curl localhost/counter = Internal Server Error
3rd curl localhost/counter = Counter value: 3  
4th curl localhost/counter = Internal Server Error
5th curl localhost/counter = Counter value: 5  
6th curl localhost/counter = Internal Server Error
```
- Now checked pod logs and find out the code is showing exception in logs.
```
$ kubectl logs cloudraft-metrics-app-d464d6c94-mn4sz -n cloudraft
 * Serving Flask app 'app'
 * Debug mode: off
WARNING: This is a development server. Do not use it in a production deployment. Use a production WSGI server instead.
 * Running on all addresses (0.0.0.0)
 * Running on http://127.0.0.1:8080
 * Running on http://10.244.0.25:8080
Press CTRL+C to quit
10.244.0.7 - - [01/May/2025 19:11:39] "GET /counter HTTP/1.1" 200 -
[2025-05-01 19:11:41,309] ERROR in app: Exception on /counter [GET]
Traceback (most recent call last):
  File "/usr/local/lib/python3.12/site-packages/flask/app.py", line 1511, in wsgi_app
    response = self.full_dispatch_request()
               ^^^^^^^^^^^^^^^^^^^^^^^^^^^^
  File "/usr/local/lib/python3.12/site-packages/flask/app.py", line 919, in full_dispatch_request
    rv = self.handle_user_exception(e)
         ^^^^^^^^^^^^^^^^^^^^^^^^^^^^^
  File "/usr/local/lib/python3.12/site-packages/flask/app.py", line 917, in full_dispatch_request
    rv = self.dispatch_request()
         ^^^^^^^^^^^^^^^^^^^^^^^
  File "/usr/local/lib/python3.12/site-packages/flask/app.py", line 902, in dispatch_request
    return self.ensure_sync(self.view_functions[rule.endpoint])(**view_args)  # type: ignore[no-any-return]
           ^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^
  File "/app/app.py", line 17, in counter_page
    metrics.trigger_background_collection()
  File "/app/metrics.py", line 6, in trigger_background_collection
    delay = random.randint(180, 30)
            ^^^^^^^^^^^^^^^^^^^^^^^
  File "/usr/local/lib/python3.12/random.py", line 336, in randint
    return self.randrange(a, b+1)
           ^^^^^^^^^^^^^^^^^^^^^^
  File "/usr/local/lib/python3.12/random.py", line 319, in randrange
    raise ValueError(f"empty range in randrange({start}, {stop})")
ValueError: empty range in randrange(180, 31)
10.244.0.7 - - [01/May/2025 19:11:41] "GET /counter HTTP/1.1" 500 -
10.244.0.7 - - [01/May/2025 19:11:42] "GET /counter HTTP/1.1" 200 -
[2025-05-01 19:16:56,682] ERROR in app: Exception on /counter [GET]
Traceback (most recent call last):
  File "/usr/local/lib/python3.12/site-packages/flask/app.py", line 1511, in wsgi_app
    response = self.full_dispatch_request()
               ^^^^^^^^^^^^^^^^^^^^^^^^^^^^
  File "/usr/local/lib/python3.12/site-packages/flask/app.py", line 919, in full_dispatch_request
    rv = self.handle_user_exception(e)
         ^^^^^^^^^^^^^^^^^^^^^^^^^^^^^
  File "/usr/local/lib/python3.12/site-packages/flask/app.py", line 917, in full_dispatch_request
    rv = self.dispatch_request()
         ^^^^^^^^^^^^^^^^^^^^^^^
  File "/usr/local/lib/python3.12/site-packages/flask/app.py", line 902, in dispatch_request
    return self.ensure_sync(self.view_functions[rule.endpoint])(**view_args)  # type: ignore[no-any-return]
           ^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^
  File "/app/app.py", line 17, in counter_page
    metrics.trigger_background_collection()
  File "/app/metrics.py", line 6, in trigger_background_collection
    delay = random.randint(180, 30)
            ^^^^^^^^^^^^^^^^^^^^^^^
  File "/usr/local/lib/python3.12/random.py", line 336, in randint
    return self.randrange(a, b+1)
           ^^^^^^^^^^^^^^^^^^^^^^
  File "/usr/local/lib/python3.12/random.py", line 319, in randrange
    raise ValueError(f"empty range in randrange({start}, {stop})")
ValueError: empty range in randrange(180, 31)
10.244.0.7 - - [01/May/2025 19:16:56] "GET /counter HTTP/1.1" 500 -
10.244.0.7 - - [01/May/2025 19:16:58] "GET /counter HTTP/1.1" 200 -
[2025-05-01 20:42:23,260] ERROR in app: Exception on /counter [GET]
Traceback (most recent call last):
  File "/usr/local/lib/python3.12/site-packages/flask/app.py", line 1511, in wsgi_app
    response = self.full_dispatch_request()
               ^^^^^^^^^^^^^^^^^^^^^^^^^^^^
  File "/usr/local/lib/python3.12/site-packages/flask/app.py", line 919, in full_dispatch_request
    rv = self.handle_user_exception(e)
         ^^^^^^^^^^^^^^^^^^^^^^^^^^^^^
  File "/usr/local/lib/python3.12/site-packages/flask/app.py", line 917, in full_dispatch_request
    rv = self.dispatch_request()
         ^^^^^^^^^^^^^^^^^^^^^^^
  File "/usr/local/lib/python3.12/site-packages/flask/app.py", line 902, in dispatch_request
    return self.ensure_sync(self.view_functions[rule.endpoint])(**view_args)  # type: ignore[no-any-return]
           ^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^
  File "/app/app.py", line 17, in counter_page
    metrics.trigger_background_collection()
  File "/app/metrics.py", line 6, in trigger_background_collection
    delay = random.randint(180, 30)
            ^^^^^^^^^^^^^^^^^^^^^^^
  File "/usr/local/lib/python3.12/random.py", line 336, in randint
    return self.randrange(a, b+1)
           ^^^^^^^^^^^^^^^^^^^^^^
  File "/usr/local/lib/python3.12/random.py", line 319, in randrange
    raise ValueError(f"empty range in randrange({start}, {stop})")
ValueError: empty range in randrange(180, 31)
10.244.0.7 - - [01/May/2025 20:42:23] "GET /counter HTTP/1.1" 500 -
```
- As per the logs the error occurs in the trigger_background_collection function in metrics.py due to an invalid range in random.randint(180, 30). The random.randint(a, b) function requires the first argument (a) to be less than or equal to the second argument (b), but here, 180 is greater than 30, causing the ValueError: empty range in randrange(180, 31).
- To Fix it Reverse the arguments in the random.randint call to ensure the lower bound is less than or equal to the upper bound. For example, if you want a random integer between 30 and 180 (inclusive).
- To do that fix I have updated container's program file by executing a command with deployment.
- After this source code changes app started to work properly and able to receive response with even number of requests as well but I noticed one more thing is that wheneven I hit a even number request then it taking longer time to respond as compair to odd number requests. After checking further more I found out "@app.route('/counter')" route is calling "metrics.trigger_background_collection()" function on every request which matches this condition "if counter % 2 == 0" and in "metrics.trigger_background_collection()" a time.sleep(delay) function is present with take values from random.randint(30, 180) which adds a random number between range of 30 to 180. Now to remove this delay I have modified random.randint(30, 180) values to this random.randint(0, 0), so randint() can only provide 0 value for the sleep(). After doing all this things and updating container's code it start to provide response instantly.
- To Do all the changes under the pods i am using configmaps with helm chart which is present at /cloudraft/helm-chart/templates/configmap.yaml of this repo and injected it under the pods container and update code at /app/metrics.py 
```
1st curl localhost/counter = Counter value: 1  
2nd curl localhost/counter = Counter value: 2  
3rd curl localhost/counter = Counter value: 3  
4th curl localhost/counter = Counter value: 4  
5th curl localhost/counter = Counter value: 5  
6th curl localhost/counter = Counter value: 6
```
- Now these are new pod logs.
```
$ kubectl logs cloudraft-metrics-app-66df9d7fb9-24757 -n cloudraft
 * Serving Flask app 'app'
 * Debug mode: off
WARNING: This is a development server. Do not use it in a production deployment. Use a production WSGI server instead.
 * Running on all addresses (0.0.0.0)
 * Running on http://127.0.0.1:8080
 * Running on http://10.244.0.14:8080
Press CTRL+C to quit
10.244.0.2 - - [02/May/2025 08:05:12] "GET /counter HTTP/1.1" 200 -
10.244.0.2 - - [02/May/2025 08:05:14] "GET /counter HTTP/1.1" 200 -
10.244.0.2 - - [02/May/2025 08:05:16] "GET /counter HTTP/1.1" 200 -
10.244.0.2 - - [02/May/2025 08:05:18] "GET /counter HTTP/1.1" 200 -
10.244.0.2 - - [02/May/2025 08:05:19] "GET /counter HTTP/1.1" 200 -
10.244.0.2 - - [02/May/2025 08:05:21] "GET /counter HTTP/1.1" 200 -
```
- As a result getting responses from the metrics-app become stable for http://localhost/counter URL.
- Check if created env variable and configmap changes are present under the pod.
```
1st exec under the Pod.
$ kubectl exec -it cloudraft-metrics-app-66df9d7fb9-24757 -n cloudraft -- sh

2nd Check env variables under pod.
$ kubectl exec -it cloudraft-metrics-app-576bfdf945-rxxw8 -n cloudraft -- sh
$ printenv
KUBERNETES_SERVICE_PORT=443
KUBERNETES_PORT=tcp://10.96.0.1:443
HOSTNAME=cloudraft-metrics-app-576bfdf945-rxxw8
HOME=/
GPG_KEY=7169605F62C751356D054A26A821E680E5FA6305
CLOUDRAFT_SERVICE_SERVICE_PORT_HTTP=8080
PYTHON_SHA256=07ab697474595e06f06647417d3c7fa97ded07afc1a7e4454c5639919b46eaea
CLOUDRAFT_SERVICE_SERVICE_HOST=10.96.152.152
CLOUDRAFT_SERVICE_PORT_8080_TCP_ADDR=10.96.152.152
TERM=xterm
KUBERNETES_PORT_443_TCP_ADDR=10.96.0.1
CLOUDRAFT_SERVICE_PORT_8080_TCP_PORT=8080
CLOUDRAFT_SERVICE_PORT_8080_TCP_PROTO=tcp
PATH=/usr/local/bin:/usr/local/sbin:/usr/local/bin:/usr/sbin:/usr/bin:/sbin:/bin
KUBERNETES_PORT_443_TCP_PORT=443
CLOUDRAFT_SERVICE_SERVICE_PORT=8080
KUBERNETES_PORT_443_TCP_PROTO=tcp
CLOUDRAFT_SERVICE_PORT=tcp://10.96.152.152:8080
LANG=C.UTF-8
PYTHON_VERSION=3.12.10
CLOUDRAFT_SERVICE_PORT_8080_TCP=tcp://10.96.152.152:8080
KUBERNETES_SERVICE_PORT_HTTPS=443
KUBERNETES_PORT_443_TCP=tcp://10.96.0.1:443
KUBERNETES_SERVICE_HOST=10.96.0.1
PWD=/app
PASSWORD=MYPASSWORD

3rd Check if configmap is injected or not in /app/metrics.py file.
$ cat metrics.py
import collector
import random
import time

def trigger_background_collection():
    delay = random.randint(0, 0)
    time.sleep(delay)
    collector.launch_collector()
```

<br />  <br /> 
## 6) Best practices are follows: 
### 1st Used Helm for reproducible deployments.
- Added templese are deployment, service, ingerss-nginx, configmap, secret.
- For pod added resource allocations to limit their resource usage.
- Included secrets management for sensitive data and configmap for runtime fixes.
- For pod security read-only file system, non-root user pod access which will prevent kernal access from pod or container.
- Added rolling update strategy and liveness & readyness probs for zero downtime deployment and self heal purposes respectivily.
### 2nd Automated deployment with ArgoCD for GitOps.
### 3rd Configured ingress for external access.

