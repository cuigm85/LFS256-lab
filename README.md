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
