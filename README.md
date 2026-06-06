repo
│
├─ charts
│   └─ nginx
│       ├─ Chart.yaml
│       ├─ values.yaml
│       └─ templates
│           └─ deployment.yaml
│
├─ secrets
│   └─ nginx-secret.yaml
│
└─ envs
    ├─ stage
    │   ├─ kustomization.yaml
    │   └─ values.yaml
    │
    └─ main
        ├─ kustomization.yaml
        └─ values.yaml
        
kubectl create namespace argocd       

kubectl apply -n argocd -f https://raw.githubusercontent.com/argoproj/argo-cd/stable/manifests/install.yaml

kubectl wait deploy -n argocd --all --for condition=Available --timeout=2m
-----------------------
argo بدون tls برای اجرا در killerkoda

kubectl -n argocd patch configmap argocd-cmd-params-cm \
  --type merge \
  -p '{"data":{"server.insecure":"true"}}'

kubectl rollout restart deployment argocd-server -n argocd

kubectl port-forward --address 0.0.0.0 -n argocd svc/argocd-server 8080:80
--------------------------------------

راه 2: NodePort
kubectl patch svc argocd-server -n argocd -p '{"spec":{"type":"NodePort"}}'

kubectl get svc argocd-server -n argocd
-----------------------------------------------
kubectl get pods -n argocd

kubectl annotate app nginx-stage -n argocd argocd.argoproj.io/refresh=hard --overwrite

kubectl describe app nginx-stage -n argocd | tail -n 20

kubectl -n argocd get secret argocd-initial-admin-secret \
-o jsonpath="{.data.password}" | base64 -d

kubectl port-forward --address 0.0.0.0 svc/argocd-server -n argocd 8080:443

https://0.0.0.0:8080

admin
