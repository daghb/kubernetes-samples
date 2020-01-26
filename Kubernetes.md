# Intro til Kubernetes

## Prereq

* Docker - startes opp - https://docs.docker.com/docker-for-windows/
* Gcloud CLI (for dette eksemplet) - https://cloud.google.com/sdk/gcloud/

## Innlendende oppsett av cluster

1. https://console.cloud.google.com/kubernetes
2. Opprett cluster
3. Husk Zone -> North 1A, v1.15.7
4. Minimum 3 noder
5. Opprett - tar ca. 2 minutter
6. G† innom Monitoring for † starte init der

## Examples app

```powershell
git clone https://github.com/GoogleCloudPlatform/kubernetes-engine-samples
cd kubernetes-engine-samples/hello-app
docker build -t gcr.io/poetic-glass-266311/hello-app:v1 .
gcloud auth configure-docker
docker push gcr.io/poetic-glass-266311/hello-app:v1
kubectl create deployment hello-web --image=gcr.io/poetic-glass-266311/hello-app:v1
kubectl expose deployment hello-web --type=LoadBalancer --port 80 --target-port 8080
```

## Example ingress

```powershell
kubectl apply -f web-deployment.yaml
kubectl apply -f ingress.yaml
```
