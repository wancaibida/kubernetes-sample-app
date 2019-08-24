# Sample Kubernetes Web App

## 环境配置

### 连接cluster 

```
gcloud container clusters get-credentials CLUSER_NAME --zone us-central1-a --project PROJECT_NAME
```

### 安装helm和tiller
```
kubectl create serviceaccount --namespace kube-system tiller
kubectl create clusterrolebinding tiller-cluster-rule --clusterrole=cluster-admin --serviceaccount=kube-system:tiller
helm init --service-account tiller
```

### 安装nginx-ingress

```
helm install --name nginx-ingress stable/nginx-ingress --set rbac.create=true --set controller.publishService.enabled=true
```

### 安装let's encrypt

```
kubectl apply -f https://raw.githubusercontent.com/jetstack/cert-manager/release-0.8/deploy/manifests/00-crds.yaml
kubectl create -f issuer.yaml
```

[issuer.yaml](https://raw.githubusercontent.com/wancaibida/kubernetes-sample-app/master/issuer.yaml)
```
apiVersion: certmanager.k8s.io/v1alpha1
kind: ClusterIssuer
metadata:
  name: letsencrypt-prod
spec:
  acme:
    # The ACME server URL
    server: https://acme-v02.api.letsencrypt.org/directory
    # Email address used for ACME registration
    email: youremail@yourdomain.com
    # Name of a secret used to store the ACME account private key
    privateKeySecretRef:
      name: letsencrypt-prod
    # Enable the HTTP-01 challenge provider
    http01: {}
```

### 配置DNS
获取nginx-ingress-controller的IP

```
kubectl get svc -n default
```
output:

```
NAME                            TYPE           CLUSTER-IP     EXTERNAL-IP     PORT(S)                      AGE
kubernetes                      ClusterIP      10.64.0.1      <none>          443/TCP                      64m
nginx-ingress-controller        LoadBalancer   10.64.11.170   35.188.93.188   80:30393/TCP,443:30515/TCP   59m
nginx-ingress-default-backend   ClusterIP      10.64.5.162    <none>          80/TCP                       59m
```
将你的DNS记录指向 EXTERNAL-IP -> `35.188.93.188`

## 部署web项目

### 创建namespace
```
kubectl create ns demo
kubectl config set-context $(kubectl config current-context) --namespace=demo
```

### 部署项目 

#### Deployment

```
kubectl apply -f deployment.yaml
```

#### Service

```
kubectl apply -f service.yaml
```

#### Ingress

```
kubectl apply -f ingress.yaml
```


# Links
* https://cloud.google.com/community/tutorials/nginx-ingress-gke
* https://www.digitalocean.com/community/tutorials/how-to-set-up-an-nginx-ingress-on-digitalocean-kubernetes-using-helm
* https://gist.github.com/snormore/c7c2935d746531ed0d75064a6ad6058e
* https://github.com/helm/helm/issues/3130#issuecomment-372931407
* [kubernetes-sample-app](https://github.com/wancaibida/kubernetes-sample-app)