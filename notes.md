# Training Guide

## Kubernetes

### Pod

``` bash
kc apply -f pod.yaml
kc logs myapp-pod
kc get po -w
kc delete po myapp-pod
```

### ReplicaSet


``` bash
kc apply -f replicaset.yaml
kc get rs
kc get po
```

- show that the pods are running on different nodes
- show that updates to the replicaset don't actually propagate until a pod is deleted (change image)
- delete the replicaset and show pods delete

``` bash
kc delete rs/frontend
kc get po
```

### Deployment

``` bash
kc create deploy nginx --image=nginx:1.14-alpine
kc get deploy
kc get po
kc delete deploy/nginx
kc apply -f deployment.yaml
kc get deploy
kc get po
```

- describe pod, look at the image
- scale deployment

``` bash
kc scale deploy/nginx --replicas=3
kc rollout status deploy/nginx
kc get deploy
kc get po
```

- upgrade with bad image

``` bash
kc set image deploy/nginx nginx=nginx:1.15-alpne --record
kc rollout status deploy/nginx
kc get po
kc rollout undo deploy/nginx
```

- redo upgrade from manifest

``` bash
kc apply -f deployment.yaml
```

### Services

``` bash
kc expose deploy/nginx --type=NodePort
kc get service
```

- go look at it

``` bash
curl -I rancher01:<port>
```

- point out that expose only knows what port to use b/c it's in the deployment
- comment out the port and redeploy the deployment
- `kubectl expose` and watch it fail
- specify the port

``` bash
kc expose deploy/nginx --type=NodePort --port=80
```

- nuke it again
- what if we want to specify the nodeport?
- create with `kubectl create`

``` bash
kc delete service/nginx
kc create service nodeport nginx --node-port=31337 --tcp=80:80
curl -I rancher01:31337
```

- this is all cool, but we don't want to manage ports. unfortunately my house isn't an Amazon region, so I can't show LoadBalancer, but for non-cloud deployments there _is_ MetalLB (show in browser).
- instead let's redeploy the service as ClusterIP and put an ingress in front of it

``` bash
kc delete service/nginx
kc expose service/nginx
```

- show the YAML
- nuke the service
- redeploy it from the YAML

``` bash
kc apply -f service.yaml
```

### Ingress

- show `ingress-single.yaml`

``` bash
kc apply -f ingress-single.yaml
kc get ingress
```

- visit <https://training.cl.monach.us>
- what about multiple apps?

``` bash
kc delete ingress/single-ingress
kc apply -f ingress-fanout.yaml
```

- visit <https://training.cl.monach.us> (works)
- visit <https://training.cl.monach.us/rancher> (fail)

```
kc create deploy hello-world --image rancher/hello-world
kc expose deploy/hello-world (will fail)
kc expose deploy/hello-world --port=80
```

- visit <https://training.cl.monach.us/rancher> (issues)
- why issues? (show console w/ 404 and then `kc get ingress/fanout-ingress -o yaml` and discuss)
- fix yaml and apply
- refresh browser and show that it works

### ConfigMap

- we're going to use the k8s configmap tutorial at <https://kubernetes.io/docs/tutorials/configuration/>

``` bash
cd configmap
kc create configmap redis-config --from-file=redis-config
kc get configmap redis-config -o yaml
kc apply -f pod.yaml
kc exec -it redis redis-cli
CONFIG GET maxmemory
CONFIG GET maxmemory-policy
```

- how does this work?
- shell into the pod and show `/redis-master/redis.conf`
- show that it updates automatically, although detecting the update is left to the application

- talk about secrets
- show borgmatic config that uses configmap

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

- deploy `superseb/rancher-demo` as a workload
    - expose port 8080
- put an Ingress in front of it
    - use `training.cl.monach.us`
- scale it
cat i
