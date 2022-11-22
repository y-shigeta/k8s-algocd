# algoCD 
how to set up algo CD

## install
- overall
```
1. set up github actions
2. create service account for github and gcr
3. Set secret for service account on  github actions
4. create manifest repository
5. create GKE
6. install algo CD

```

- 
```
kubectl create namespace argocd
kubectl apply -n argocd -f https://raw.githubusercontent.com/argoproj/argo-cd/stable/manifests/install.yaml
(for ha cluster)kubectl apply -n argocd -f https://raw.githubusercontent.com/argoproj/argo-cd/v1.0.1/manifests/ha/install.yaml
kubectl patch svc argocd-server -n argocd -p '{"spec": {"type": "LoadBalancer"}}'

- get global ip for algoCD
GIP=`kubectl get svc -n argocd | grep LoadBalancer | awk '{print $4}'`
echo $GIP

- get admin password. algo cd helm search hubpod name is password 
PWD=`kubectl -n argocd get secret argocd-initial-admin-secret -o jsonpath="{.data.password}" | base64 -d`
echo $PWD

```
