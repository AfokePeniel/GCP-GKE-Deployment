# Steps to Deploy an Application to GCP Kubernetes Cluster

This guide walks through setting up and deploying applications to Google Kubernetes Engine (GKE).
## Prerequisites Installation
Ensure the following are set up:

- Google Cloud Account: Access to a GCP project with billing enabled.
- gcloud CLI: Installed and authenticated to your GCP account.
- kubectl CLI: Installed and configured.
- Docker: Installed for building container images.

### Install Google Cloud SDK
``` 
echo "deb [signed-by=/usr/share/keyrings/cloud.google.gpg] https://packages.cloud.google.com/apt cloud-sdk main" | sudo tee -a /etc/apt/sources.list.d/google-cloud-sdk.list
sudo apt-get install apt-transport-https ca-certificates gnupg
curl https://packages.cloud.google.com/apt/doc/apt-key.gpg | sudo apt-key --keyring /usr/share/keyrings/cloud.google.gpg add -
sudo apt-get update && sudo apt-get install google-cloud-sdk
```

### Verify Installation
``` 
gcloud --version
```

### Install GKE-Gcloud Auth Plugin
``` 
sudo apt-get install google-cloud-sdk-gke-gcloud-auth-plugin
```

### Verify the Plugin
``` 
gke-gcloud-auth-plugin --version
```

### Install Kubectl
Refer to the [kubectl installation guide](https://kubernetes.io/docs/tasks/tools/install-kubectl/).

### Install Docker
Refer to the [Docker installation guide](https://docs.docker.com/get-docker/).

### Verify Docker Installation
``` 
docker --version
```

Make sure to replace the placeholder values with your actual project ID, compute zone, and other relevant information.
<Cluster-Name>
<YOUR_PROJECT_ID>
<YOUR_COMPUTE_ZONE>

## Configure Google Cloud SDK

### Authenticate with Gcloud
This will authenticate using your active Google browser account if opened.
``` 
gcloud auth login
```

### Set Project ID
``` 
gcloud config set project YOUR_PROJECT_ID
```

### Set Compute Zone
``` 
gcloud config set compute/zone YOUR_COMPUTE_ZONE
```

### Verify Configuration
``` 
gcloud config list
```

## Enable Required Google Services
Ensure the following services are enabled:
``` 
gcloud services enable container.googleapis.com \
                        storage.googleapis.com \
                        artifactregistry.googleapis.com \
                        cloudbuild.googleapis.com \
                        containerregistry.googleapis.com \
                        iamcredentials.googleapis.com \
                        compute.googleapis.com \
                        logging.googleapis.com \
                        monitoring.googleapis.com
```

### Verify Enabled Services
``` 
gcloud services list --enabled
```

## Create Kubernetes Cluster
``` 
gcloud container clusters create <Cluster-Name> --num-nodes=3
```

## Configure Kubectl
Set kubectl to use gcloud kubeconfig:
``` 
gcloud container clusters get-credentials <Cluster-Name> --zone YOUR_COMPUTE_ZONE --project YOUR_PROJECT_ID
```

### Verify Current Context
``` 
kubectl config current-context
```

## Application Deployment

### Create Application Files
Create the following files:
``` 
vi app.py
vi deployment.yml
vi service.yml
vi Dockerfile
vi requirements.txt
```

### Build the Docker Image
``` 
docker build -t gcr.io/YOUR_PROJECT_ID/my-app:v1 .
```

### Configure Docker Authentication
``` 
gcloud auth configure-docker
```

### Push Image to GCP Container Registry
``` 
docker push gcr.io/YOUR_PROJECT_ID/my-app:v1
```

### Deploy Application to Kubernetes
Apply the deployment and service files:
``` 
kubectl apply -f deployment.yml
kubectl apply -f service.yml
```

### Verify Deployment and Service
``` 
kubectl get deployments
kubectl get pods
kubectl get services
```
> Wait a few minutes for the service to provision and display the external IP.

### Test Deployment

#### Verify via Terminal
``` 
curl http://external-IP-address
```

#### Verify via Browser
Navigate to:
```
http://external-IP-address
```

### Clean Up Resources

#### 1. Delete Kubernetes Resources

```
# Delete the service
kubectl delete service my-app-service

# Delete the deployment
kubectl delete deployment my-app

# Or delete using configuration files
kubectl delete -f service.yml
kubectl delete -f deployment.yml
```

#### 2. Delete GKE Cluster

# Ensure Kubernetes Engine API is enabled
gcloud services enable container.googleapis.com

# Wait a few minutes, then delete the cluster
gcloud container clusters delete <Cluster-Name> --zone <YOUR_COMPUTE_ZONE>

# Verify cluster deletion
gcloud container clusters list

#### 3. Clean Up Container Images

```
# Delete container images from Container Registry
gcloud container images delete gcr.io/YOUR_PROJECT_ID/my-app:v1 --force-delete-tags
```
# Disable the container service
```
gcloud services disable containerregistry.googleapis.com
```
#### 4. Disable DependentServices

# First, disable dependent services
gcloud services disable cloudapis.googleapis.com --force

# Then disable storage service
gcloud services disable storage.googleapis.com --force

# Finally, disable remaining services
gcloud services disable \
    container.googleapis.com \
    artifactregistry.googleapis.com \
    cloudbuild.googleapis.com \
    containerregistry.googleapis.com \
    iamcredentials.googleapis.com \
    compute.googleapis.com \
    logging.googleapis.com \
    monitoring.googleapis.com \
    --force

# Verify specific services status
echo "If services appear below, they are still ENABLED:"
gcloud services list --enabled | grep -E 'container|storage|artifact|cloudbuild|compute|logging|monitoring'

#### 5. Clean Local Resources

```
# Clean Docker images locally
docker rmi gcr.io/YOUR_PROJECT_ID/my-app:v1

# Remove kubectl context
kubectl config delete-context gke_YOUR_PROJECT_ID_YOUR_COMPUTE_ZONE_CLUSTER_NAME
```

**Note**: Make sure to replace:
- `<Cluster-Name>`
- `YOUR_PROJECT_ID`
- `YOUR_COMPUTE_ZONE`
with your actual values.

		
	
		
		
