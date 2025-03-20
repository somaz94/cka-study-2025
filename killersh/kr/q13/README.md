# Gateway API로 Ingress 교체하기

<br/>

## 문제 설명
`ssh cka7968`에서 다음 작업을 수행합니다:

Project r500 팀이 기존 Ingress(networking.k8s.io)를 Gateway API(gateway.networking.k8s.io) 솔루션으로 교체하려고 합니다. 기존 Ingress는 `/opt/course/13/ingress.yaml`에서 확인할 수 있습니다.

Namespace `project-r500`에서 기존 Gateway에 대해 다음을 수행하세요:

1. 기존 Ingress의 라우트를 복제하는 HTTPRoute `traffic-director` 생성
2. 새로운 HTTPRoute에 `/auto` 경로를 추가하여:
   - User-Agent가 정확히 `mobile`이면 mobile로 리다이렉트
   - 그 외의 경우 desktop으로 리다이렉트

<br/>

## 해결 방법

### 1. 기존 Ingress 분석
```yaml
# /opt/course/13/ingress.yaml
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  name: traffic-director
spec:
  ingressClassName: nginx
  rules:
    - host: r500.gateway
      http:
        paths:
          - path: /desktop
            pathType: Prefix
            backend:
              service:
                name: web-desktop
                port:
                  number: 80
          - path: /mobile
            pathType: Prefix
            backend:
              service:
                name: web-mobile
                port:
                  number: 80
```

<br/>


### 2. Gateway 확인
```bash
k get gateway -n project-r500
NAME            CLASS   HOSTS   AGE
main            nginx   r500.gateway   10m

# Gateway 상세 정보 확인
k get gateway main -n project-r500 -o yaml

# Gateway 예시
piVersion: gateway.networking.k8s.io/v1
kind: Gateway
metadata:
  name: main
  namespace: project-r500
spec:
  gatewayClassName: nginx    # 사용할 Gateway 구현체 지정
  listeners:                 # Gateway가 수신할 트래픽 설정
  - name: http              # 리스너 이름
    port: 80               # 수신할 포트
    protocol: HTTP         # 프로토콜 (HTTP, HTTPS, TCP 등)
    allowedRoutes:         # 허용할 라우트 설정
      namespaces:
        from: Same         # 같은 네임스페이스의 라우트만 허용

# HTTPS 사용하는 경우 예시
apiVersion: gateway.networking.k8s.io/v1
kind: Gateway
metadata:
  name: main
  namespace: project-r500
spec:
  gatewayClassName: nginx
  listeners:
  - name: https
    port: 443
    protocol: HTTPS
    tls:
      mode: Terminate
      certificateRefs:
      - name: my-cert
    allowedRoutes:
      namespaces:
        from: Same
```

<br/>

### 3. HTTPRoute 생성
```yaml
# httproute.yaml
apiVersion: gateway.networking.k8s.io/v1
kind: HTTPRoute
metadata:
  name: traffic-director
  namespace: project-r500
spec:
  parentRefs:
    - name: main   # 기존 Gateway 사용
  hostnames:
    - "r500.gateway"
  rules:
    # desktop 경로
    - matches:
        - path:
            type: PathPrefix
            value: /desktop
      backendRefs:
        - name: web-desktop
          port: 80
    # mobile 경로
    - matches:
        - path:
            type: PathPrefix
            value: /mobile
      backendRefs:
        - name: web-mobile
          port: 80
    # auto 경로 - mobile 조건
    - matches:
        - path:
            type: PathPrefix
            value: /auto
          headers:
            - type: Exact
              name: user-agent
              value: mobile
      backendRefs:
        - name: web-mobile
          port: 80
    # auto 경로 - 기본값
    - matches:
        - path:
            type: PathPrefix
            value: /auto
      backendRefs:
        - name: web-desktop
          port: 80
```

<br/>

### 3. 동작 확인
```bash
# desktop 경로 테스트
curl r500.gateway:30080/desktop
Web Desktop App

# mobile 경로 테스트
curl r500.gateway:30080/mobile
Web Mobile App

# auto 경로 테스트 (mobile)
curl -H "User-Agent: mobile" r500.gateway:30080/auto
Web Mobile App

# auto 경로 테스트 (기타)
curl r500.gateway:30080/auto
Web Desktop App
```

<br/>

## 주의사항
- Gateway API는 기존 Ingress보다 더 유연하고 확장 가능한 구조를 제공
- rules의 순서가 중요함 (특히 /auto 경로에서 mobile 조건이 먼저 와야 함)
- path와 header는 AND 조건으로 연결됨
- 기존 Gateway와 GatewayClass가 이미 설정되어 있어야 함
