
step1. create efk
```
kubectl apply -f nginx-ingress.yaml
kubectl label node k8s119.centos.k8scloud.site log=true
kubectl create namespace logging
kubectl apply -f elasticsearch.yaml
kubectl apply -f kibana.yaml
sudo su - << FOE
echo "192.168.31.119 kibana.k8s119.xip.io" >> /etc/hosts
FOE
# open the browser and access kibana.k8s119.xip.io
kubectl apply -f fluentd-es-config-main.yaml  
kubectl apply -f fluentd-configmap.yaml  
kubectl apply -f fluentd.yaml  
```

step2. verify efk
```
kubectl apply -f counter-pod.yaml
```

other info.

a. about nginx-ingress.yaml
```
wget https://raw.githubusercontent.com/kubernetes/ingress-nginx/nginx-0.30.0/deploy/static/mandatory.yaml
mv mandatory.yaml nginx-ingress.yaml
vi nginx-ingress.yaml
---add
    spec:
      hostNetwork: true #添加为host模式
---
```

b. about HTTPS访问：

```powershell
#自签名证书
openssl req -x509 -nodes -days 2920 -newkey rsa:2048 -keyout tls.key -out tls.crt -subj "/CN=*.xip.io/O=ingress-nginx"

# 证书信息保存到secret对象中，ingress-nginx会读取secret对象解析出证书加载到nginx配置中
kubectl -n demo create secret tls https-secret --key tls.key --cert tls.crt

# 修改yaml
---
  tls:
  - hosts:
    - myblog.xip.io
    secretName: https-secret
---
```
