# Kubernetes na Digital Ocean: Gerenciando aplicações conteinerizadas

## Preparando o ambiente
```
$ sudo apt-get update && sudo apt-get install -y apt-transport-https gnupg2 curl
$ curl -s https://packages.cloud.google.com/apt/doc/apt-key.gpg | sudo apt-key add -
$ echo "deb https://apt.kubernetes.io/ kubernetes-xenial main" | sudo tee -a /etc/apt/sources.list.d/kubernetes.list
$ sudo apt-get update
$ sudo apt-get install -y kubectl
$ sudo snap install doctl
$ doctl auth init
$ sudo snap connect doctl:kube-config
$ doctl kubernetes cluster kubeconfig save cluster1
$ doctl kubernetes cluster list
$ doctl kubernetes cluster node-pool list cluster1
$ kubectl cluster-info
$ kubectl get nodes
$ kubectl get services
```

## 1-Click Apps (Marketplace --> WordPress Kubernetes)
```
$ kubectl get pods -A
$ kubectl get pods -n wordpress
$ kubectl get services -A
$ kubectl get services -n wordpress
$ kubectl get deploy -n wordpress
$ kubectl describe deploy wordpress -n wordpress | more
$ kubectl scale --current-replicas=1 --replicas=3 deployment/wordpress -n wordpress
$ kubectl get pods -n wordpress
```

## Monitorando o cluster (Marketplace --> Kubernetes Metrics Server)
```
$ sudo apt install apache2-utils
$ ab --help
$ ab -n 5000 -c 1000 http://206.189.253.245/
$ kubectl top -h
$ kubectl top node
$ kubectl top pod -n wordpress
$ kubectl top pod -A
$ kubectl get services -A
```

## Auto Scaling
```
$ doctl kubernetes cluster node-pool update cluster1 pool-cluster1 --min-nodes=3 --max-nodes=6
$ kubectl get nodes
$ kubectl apply -f .k8s/hello.yaml
$ kubectl get pods
$ kubectl apply -f .k8s/carga.yaml
$ kubectl top nodes
$ kubectl get pods
$ kubectl scale --replicas=3 -f .k8s/carga.yaml
$ kubectl delete -f .k8s/hello.yaml
$ kubectl delete -f .k8s/carga.yaml
```

## Preparando a infraestrutura da nossa aplicação
[Fonte](https://github.com/ricardomerces/airportnames)
### Como executar:
```
$ docker run --name airportnames -p 8080:8080 rmerces/airportnames
```
### Consulta:
iataCode --> código iata dos aeroportos
```
http://localhost:8080/airportName?iataCode=SDU
```

## Deploy da aplicação
```
$ kubectl create namespace airportnames
$ kubectl get namespaces
$ kubectl get pods -A
$ kubectl top node -n airportnames
$ kubectl top pod -n airportnames
$ kubectl apply -f .k8s/deployment.yaml
$ kubectl top pod -A
$ kubectl get pods
$ kubectl delete -f .k8s/deployment.yaml
$ kubectl apply -f .k8s/deployment.yaml --namespace=airportnames
$ kubectl get pods -A
$ kubectl top pod -n airportnames
```

## Criando o Load Balancer
```
$ kubectl apply -f .k8s/service.yaml --namespace=airportnames
$ kubectl get services -A
$ kubectl get services -n airportnames
```
### Consulta:
iataCode --> código iata dos aeroportos
```
http://104.248.109.188/airportName?iataCode=SDU
```

## HPA: Horizontal Pod Autoscaler
```
$ doctl kubernetes cluster node-pool update cluster1 pool-cluster1 --auto-scale=true --min-nodes=2 --max-nodes=5
$ kubectl apply -f .k8s/hpa/hpa.yaml --namespace=airportnames
$ kubectl get hpa -A
$ kubectl get hpa -n airportnames
$ kubectl apply -f .k8s/hpa/deployment.yaml --namespace=airportnames
$ kubectl get pods -A
$ kubectl describe pod airportnames-7f6478875f-9zl2x -n airportnames
$ kubectl get hpa -A
$ kubectl apply -f .k8s/hpa/carga.yaml --namespace=airportnames
$ kubectl scale --replicas=5 -f .k8s/hpa/carga.yaml --namespace=airportnames
$ kubectl get hpa -A
NAMESPACE      NAME           REFERENCE                 TARGETS    MINPODS   MAXPODS   REPLICAS   AGE
airportnames   airportnames   Deployment/airportnames   100%/80%   2         40        30         3h5m
$ kubectl top nodes
NAME                  CPU(cores)   CPU%   MEMORY(bytes)   MEMORY%   
pool-cluster1-3d4cc   593m         59%    1111Mi          70%       
pool-cluster1-3d4cf   702m         70%    1074Mi          68%       
pool-cluster1-3d4cu   649m         64%    1128Mi          71%       
pool-cluster1-3dhb7   785m         78%    993Mi           63%       
pool-cluster1-3dhbc   187m         18%    960Mi           61%
$ kubectl delete -f .k8s/hpa/carga.yaml --namespace=airportnames
$ kubectl get pods -n airportnames
$ doctl kubernetes cluster node-pool update cluster1 pool-cluster1 --auto-scale=false --count=2
```

## doctl autocomplete
```
$ cd ~
$ echo "source <(doctl completion bash)" >> .profile
$ cat .profile
$ source <(doctl completion bash)
```

## Documentação
- [Install and Set Up kubectl](https://kubernetes.io/docs/tasks/tools/install-kubectl/)
- [How to Install and Configure doctl](https://www.digitalocean.com/docs/apis-clis/doctl/how-to/install/)
- [kubectl Cheat Sheet](https://kubernetes.io/pt/docs/reference/kubectl/cheatsheet/)
- [doctl](https://github.com/digitalocean/doctl)
