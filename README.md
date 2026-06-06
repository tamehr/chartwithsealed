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
        
#install argo-cd       

kubectl create namespace argocd       

kubectl apply -n argocd -f https://raw.githubusercontent.com/argoproj/argo-cd/stable/manifests/install.yaml

kubectl wait deploy -n argocd --all --for condition=Available --timeout=2m
-----------------------
problem ui argo-cd not load in killerkoda 

#solution 1:argo without tls for run in killerkoda

kubectl -n argocd patch configmap argocd-cmd-params-cm \
  --type merge \
  -p '{"data":{"server.insecure":"true"}}'

kubectl rollout restart deployment argocd-server -n argocd

kubectl port-forward --address 0.0.0.0 -n argocd svc/argocd-server 8080:80

kubectl get svc argocd-server -n argocd
--------------------------------------

#solution 2: node port for whithowt tls

kubectl patch svc argocd-server -n argocd -p '{"spec":{"type":"NodePort"}}'

kubectl get svc argocd-server -n argocd
-----------------------------------------------
for kustomize with helm should --enable-helm in setting argo-cd set

# Patch the argocd-cm ConfigMap to add --enable-helm
kubectl patch configmap argocd-cm -n argocd --type merge \
  -p '{"data":{"kustomize.buildOptions":"--enable-helm"}}'

# Restart the repo server to pick up the change
kubectl rollout restart deployment argocd-repo-server -n argocd
# Wait for it to be ready
kubectl rollout status deployment argocd-repo-server -n argocd --timeout=60s
# Verify the configuration
kubectl get configmap argocd-cm -n argocd \
  -o jsonpath='{.data.kustomize\.buildOptions}'
# Output: --enable-helm

#restart repo-server after Patch 
kubectl rollout status deployment/argocd-repo-server -n argocd

kubectl get pods -n argocd | grep repo

----------------------------------------------

kubectl get pods -n argocd

kubectl annotate app nginx-stage -n argocd argocd.argoproj.io/refresh=hard --overwrite

kubectl describe app nginx-stage -n argocd | tail -n 50

kubectl get pods -n main
#secret
kubectl get secret nginx-secret -n main

kubectl describe secret nginx-secret -n main

#view real secret
kubectl get secret nginx-secret -n main -o yaml
----------------
user argo: admin
pass:
kubectl -n argocd get secret argocd-initial-admin-secret \
-o jsonpath="{.data.password}" | base64 -d
--------------------------------------
#for argocd ui with tls 

kubectl port-forward --address 0.0.0.0 svc/argocd-server -n argocd 8080:443

------------------------------------------
#for view real secret 
kubectl get secret nginx-secret -n main \
-o jsonpath='{.data.password}' | base64 -d
--------------------------
5. ترتیب اعمال Secret و Helm

Kustomize همه چیز را با هم رندر می‌کند، اما اگر Deployment داخل Chart به Secret وابسته باشد، معمولاً مشکلی پیش نمی‌آید چون Kubernetes همه Manifestها را در یک Sync دریافت می‌کند.

برای مطمئن‌تر بودن می‌توان از Sync Wave  روی secret استفاده کرد تا قبل از helm اعمال شود :

metadata:
  annotations:
    argocd.argoproj.io/sync-wave: "-1"
---------------------------------------------



