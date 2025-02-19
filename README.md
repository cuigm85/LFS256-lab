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

# Argo CD

[REF1](https://github.com/kubernetes/ingress-nginx/blob/main/docs/user-guide/nginx-configuration/annotations.md)

## Lab 3.1 - Installing Argo CD

### 3.1.1 Deploy Argo CD to Kubernetes

```
# NS 생성
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

# Argo Rollouts

## Lab 5.1 - Installing Argo Rollouts

### Deploy Argo Ro

```
# NS 생성
kubectl create namespace argo-rollouts


# 설치
kubectl apply -n argo-rollouts -f \
  https://github.com/argoproj/argo-rollouts/releases/latest/download/install.yaml

# 확인
kubectl get pods -n argo-rollouts
```

### Install Rollouts kubectl Plugin

[https://argoproj.github.io/argo-rollouts/installation/](https://argoproj.github.io/argo-rollouts/installation/)

```
# 설치
curl -LO https://github.com/argoproj/argo-rollouts/releases/latest/download/kubectl-argo-rollouts-linux-amd64
chmod +x ./kubectl-argo-rollouts-linux-amd64
sudo mv ./kubectl-argo-rollouts-linux-amd64 /usr/local/bin/kubectl-argo-rollouts

# 확인
kubectl argo rollouts version
```

```
# 자동완성
# source <(kubectl-argo-rollouts completion bash)

# 사용법 예시
# kubectl get rollout
# kubectl argo rollouts get rollout
# kubectl argo rollouts promote
# kubectl argo rollouts undo
```

## Lab 5.2 - Argo Rollouts Blue-Green

### Creating Blue-Green Deployments with Argo Rollouts

```
# Install Resources

kubectl get rollout
# No resources found in default namespace.

cat <<EOF | kubectl apply -f -
apiVersion: argoproj.io/v1alpha1
kind: Rollout
metadata:
  name: rollout-bluegreen
spec:
  replicas: 2
  revisionHistoryLimit: 2
  selector:
    matchLabels:
      app: rollout-bluegreen
  template:
    metadata:
      labels:
        app: rollout-bluegreen
    spec:
      containers:
      - name: rollouts-demo
        image: argoproj/rollouts-demo:blue
        imagePullPolicy: Always
        ports:
        - containerPort: 8080
  strategy:
    blueGreen:
      activeService: rollout-bluegreen-active
      previewService: rollout-bluegreen-preview
      autoPromotionEnabled: false
EOF

# rollout.argoproj.io/rollout-bluegreen created

# 확인
kubectl get rollout
kubectl argo rollouts get ro rollout-bluegreen
```

```
cat <<EOF | kubectl apply -f -
apiVersion: v1
kind: Service
metadata:
  labels:
    app: rollout-bluegreen-active
  name: rollout-bluegreen-active
spec:
  ports:
  - name: "80"
    port: 80
    protocol: TCP
    targetPort: 80
  selector:
    app: rollout-bluegreen
  type: ClusterIP
EOF
```

```
cat <<EOF | kubectl apply -f -
apiVersion: v1
kind: Service
metadata:
  labels:
    app: rollout-bluegreen-preview
  name: rollout-bluegreen-preview
spec:
  ports:
  - name: "80"
    port: 80
    protocol: TCP
    targetPort: 80
  selector:
    app: rollout-bluegreen
  type: ClusterIP
EOF
```
