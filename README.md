# argocd-prometheus-alertmanager

## 🌟 Visão Geral
Este lab visa fornecer uma configuração para monitorar aplicações e infraestrutura em um cluster Kubernetes. Ele demonstra como:
* Usar o ArgoCD para gerenciar de forma declarativa (GitOps) a implantação do stack de monitoramento;
* Implantar o Prometheus para coletar métricas de diversos endpoints no cluster, incluindo o próprio ArgoCD e suas aplicações.

## ⚙️ Pré-requisitos
* Minikube
* ArgoCD
* Argo CLI

## 🛠️ Instalação dos pré-requisitos
### Minikube
```
curl -LO https://github.com/kubernetes/minikube/releases/latest/download/minikube-linux-amd64
sudo install minikube-linux-amd64 /usr/local/bin/minikube && rm minikube-linux-amd64 
```
#### Inicialização do cluster local
```
minikube start --nodes 2 -p argocd-prometheus-alertmanager
```
Resultado esperado:
```
kubectl get pods -A
NAMESPACE     NAME                                                     READY   STATUS    RESTARTS      AGE   IP             NODE                                 NOMINATED NODE   READINESS GATES
kube-system   coredns-66bc5c9577-qfmlw                                 1/1     Running   0             3m    10.244.0.2     argocd-prometheus-alertmanager       <none>           <none>
kube-system   etcd-argocd-prometheus-alertmanager                      1/1     Running   0             3m    192.168.67.2   argocd-prometheus-alertmanager       <none>           <none>
kube-system   kindnet-8ln8g                                            1/1     Running   0             3m    192.168.67.3   argocd-prometheus-alertmanager-m02   <none>           <none>
kube-system   kindnet-xvc5t                                            1/1     Running   0             3m    192.168.67.2   argocd-prometheus-alertmanager       <none>           <none>
kube-system   kube-apiserver-argocd-prometheus-alertmanager            1/1     Running   0             3m    192.168.67.2   argocd-prometheus-alertmanager       <none>           <none>
kube-system   kube-controller-manager-argocd-prometheus-alertmanager   1/1     Running   0             3m    192.168.67.2   argocd-prometheus-alertmanager       <none>           <none>
kube-system   kube-proxy-q8c7h                                         1/1     Running   0             3m    192.168.67.2   argocd-prometheus-alertmanager       <none>           <none>
kube-system   kube-proxy-t9vlx                                         1/1     Running   0             3m    192.168.67.3   argocd-prometheus-alertmanager-m02   <none>           <none>
kube-system   kube-scheduler-argocd-prometheus-alertmanager            1/1     Running   0             3m    192.168.67.2   argocd-prometheus-alertmanager       <none>           <none>
kube-system   storage-provisioner                                      1/1     Running   0             3m    192.168.67.2   argocd-prometheus-alertmanager       <none>           <none>
```
### ArgoCD
```
kubectl create namespace argocd
kubectl apply -n argocd -f https://raw.githubusercontent.com/argoproj/argo-cd/stable/manifests/install.yaml
```
resultado esprerado:
```
kubectl get pods -n argocd
NAMESPACE     NAME                                                     READY   STATUS    RESTARTS      AGE   IP             NODE                                 NOMINATED NODE   READINESS GATES
argocd        argocd-application-controller-0                          1/1     Running   0             1m    10.244.1.3     argocd-prometheus-alertmanager-m02   <none>           <none>
argocd        argocd-applicationset-controller-7b6ff755dc-cx84w        1/1     Running   0             1m    10.244.1.2     argocd-prometheus-alertmanager-m02   <none>           <none>
argocd        argocd-dex-server-584f7d88dc-xflft                       1/1     Running   0             1m    10.244.1.8     argocd-prometheus-alertmanager-m02   <none>           <none>
argocd        argocd-notifications-controller-67cdd486c6-fgn5r         1/1     Running   0             1m    10.244.1.7     argocd-prometheus-alertmanager-m02   <none>           <none>
argocd        argocd-redis-6dbb9f6cf4-9v4m9                            1/1     Running   0             1m    10.244.1.6     argocd-prometheus-alertmanager-m02   <none>           <none>
argocd        argocd-repo-server-57bdcb5898-t6fc2                      1/1     Running   0             1m    10.244.1.4     argocd-prometheus-alertmanager-m02   <none>           <none>
argocd        argocd-server-57d9cc9bcf-dzw2z                           1/1     Running   0             1m    10.244.1.5     argocd-prometheus-alertmanager-m02   <none>           <none>

kubectl get svc -n argocd
NAME                                      TYPE        CLUSTER-IP       EXTERNAL-IP   PORT(S)                      AGE
argocd-applicationset-controller          ClusterIP   10.100.145.156   <none>        7000/TCP,8080/TCP            19h
argocd-dex-server                         ClusterIP   10.96.64.109     <none>        5556/TCP,5557/TCP,5558/TCP   19h
argocd-metrics                            ClusterIP   10.109.33.239    <none>        8082/TCP                     19h
argocd-notifications-controller-metrics   ClusterIP   10.97.245.96     <none>        9001/TCP                     19h
argocd-redis                              ClusterIP   10.108.198.62    <none>        6379/TCP                     19h
argocd-repo-server                        ClusterIP   10.100.131.28    <none>        8081/TCP,8084/TCP            19h
argocd-server                             ClusterIP   10.109.45.175    <none>        80/TCP,443/TCP               19h
argocd-server-metrics                     ClusterIP   10.101.149.69    <none>        8083/TCP                     19h
```
Imprimir secrets (password) do usuário admin para efetuar acesso:
```
kubectl -n argo get secret argocd-initial-admin-secret -o jsonpath="{.data.password}" | base64 -d; echo
```
Efetuar port-forward para efetuar acesso ao ArgoCD GUI:
```
kubectl port-forward svc/argocd-server -n argocd 8080:443
```
Acessar a URL localhost:8080 e usar as credências a seguir:
* User: admin
* Password: secrets (informado nos passos anteriores)
> NOTA: Efetuar alteração da senha.

