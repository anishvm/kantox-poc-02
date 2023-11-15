# Hello World GitOps With ArgoCD & K8S

### About
Project contains the necessary templates and configuration files for the deployment of django hello-world application to K8S cluster. 
Continuous delivery is implemented using GitHub workflows. We're not going to discuss
the application itself in this project.

### Prerequisites
- minikube
- kubectl
- helm
- DockerHub registry

### Project Overview
- **hello-world** is the application directory and contains the Dockerfile and K8S manifests.
- **chart** directory contains the helm templates and configuration.
- **docs** directory is published via GitHub pages from gh-pages branch to be
used as Helm repo to publish helm charts for this app.
- **dev.ym & prod.yml** are manifests for ArgoCD.
- ArgoCD apps are created with autosync from repository

### Steps
- Start the minikube cluster
```
minikube start
```

- Access the K8S dashboard
```
minikube dashboard
```

- Setting up Argo CD
```
kubectl create namespace argocd
kubectl apply -n argocd -f https://raw.githubusercontent.com/argoproj/argo-cd/stable/manifests/install.yaml
```

- Access the Argo CD UI
```
kubectl -n argocd port-forward svc/argocd-server 8080:443
```

- Log in
  - Retrieve the Argo CD admin password by running the following command
```
kubectl -n argocd get secret argocd-initial-admin-secret -o jsonpath="{.data.password}" | base64 -d
```

- Deploy the app to dev environment
```
kubectl apply -f dev.yml -n argocd
```

- Deploy the app to prod environment
```
kubectl apply -f prod.yml -n argocd
```

- Accessing apps
  - Run the tunnel in a separate terminal to access externally
  - Use the external IP and port to laucnch the app  via browser
```
minkube tunnel
kubectl get svc -A
```

## GitHub Workflows
Note: Anything pushed to main branch and deployed will be tagged accordingly. We follow [Semver](https://semver.org/) semantics.
- pull_request to main branch 
  - checkout
  - build & push the docker image to the repo
  - update the K8S manifests in hello-world directory
  - push the changes to the source branch

- push to main branch 
  - checkout
  - get the latest tag value
  - build & push the docker image to the repo
  - update the tag in git 
  - install helm cli
  - checkout gh-pages branch
  - update the new image tag in values file
  - package helm charts with the new version
  - update the helm index and push

## Deployment Strategy

The applications are deployed via ArgoCD with git/helm repo. Any changes are detected and deployed automatically.
We use the rolling update strategy in K8S to make sure a zero downtime.