# LFS256-lab

### [Installation Guide : Helm](https://helm.sh/docs/intro/install/#from-the-binary-releases)

```
HELM_VERSION=v3.17.1
wget https://get.helm.sh/helm-${HELM_VERSION}-linux-amd64.tar.gz
tar zxvf helm-${HELM_VERSION}-linux-amd64.tar.gz
sudo mv linux-amd64/helm /usr/local/bin/helm
```

### [Installation Guide : Ingress NGINX Controller](https://kubernetes.github.io/ingress-nginx/deploy/#quick-start)

```
INGRESS_NGINX_VERSION=4.12.0
wget https://github.com/kubernetes/ingress-nginx/releases/download/helm-chart-${INGRESS_NGINX_VERSION}/ingress-nginx-${INGRESS_NGINX_VERSION}.tgz
tar zxvf ingress-nginx-${INGRESS_NGINX_VERSION}.tgz

cat <<EOF | tee values.yaml
controller:
  service:
    type: NodePort
    nodePorts:
      http: 32080
      https: 32443
      tcp:
        8080: 32808
EOF

helm upgrade --install ingress-nginx ./ingress-nginx --namespace ingress-nginx --create-namespace -f values.yaml
```

# ARGO CD
[REF1](https://github.com/kubernetes/ingress-nginx/blob/main/docs/user-guide/nginx-configuration/annotations.md)

## Lab 3.1 - Installing Argo CD

### 3.1.1 Deploy Argo CD to Kubernetes

```
# NS생성
kubectl create namespace argocd

# 설치
kubectl apply -n argocd -f \
  https://raw.githubusercontent.com/argoproj/argo-cd/stable/manifests/install.yaml

# 확인
kubectl get pods -n argocd
```

### 3.1.2 Accessing Argo CD

```
# 초기 암호 확인
kubectl -n argocd get secret argocd-initial-admin-secret -o \
jsonpath="{.data.password}" | base64 -d; echo
# 샘플 출력: 2bVRZRB7WFuKo43d
```
```
# 서버 접근 설정 port-forward
kubectl port-forward svc/argocd-server -n argocd 8080:443
```
```
# 서버 접근 설정 ingress
cat <<EOF | tee argocd-ingress-v1.yaml
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  name: argocd-ingress
  namespace: argocd
  annotations:
    nginx.ingress.kubernetes.io/rewrite-target: /
    nginx.ingress.kubernetes.io/force-ssl-redirect: "true"
    nginx.ingress.kubernetes.io/backend-protocol: "HTTPS"
    nginx.ingress.kubernetes.io/service-upstream: "true"
spec:
  ingressClassName: nginx
  rules:
  - http:
      paths:
      - path: /
        pathType: Prefix
        backend:
          service:
            name: argocd-server
            port:
              number: 443
EOF

kubectl apply -f argocd-ingress-v1.yaml
```
```
# 서버 접근 설정 ingress
cat <<EOF | tee argocd-ingress-v2.yaml
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  name: argocd-ingress
  namespace: argocd
  annotations:
    nginx.ingress.kubernetes.io/rewrite-target: /
    nginx.ingress.kubernetes.io/force-ssl-redirect: "true"
    nginx.ingress.kubernetes.io/backend-protocol: "HTTPS"
    nginx.ingress.kubernetes.io/service-upstream: "true"
spec:
  ingressClassName: nginx
  rules:
  - host: "argocd.localhost"
    http:
      paths:
      - path: /
        pathType: Prefix
        backend:
          service:
            name: argocd-server
            port:
              number: 443
EOF

kubectl apply -f argocd-ingress-v2.yaml
```