### Argo CLI
```
VERSION=$(curl -L -s https://raw.githubusercontent.com/argoproj/argo-cd/stable/VERSION)
curl -sSL -o argocd-linux-amd64 https://github.com/argoproj/argo-cd/releases/download/v$VERSION/argocd-linux-amd64
sudo mv argocd-linux-amd64 /usr/local/bin/argocd
sudo chmod 555 /usr/local/bin/argocd 
```
Resultado esperado:
```
argocd version
argocd: v3.2.0+66b2f30
  BuildDate: 2025-11-04T15:21:01Z
  GitCommit: 66b2f302d91a42cc151808da0eec0846bbe1062c
  GitTreeState: clean
  GoVersion: go1.25.0
  Compiler: gc
  Platform: linux/amd64
argocd-server: v3.2.0+66b2f30
  BuildDate: 2025-11-04T14:51:35Z
  GitCommit: 66b2f302d91a42cc151808da0eec0846bbe1062c
  GitTreeState: clean
  GoVersion: go1.25.0
  Compiler: gc
  Platform: linux/amd64
  Kustomize Version: v5.7.0 2025-06-28T07:00:07Z
  Helm Version: v3.18.4+gd80839c
  Kubectl Version: v0.34.0
  Jsonnet Version: v0.21.0
```
Efetuar login no cluster:
```
kubectl port-forward svc/argocd-server -n argocd 8080:443
argocd login localhost:8080
```
## 📦 Estrutura do repositório
```
├── appset/                           # Manifestos do ArgoCD (ApplicationSet) para a implantação
│   ├── prometheus-cdrs.yaml          # Configurações para CRDs do Prometheus
│   └── prometheus.yaml               # Configurações do Prometheus
├── values/                           # Diretório de definições do prometheus
│   ├── core-values/                  # Diretório dos values comum do prometheus para todos os clusteres
│   |   └── default-prometheus.yaml   # Values do prometheus
│   └── custom-values/                # Diretório de definições específicas por cluster do prometheus
│       ├── rules/                    # Diretório de definições regras / grupos de alertas
│       └── in-cluster.yaml           # Values de definições específicas por cluster do prometheus
└── README.md
```
## 🚀 Como Usar
Clonar o Repositório
```
git clone https://github.com/douglasgrande/argocd-prometheus-alertmanager.git
cd argocd-prometheus-alertmanager
```
Criar as ApplicationSets
```
# 1
kubectl apply -f prometheus-cdrs.yaml

# 2
kubectl apply -f prometheus.yaml
```
Resultado esperado:
Via ArgoCD CLI
```
argocd appset list
NAME                         PROJECT  SYNCPOLICY  CONDITIONS                                                                                                                                                                                                                                                             REPO                                                            PATH                             TARGET
argocd/lab-prometheus-crds   default  nil         [{ParametersGenerated Successfully generated parameters for all Applications 2025-11-26 14:28:48 -0300 -03 True ParametersGenerated} {ResourcesUpToDate All applications have been generated successfully 2025-11-26 14:28:48 -0300 -03 True ApplicationSetUpToDate}]  https://github.com/prometheus-operator/prometheus-operator.git  example/prometheus-operator-crd  v0.70.0
argocd/lab-prometheus-stack  default  nil         [{ParametersGenerated Successfully generated parameters for all Applications 2025-11-27 08:56:33 -0300 -03 True ParametersGenerated} {ResourcesUpToDate All applications have been generated successfully 2025-11-27 08:56:33 -0300 -03 True ApplicationSetUpToDate}]  https://prometheus-community.github.io/helm-charts                                               55.7.0

argocd app list
NAME                                CLUSTER                         NAMESPACE   PROJECT  STATUS  HEALTH   SYNCPOLICY  CONDITIONS  REPO                                                            PATH                             TARGET
argocd/in-cluster-prometheus-crds   https://kubernetes.default.svc  monitoring  default  Synced  Healthy  Auto-Prune  <none>      https://github.com/prometheus-operator/prometheus-operator.git  example/prometheus-operator-crd  v0.70.0
argocd/in-cluster-prometheus-stack  https://kubernetes.default.svc  monitoring  default  Synced  Healthy  Auto-Prune  <none>      https://prometheus-community.github.io/helm-charts                                               55.7.0
```
Via ArgoCD GUI
<img width="919" height="373" alt="image" src="https://github.com/user-attachments/assets/0ec65907-4c74-4148-86ca-308b67187d82" />
> NOTA: Atualmente não é possível de forma geral visualizar ApplicationSets via ArgoCD GUI.
