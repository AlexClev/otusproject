# otusproject
Для обеспечения доступа к ArgoCD установливаем ingress-контролер:

kubectl apply -f https://raw.githubusercontent.com/kubernetes/ingress-nginx/controller-v1.8.2/deploy/static/provider/cloud/deploy.yaml

данный манифест так же доступен тут:kubectl apply -f https://github.com/AlexClev/otusproject/blob/main/ingress/ingress-deploy.yaml , но не применятеся по магическим причинам