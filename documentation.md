# CloudRaft Metrics-App Deployment Documentation.
### 1) Set up KIND cluster:
- Created a KIND cluster with kind-config.yaml to expose ports 80 and 443.    
- Command: kind create cluster --config kind-config.yaml --name kind-cluster
### 2) Set up ingress:
- Configure ingress nginx controller to expose Cluster services.
### 3) Created Helm chart:
- Chart: cloudraft with image ghcr.io/cloudraftio/metrics-app:1.0 for deployment.
- Configured port 8080 with ClusterIP service then mapped with ingress ingress config and secret with PASSWORD=MYPASSWORD.  
- Pushed to Git repository: https://github.com/HimanshuVK1/cloudraft
- Set up an ingress resource to expose the app externally at http://localhost/counter
### 4) Setup ArgoCD:
- Deploy ArgoCD under K8s Cluster directly.
- Install argocd cli for windows with chocolate and logged in via CLI.
- Created argocd application with Kubernetes manifest file to deploy the Helm chart which is present at /cloudraft/argocd/cloudraft-metrics-app.yaml path of this repo.
### 5) Validated Metrics-App:
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

