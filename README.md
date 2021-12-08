# CICD-with-Github-Actions-into-GKE
CICD with Github Actions to automate deployment into GKE. This demo repo will guide with simple steps to automate quarkus workload with the help of Maven

## POC:
<li>
  Enable CICD:  Deploy the quarkus workload into GKE when code comitted
 </li>
<li>
  Build:  Maven is used to build and push the image into GCR repo
</li>
<li>
  Deploy:  Quakus Kubernetes extention generates service/deployment manifes which will be used to deploy into Kubernetes cluster. Istio Ingress gateway enabled to access our workload
</li>


## Prerequisite

<li>
Kubernetes Cluster (tested on v1.21.5-gke.1302)
 </li>
 <li>
Istio Instalation (tested on istio-1.12.0)
</li>
<li>
Kiali Dashboard
</li>

## Steps

1) Create Service Account in Google Cloud Platform

    a) Goto 'IAM & Admin' --> 'Service Accounts'
    
    b) create a service account with all the necessary permissions
    
    c) Create new key for the service account created(download the json key file in local machine)
    
    d) endode the key using base64, this encoded key should be updated in GitGub secrets
    
2) Create secrets in your git repo

    a) Goto 'Settings' --> secrets
    
    b) create secrets for the below
    
         GKE_PROJECT={your google project ID}
         GKE_SA_KEY={encoded service account key, refer step 1)d)}

2) Enable actions with Github Actions

    a) Goto 'Actions' of your repo
    
    b) Select 'Build and Deploy to GKE' and click 'Set up this Workflow'
    
    c) Rename .yml file (example CICD.yml)
    
~~~ 
name: Build and Deploy to GKE

on:
  push:
    branches:
      - main

env:
  PROJECT_ID: ${{ secrets.GKE_PROJECT }}
  GKE_CLUSTER: cluster-1    # TODO: update to cluster name
  GKE_ZONE: us-central1-c   # TODO: update to cluster zone
  DEPLOYMENT_NAME: gke-test # TODO: update to deployment name
  IMAGE: static-site

jobs:
  setup-build-publish-deploy:
    name: Setup, Build, Publish, and Deploy
    runs-on: ubuntu-latest
    environment: production

    steps:
    - name: Checkout
      uses: actions/checkout@v2

    # Setup gcloud CLI
    - uses: google-github-actions/setup-gcloud@v0.2.0
      with:
        service_account_key: ${{ secrets.GKE_SA_KEY }}
        project_id: ${{ secrets.GKE_PROJECT }}

    # Configure Docker to use the gcloud command-line tool as a credential
    # helper for authentication
    - run: |-
        gcloud --quiet auth configure-docker
    # Get the GKE credentials so we can deploy to the cluster
    - uses: google-github-actions/get-gke-credentials@v0.2.1
      with:
        cluster_name: ${{ env.GKE_CLUSTER }}
        location: ${{ env.GKE_ZONE }}
        credentials: ${{ secrets.GKE_SA_KEY }}

    # Set up kustomize
    - name: Set up Kustomize
      run: |-
        curl -sfLo kustomize https://github.com/kubernetes-sigs/kustomize/releases/download/v3.1.0/kustomize_3.1.0_linux_amd64
        chmod u+x ./kustomize
        
    # Build and Push the Docker image to Google Container Registry
    - name: Build with Maven
      run: mvn -B clean package -Dnative -Dquarkus.container-image.push=true --file pom.xml

    # Deploy the Docker image to the GKE cluster
    - name: Deploy
      run: |-
        kubectl apply -f yamls/service-account.yaml
        kubectl apply -f yamls/cicd-demo-svc.yaml
        
        kubectl apply -f target/kubernetes/kubernetes.yml
        
        kubectl apply -f yamls/cicd-gateway.yaml
        
        kubectl apply -f yamls/cicd-virtualservice.yaml
~~~ 

**Note:** If all the above steps are correct, whenever code commited, this repo will be built, native image pushed into GCR and workload will be deployed into kubernetes cluster along with Istio Ingress gateway.
        
        
