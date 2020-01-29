# Intro til Kubernetes

## Prereq

* Docker - startes opp - https://docs.docker.com/docker-for-windows/
* Gcloud SDK/CLI (for dette eksemplet) - https://cloud.google.com/sdk/docs/quickstart-linux

## Innledende oppsett av cluster

1. https://console.cloud.google.com/kubernetes - `gcloud config set compute/zone europe-north1`
2. Opprett cluster - evt. 
```
gcloud beta container --project "poetic-glass-266311" clusters create "demo" --zone "europe-north1-a" --no-enable-basic-auth --cluster-version "1.13.11-gke.23" --machine-type "n1-standard-1" --image-type "COS" --disk-type "pd-standard" --disk-size "100" --scopes "https://www.googleapis.com/auth/devstorage.read_only","https://www.googleapis.com/auth/logging.write","https://www.googleapis.com/auth/monitoring","https://www.googleapis.com/auth/servicecontrol","https://www.googleapis.com/auth/service.management.readonly","https://www.googleapis.com/auth/trace.append" --num-nodes "3" --enable-cloud-logging --enable-cloud-monitoring --enable-ip-alias --network "projects/poetic-glass-266311/global/networks/default" --subnetwork "projects/poetic-glass-266311/regions/europe-north1/subnetworks/default" --default-max-pods-per-node "110" --addons HorizontalPodAutoscaling,HttpLoadBalancing --enable-autoupgrade --enable-autorepair  --enable-autoscaling
```

```
gcloud beta container --project "poetic-glass-266311" clusters create "demo" --zone "europe-north1-a" --no-enable-basic-auth --cluster-version "1.13.11-gke.23" --machine-type "n1-standard-1" --image-type "COS" --disk-type "pd-standard" --disk-size "100" --scopes "https://www.googleapis.com/auth/devstorage.read_only","https://www.googleapis.com/auth/logging.write","https://www.googleapis.com/auth/monitoring","https://www.googleapis.com/auth/servicecontrol","https://www.googleapis.com/auth/service.management.readonly","https://www.googleapis.com/auth/trace.append" --enable-cloud-logging --enable-cloud-monitoring --enable-ip-alias --network "projects/poetic-glass-266311/global/networks/default" --subnetwork "projects/poetic-glass-266311/regions/europe-north1/subnetworks/default" --default-max-pods-per-node "110" --addons HorizontalPodAutoscaling,HttpLoadBalancing --enable-autoupgrade --enable-autorepair  --enable-autoscaling --min-nodes 1 --max-nodes 8 --num-nodes 2
```
5. Opprett - tar ca. 2 minutter
6. G† innom Monitoring for † starte init der

## Examples app

```ssh
git clone https://github.com/GoogleCloudPlatform/kubernetes-engine-samples
cd kubernetes-engine-samples/hello-app
docker build -t gcr.io/poetic-glass-266311/hello-app:v1 .
gcloud auth configure-docker
docker push gcr.io/poetic-glass-266311/hello-app:v1
kubectl create deployment hello-web --image=gcr.io/poetic-glass-266311/hello-app:v1
kubectl expose deployment hello-web --type=LoadBalancer --port 80 --target-port 8080
```

## Example ingress with scaling

```ssh
kubectl apply -f web-deployment.yaml
kubectl apply -f ingress.yaml
kubectl get pods -o wide
# kubectl scale deployments/web --replicas=4
# kubectl get pods -o wide
# kubectl scale deployments/web --replicas=2
# kubectl get pods -o wide
```

## Static IP

```ssh
gcloud compute addresses create web-static-ip --global
kubectl delete -f ingress.yaml
kubectl apply -f ingress-with-static-ip.yaml
```

## Autoscale

```ssh
kubectl run php-apache --image=gcr.io/google\_containers/hpa-example --requests=cpu=500m,memory=500M --expose --port=80 
kubectl run -i --tty service-test --image=busybox /bin/sh  
wget -q -O- http://php-apache.default.svc.cluster.local

kubectl autoscale deployment php-apache --cpu-percent=50 --min=1 --max=10
kubectl get hpa

kubectl run -i --tty load-generator --image=busybox /bin/sh
while true; do wget -q -O- http://php-apache.default.svc.cluster.local; done

kubectl get hpa
kubectl get deployment php-apache
kubectl describe pod
kubectl get pods
```
