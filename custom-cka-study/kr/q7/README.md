# 서비스 및 인그레스 구성

<br/>

## 1. NodePort 서비스 구성
```yaml
# 테스트용 애플리케이션 배포
apiVersion: apps/v1
kind: Deployment
metadata:
  name: web-app
  namespace: default
spec:
  replicas: 2
  selector:
    matchLabels:
      app: web-app
  template:
    metadata:
      labels:
        app: web-app
    spec:
      containers:
      - name: nginx
        image: nginx:1.19
        ports:
        - containerPort: 80
---
# NodePort 서비스 생성
apiVersion: v1
kind: Service
metadata:
  name: web-app-service
  namespace: default
spec:
  type: NodePort
  selector:
    app: web-app
  ports:
  - port: 80
    targetPort: 80
    nodePort: 30080
```

<br/>

## 2. 인그레스 컨트롤러 설정
```bash
# NGINX 인그레스 컨트롤러 설치
kubectl apply -f https://raw.githubusercontent.com/kubernetes/ingress-nginx/controller-v1.8.1/deploy/static/provider/cloud/deploy.yaml

# 인그레스 클래스 생성
cat <<EOF | kubectl apply -f -
apiVersion: networking.k8s.io/v1
kind: IngressClass
metadata:
  name: nginx
  annotations:
    ingressclass.kubernetes.io/is-default-class: "true"
spec:
  controller: k8s.io/ingress-nginx
EOF
```

<br/>

## 3. 도메인 기반 라우팅 설정
```yaml
# 추가 테스트 애플리케이션 배포
apiVersion: apps/v1
kind: Deployment
metadata:
  name: web-app2
  namespace: default
spec:
  replicas: 2
  selector:
    matchLabels:
      app: web-app2
  template:
    metadata:
      labels:
        app: web-app2
    spec:
      containers:
      - name: nginx
        image: nginx:1.19
        ports:
        - containerPort: 80
---
# 두 번째 서비스 생성
apiVersion: v1
kind: Service
metadata:
  name: web-app2-service
spec:
  selector:
    app: web-app2
  ports:
  - port: 80
    targetPort: 80
---
# 도메인 기반 라우팅 인그레스 규칙
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  name: web-routing
  annotations:
    nginx.ingress.kubernetes.io/rewrite-target: /
spec:
  ingressClassName: nginx
  rules:
  - host: app1.example.com
    http:
      paths:
      - path: /
        pathType: Prefix
        backend:
          service:
            name: web-app-service
            port:
              number: 80
  - host: app2.example.com
    http:
      paths:
      - path: /
        pathType: Prefix
        backend:
          service:
            name: web-app2-service
            port:
              number: 80
```

<br/>

## 4. SSL/TLS 인증서 구성
```bash
# SSL 인증서 및 키 생성
openssl req -x509 -nodes -days 365 -newkey rsa:2048 \
  -keyout /tmp/tls.key \
  -out /tmp/tls.crt \
  -subj "/CN=*.example.com"

# TLS Secret 생성
kubectl create secret tls example-tls \
  --key /tmp/tls.key \
  --cert /tmp/tls.crt
```

```yaml
# TLS 설정이 포함된 인그레스
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  name: web-routing-tls
  annotations:
    nginx.ingress.kubernetes.io/rewrite-target: /
spec:
  ingressClassName: nginx
  tls:
  - hosts:
    - app1.example.com
    - app2.example.com
    secretName: example-tls
  rules:
  - host: app1.example.com
    http:
      paths:
      - path: /
        pathType: Prefix
        backend:
          service:
            name: web-app-service
            port:
              number: 80
  - host: app2.example.com
    http:
      paths:
      - path: /
        pathType: Prefix
        backend:
          service:
            name: web-app2-service
            port:
              number: 80
