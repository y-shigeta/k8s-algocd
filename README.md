# algoCD

how to set up algo CD
- [reference](https://blog.cybozu.io/entry/2019/11/21/100000)
- [how to add external k8s](https://blog.mosuke.tech/entry/2022/02/08/argocd-deploy-to-external-cluster/)
- [github actions with algocd](https://igboie.medium.com/kubernetes-ci-cd-with-github-github-actions-and-argo-cd-36b88b6bda64)
- [sample](https://qiita.com/sourjp/items/4547bc81c12286007553)
- [algocd officail](https://argoproj.github.io/argo-rollouts/installation/)
- [microk8s istio/algo](https://korattablog.com/kubernetes/)
- [algocd notification](https://techstep.hatenablog.com/entry/2020/10/28/131742)

## overview

- overall

```
1. set up github actions
2. create service account for github and gcr
3. Set secret for service account on  github actions
4. create manifest repository
5. create GKE
6. install algo CD

```

## Github actions
- create dockerfile and application files
mkdir -p .github/workflow/
cp -p ../github-actions/push-container-gcr.yml .github/workflow/push-container-gcr.yml
git push origin

- Create service account for gcr

- Set secret on github
cat {ダウンロードした鍵のパスを指定} | base64


## algocd
- install algocd

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

- after login, please application as application.yaml
kubectl apply -f application-kustomize-dev.yaml
kubectl apply -f application-kustomize-prd.yaml

### algoCD rollouts
- install argo-rollouts
kubectl create namespace argo-rollouts
kubectl apply -n argo-rollouts -f https://raw.githubusercontent.com/argoproj/argo-rollouts/stable/manifests/install.yaml

- kubectl plugin
curl -LO https://github.com/argoproj/argo-rollouts/releases/latest/download/kubectl-argo-rollouts-darwin-amd64
chmod +x ./kubectl-argo-rollouts-darwin-amd64
sudo mv ./kubectl-argo-rollouts-darwin-amd64 /usr/local/bin/kubectl-argo-rollouts
kubectl argo rollouts version
source <(kubectl-argo-rollouts completion bash)

- run sample rollouts
kubectl apply -f https://raw.githubusercontent.com/argoproj/argo-rollouts/master/docs/getting-started/basic/rollout.yaml
kubectl apply -f https://raw.githubusercontent.com/argoproj/argo-rollouts/master/docs/getting-started/basic/service.yaml

- check rollout deployment
kubectl get rollouts

- watch rollout status
kubectl argo rollouts get rollout sample-portal-nginx --watch

- update container image to httpd
kubectl argo rollouts set image portal-nginx nginx=httpd

- promote to new version
kubectl argo rollouts promote portal-nginx

- abort rollout
kubectl argo rollouts abort portal-nginx

- show dashboard
kubectl argo rollouts dashboard

### algo Notification
https://techstep.hatenablog.com/entry/2020/10/28/131742

## Todo
- blue/green deployment
- canary deployment
- github actions for CI
- deploy frontend/backend

## Cleanup

kubectl delete -n argocd -f https://raw.githubusercontent.com/argoproj/argo-cd/stable/manifests/install.yaml
kubectl delete -n argo-rollouts -f https://raw.githubusercontent.com/argoproj/argo-rollouts/stable/manifests/install.yaml
kubectl delete -n argocd -f https://raw.githubusercontent.com/argoproj/argo-rollouts/stable/manifests/install.yaml

kubectl delete -f https://raw.githubusercontent.com/argoproj/argo-rollouts/master/docs/getting-started/basic/rollout.yaml
kubectl delete -f https://raw.githubusercontent.com/argoproj/argo-rollouts/master/docs/getting-started/basic/service.yaml
