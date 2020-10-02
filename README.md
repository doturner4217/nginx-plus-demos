- [NGINX PLUS in K8s - Demos](#nginx-plus-in-k8s---demos)
  - [1. Clone repo](#1-clone-repo)
  - [2. Licenses](#2-licenses)
  - [3. Multinode Kubernetes cluster](#3-multinode-kubernetes-cluster)
    - [Deploy QBO Cluster](#deploy-qbo-cluster)
  - [4. Docker Registry](#4-docker-registry)
    - [Start](#start)
    - [Test](#test)
  - [5. Nginx Plus Docker Image](#5-nginx-plus-docker-image)
    - [Build](#build)
    - [Tag](#tag)
    - [Push](#push)
    - [Run in K8s](#run-in-k8s)
    - [Test](#test-1)
    - [Get NodePort](#get-nodeport)
      - [Get worker IPs](#get-worker-ips)
      - [Connect to NodePort](#connect-to-nodeport)
  - [6. Nginx Plus Ingress Docker Image](#6-nginx-plus-ingress-docker-image)
    - [Clone code](#clone-code)
    - [Checkout tag](#checkout-tag)
    - [Licenses](#licenses)
    - [Build & Tag & Push](#build--tag--push)
    - [Run](#run)
    - [Test](#test-2)
      - [Get worker IP](#get-worker-ip)
      - [Test TCP Ingress](#test-tcp-ingress)
      - [Test UDP Ingress](#test-udp-ingress)
      - [Troubleshooting](#troubleshooting)
  - [7. Multiple Ingresses](#7-multiple-ingresses)
    - [Deploy](#deploy)
    - [Classes](#classes)
    - [Annotations](#annotations)
    - [Test](#test-3)
  - [8. NJS](#8-njs)
    - [Use Case](#use-case)
    - [Deploy demo](#deploy-demo)
  - [9. Monitoring](#9-monitoring)
    - [Deploy Grafana & Prometheus](#deploy-grafana--prometheus)
    - [Add a dashboard](#add-a-dashboard)
    - [Test](#test-4)



# NGINX PLUS in K8s - Demos

## 1. Clone repo

```
git clone https://github.com/alexeadem/nginx-plus-demos.git
```

## 2. Licenses

```
cd nginx-plus-demos/licenses
```

> Copy your licenses under `licenses` and run `./validate-licenses.sh`

```
$ ./validate-licenses.sh 
Certificate will not expire
in the next 15 days
Certifcate expiration [ OK ]
notBefore=Jul 30 12:20:00 2018 GMT
notAfter=Jul 27 12:20:00 2028 GMT
subject=C = RU, ST = Moscow, L = Moscow, O = NGINX Inc, OU = Client certificates, CN = A-S00072502
Certificate pair match [ OK ]
```


## 3. Multinode Kubernetes cluster

### Deploy QBO Cluster

```
$ qbo add cluster -w3 -p5000 -d`hostname`
[2020/10/02 14:32:28:0991] N:  master.d5540.eadem.com                        ready
[2020/10/02 14:33:30:1159] N:  worker-353c5fdc.d5540.eadem.com               ready
[2020/10/02 14:34:20:8543] N:  worker-03f50753.d5540.eadem.com               ready
[2020/10/02 14:35:22:7378] N:  worker-6e9bcb57.d5540.eadem.com               ready

$ kubectl get nodes
NAME                              STATUS   ROLES    AGE     VERSION
master.d5540.eadem.com            Ready    master   4m6s    v1.18.1-dirty
worker-03f50753.d5540.eadem.com   Ready    <none>   113s    v1.18.1-dirty
worker-353c5fdc.d5540.eadem.com   Ready    <none>   2m54s   v1.18.1-dirty
worker-6e9bcb57.d5540.eadem.com   Ready    <none>   61s     v1.18.1-dirty

$ qbo get nodes
c9a6532d01e6 worker-6e9bcb57.d5540.eadem.com          172.17.0.6         eadem/node:v1.18.1        running             
0924454fe5f1 worker-03f50753.d5540.eadem.com          172.17.0.5         eadem/node:v1.18.1        running             
60e28030e3de worker-353c5fdc.d5540.eadem.com          172.17.0.4         eadem/node:v1.18.1        running             
e60d01ebc96f master.d5540.eadem.com                   172.17.0.3         eadem/node:v1.18.1        running  

```


> For more details on the QBO installation go to: https://github.com/alexeadem/qbo-ctl


## 4. Docker Registry

### Start
```
$ kubectl apply -f https://raw.githubusercontent.com/alexeadem/qbo-ctl/master/conf/registry.yaml
persistentvolumeclaim/registry-data created
persistentvolume/registry-data created
replicationcontroller/registry created
service/registry created
```

### Test

```
$ curl localhost:5000/v2/_catalog
{"repositories":[]}
```


## 5. Nginx Plus Docker Image

### Build
```
docker build -t nginx-plus:22-r1 --build-arg version=22-r1 -f nginx-plus/Dockerfile .
```
### Tag
```
docker tag nginx-plus:22-r1 localhost:5000/nginx-plus:22-r1
```
### Push
```
docker push localhost:5000/nginx-plus:22-r1
```

### Run in K8s 

```
kubectl apply -f nginx-plus/k8s
```
### Test
```
$ kubectl get pods
NAME                                   READY   STATUS    RESTARTS   AGE
nginx-plus-c7958bcd6-85dr4             1/1     Running   3          101s
nginx-plus-upstream-6778787f99-vwmrg   1/1     Running   0          101s
```
### Get NodePort
```
$ kubectl get svc
NAME                  TYPE        CLUSTER-IP       EXTERNAL-IP   PORT(S)                      AGE
kubernetes            ClusterIP   172.16.0.1       <none>        443/TCP                      12m
nginx-plus            NodePort    172.16.143.152   <none>        80:30080/TCP,443:30443/TCP   2m10s
nginx-plus-upstream   ClusterIP   None             <none>        <none>                       2m10s
```
#### Get worker IPs
```
$ qbo get nodes
c9a6532d01e6 worker-6e9bcb57.d5540.eadem.com          172.17.0.6         eadem/node:v1.18.1        running             
0924454fe5f1 worker-03f50753.d5540.eadem.com          172.17.0.5         eadem/node:v1.18.1        running             
60e28030e3de worker-353c5fdc.d5540.eadem.com          172.17.0.4         eadem/node:v1.18.1        running             
e60d01ebc96f master.d5540.eadem.com                   172.17.0.3         eadem/node:v1.18.1        running  
```
#### Connect to NodePort 
```
$ curl 172.17.0.4:30080
Status code: 200
Server address: 192.168.6.193:8096
Server name: nginx-plus-upstream-6778787f99-vwmrg
Date: 02/Oct/2020:14:45:17 $0000
User-Agent: curl/7.69.1
Cookie: 
URI: /
Request ID: 779da9d538851beb1d30bbf66db19704

```

## 6. Nginx Plus Ingress Docker Image


### Clone code
```
$ cd ..
$ git clone https://github.com/nginxinc/kubernetes-ingress.git
```
### Checkout tag
```
$ cd kubernetes-ingress
$ git checkout tags/v1.7.2
```
### Licenses
```
$ cp licenses/nginx-repo.crt licenses/nginx-repo.key ../kubernetes-ingress/
$ cd ../kubernetes-ingress/
```
### Build & Tag & Push
```
make DOCKERFILE=DockerfileForPlus VERSION=v1.7.2 PREFIX=localhost:5000/nginx-plus-ingress
```

### Run
```
$ kubectl create namespace nginx-ingress
$ kubectl apply -f nginx-plus-ingress/k8s/deployments/rbac/rbac.yaml
$ kubectl apply -f nginx-plus-ingress/k8s/deployments/common/ns-and-sa.yaml
$ kubectl apply -f nginx-plus-ingress/k8s/deployments/common/nginx-config.yaml
$ kubectl apply -f nginx-plus-ingress/k8s/deployments/common/ts-definition.yaml
$ kubectl apply -f nginx-plus-ingress/k8s/deployments/common/vs-definition.yaml
$ kubectl apply -f nginx-plus-ingress/k8s/deployments/common/vsr-definition.yaml
$ kubectl apply -f nginx-plus-ingress/k8s/deployments/common/default-server-secret.yaml
$ kubectl apply -f nginx-plus-ingress/k8s/deployments/deployment/nginx-plus-ingress.yaml
$ kubectl apply -f nginx-plus-ingress/k8s/tcp-udp/nginx-plus-config.yaml
```
### Test

```
$ kubectl get pods -n nginx-ingress  -o wide 
NAME                             READY   STATUS    RESTARTS   AGE   IP                NODE                              NOMINATED NODE   READINESS GATES
nginx-ingress-7f9d695cbd-8ctv5   1/1     Running   0          52s   192.168.150.135   worker-353c5fdc.d5540.eadem.com   <none>          
```
> Note that the the ingress is running on worker node `worker-6e9bcb57.d5540.eadem.com`. 

#### Get worker IP

```
qbo get nodes
c9a6532d01e6 worker-6e9bcb57.d5540.eadem.com          172.17.0.6         eadem/node:v1.18.1        running             
0924454fe5f1 worker-03f50753.d5540.eadem.com          172.17.0.5         eadem/node:v1.18.1        running             
60e28030e3de worker-353c5fdc.d5540.eadem.com          172.17.0.4         eadem/node:v1.18.1        running             
e60d01ebc96f master.d5540.eadem.com 
```
#### Test TCP Ingress

```

$ curl  172.17.0.3:5000/v2/_catalog
{"repositories":["nginx-plus","nginx-plus-ingress"]}

```

> Relevant Config map `nginx-plus-ingress/k8s/tcp-udp/nginx-plus-config.yaml`
```

      upstream coredns-udp {
          zone coredns-udp 64k;
          server kube-dns.kube-system.svc.cluster.local:53 resolve;
      }

      server {
          listen 53 udp;
          proxy_pass coredns-udp;
          proxy_responses 1;
          status_zone coredns-udp;
      }

```

#### Test UDP Ingress

```
$ nslookup registry.kube-system.svc.cluster.local 172.17.0.3
Server:         172.17.0.3
Address:        172.17.0.3#53

Name:   registry.kube-system.svc.cluster.local
Address: 172.16.69.112

```
> Relevant Config map `nginx-plus-ingress/k8s/tcp-udp/nginx-plus-config.yaml`

```
      upstream registry-tcp {
          zone registry-tcp 64k;
          server registry.kube-system.svc.cluster.local:5000 resolve;
      }

      server {
          listen 5000;
          proxy_pass registry-tcp;
          status_zone registry-tcp;
      } 
```

#### Troubleshooting

```
$ kubectl logs -f -lapp=nginx-ingress -n nginx-ingress

```

## 7. Multiple Ingresses 

### Deploy

### Classes

### Annotations

### Test

## 8. NJS

### Use Case 


### Deploy demo

## 9. Monitoring

### Deploy Grafana & Prometheus

### Add a dashboard

### Test