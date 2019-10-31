# Training Guide

## Kubernetes

### Pod

``` bash
kubectl apply -f pod.yaml
kubectl logs myapp-pod
kubectl get po -w
kubectl delete po myapp-pod
```

### Deployment

- we can launch random stuff, but this isn't repeatable

``` bash
kubectl create deploy nginx --image=nginx:1.16-alpine
kubectl get deploy
kubectl get po
kubectl delete deploy/nginx
```

- launch again using kustomize templates

```
kubectl create deploy nginx --image=nginx:1.16-alpine --dry-run -o yaml > deployment/base/deployment.yaml
kubectl apply -k deployment/base
kubectl get deploy
kubectl get po
```

- describe pod, look at the image
- scale deployment manually

``` bash
kubectl scale deploy/nginx --replicas=3
kubectl rollout status deploy/nginx
kubectl get deploy
kubectl get po
```

- upgrade with bad image

``` bash
kubectl set image deploy/nginx nginx=nginx:1.17-alpne --record
kubectl rollout status deploy/nginx
kubectl get po
kubectl rollout undo deploy/nginx
```

- redo upgrade from manifest

``` bash
kustomize build deployment/base
```

- edit base to change image and then apply

```bash
kubectl apply -k deployment/base
```

- how can we use this for different environments?

```bash
kustomize build deployment/overlay/staging
kustomize build deployment/overlay/production

kubectl apply -k deployment/overlay/staging
kubectl apply -k deployment/overlay/production
kubectl get deploy
kubectl get pods
```

### Services

- show services listening as NodePort
- go look at them

``` bash
curl -I rancher01:<port>
```

### Ingress

- show `deployment/overlay/ingress/single/ingress.yaml`

``` bash
kubectl apply -k deployment/overlay/ingress/single
kubectl get ingress
```

- visit <https://training-a.cl.monach.us>
- what about multiple apps?

``` bash
kustomize build deployment/overlay/ingress/fanout
kubectl apply -k deployment/overlay/ingress/fanout 
```

- visit <https://training-a.cl.monach.us> (fail)
- visit <https://training.cl.monach.us/nginx> (works)
- deploy rancher-demo application

```bash
kustomize build demo
kubectl apply -k demo
```

- visit <https://training-a.cl.monach.us/>

## Rancher

### Server Deploy

```
docker run -d --restart=unless-stopped -p 80:80 -p 443:443 -v /opt/rancher:/var/lib/rancher rancher/rancher:v2.1.2
```

### Node Deploy

- Deploy a custom cluster with one node in all three roles
- Watch it deploy and then show it in the clusters view

### Rancher Server Walkthrough

- Clusters
- Authentication & Security
- Storage
- Projects
- Namespaces
- Catalogs
- CLI/API/Kubectl

### Application Deployment

- deploy `monachus/rancher-demo` as a workload
    - expose port 8080
- put an Ingress in front of it
    - use `training-a.cl.monach.us`
- scale it
