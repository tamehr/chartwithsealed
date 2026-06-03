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

kubectl get pods -n argocd

kubectl -n argocd get secret argocd-initial-admin-secret \
-o jsonpath="{.data.password}" | base64 -d

kubectl port-forward --address 0.0.0.0 svc/argocd-server -n argocd 8080:443

https://0.0.0.0:8080

admin
