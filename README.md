# otusproject
Для обеспечения доступа к ArgoCD установливаем ingress-контролер:

kubectl apply -f https://raw.githubusercontent.com/kubernetes/ingress-nginx/controller-v1.8.2/deploy/static/provider/cloud/deploy.yaml

данный манифест так же доступен тут:kubectl apply -f https://raw.githubusercontent.com/AlexClev/otusproject/main/ingress/ingress-deploy.yaml 
Создаём пространство для ArgoCD 
 kubectl create namespace argocd

Устанавливаем ArgoCD
kubectl apply -n argocd -f https://raw.githubusercontent.com/argoproj/argo-cd/stable/manifests/install.yaml

 https://raw.githubusercontent.com/AlexClev/otusproject/main/argocd/argocd-deploy.yaml 

Подключаемся по SSH на удаленную машину и  ставим agro-CLI

