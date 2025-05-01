# CloudRaft Metrics-App Deployment Documentation.
### 1) Set up KIND cluster:
- Created a KIND cluster with kind-config.yaml to expose ports 80 and 443.    
- Command: kind create cluster --config kind-config.yaml --name kind-cluster
- Configure ingress nginx controller to expose Cluster services.
### 2) Created Helm chart:
- Chart: cloudraft with image ghcr.io/cloudraftio/metrics-app:1.0 for deployment.
- Configured port 8080 with ClusterIP service then mapped with ingress ingress config and secret with PASSWORD=MYPASSWORD.  
- Pushed to Git repository: https://github.com/HimanshuVK1/cloudraft
- Set up an ingress resource to expose the app externally at http://localhost/
