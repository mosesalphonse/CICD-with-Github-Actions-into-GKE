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
2) create secrets in your git repo
    a) Goto 'Settings' --> secrets
    b) create secrets for the below
         GKE_PROJECT={your google project ID}
         GKE_SA_KEY={encoded service account key, refer step 1)d)}
