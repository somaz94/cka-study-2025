# Kustomize로 HPA Autoscaler 구성하기

<br/>

## 문제 설명
`ssh cka5774`에서 다음 작업을 수행합니다:

이전에 `api-gateway` 애플리케이션은 외부 autoscaler를 사용했는데, 이를 HorizontalPodAutoscaler(HPA)로 교체해야 합니다. 애플리케이션은 다음과 같이 `api-gateway-staging`과 `api-gateway-prod` Namespace에 배포되었습니다:

```bash
kubectl kustomize /opt/course/5/api-gateway/staging | kubectl apply -f -
kubectl kustomize /opt/course/5/api-gateway/prod | kubectl apply -f -
```

`/opt/course/5/api-gateway`의 Kustomize 설정을 사용하여 다음을 수행하세요:

1. ConfigMap `horizontal-scaling-config` 완전히 제거
2. Deployment `api-gateway`를 위한 HPA `api-gateway` 추가 (min 2, max 4 replicas, CPU 사용률 50%에서 스케일링)
3. prod 환경에서는 HPA max replicas를 6으로 설정
4. staging과 prod에 변경사항 적용

<br/>

## 해결 방법

### 1. ConfigMap 제거
- `base/api-gateway.yaml, staging/api-gateway.yaml, prod/api-gateway.yaml`에서 ConfigMap 제거

```bash
k -n api-gateway-staging delete cm horizontal-scaling-config
k -n api-gateway-prod delete cm horizontal-scaling-config
```

<br/>

### 2. HPA 추가 (Base 설정)
```yaml
# base/api-gateway.yaml
apiVersion: autoscaling/v2
kind: HorizontalPodAutoscaler
metadata:
  name: api-gateway
spec:
  scaleTargetRef:
    apiVersion: apps/v1
    kind: Deployment
    name: api-gateway
  minReplicas: 2
  maxReplicas: 4
  metrics:
    - type: Resource
      resource:
        name: cpu
        target:
          type: Utilization
          averageUtilization: 50
```

<br/>

### 3. Prod HPA 설정
```yaml
# prod/api-gateway.yaml
apiVersion: autoscaling/v2
kind: HorizontalPodAutoscaler
metadata:
  name: api-gateway
spec:
  maxReplicas: 6
```

<br/>

### 4. 변경사항 적용
```bash
# Staging 적용
k kustomize staging | kubectl apply -f -

# Prod 적용
k kustomize prod | kubectl apply -f -

# ConfigMap 수동 삭제
k -n api-gateway-staging delete cm horizontal-scaling-config
k -n api-gateway-prod delete cm horizontal-scaling-config
```

<br/>

## 주의사항
- Kustomize는 상태를 유지하지 않아 원격 리소스를 자동으로 삭제하지 않습니다.
- HPA가 설정한 replicas 수가 Deployment의 기본값과 다를 수 있습니다.
- Deployment에서 replicas 설정을 제거하는 것이 좋습니다.

<br/>

## Kustomize vs Helm
### Kustomize
- 상태 관리 없음
- 단순한 구조
- 수동 정리 필요할 수 있음

### Helm
- 상태 관리 가능
- 원격 리소스 추적 가능
- 상태 오류/불일치 시 복잡해질 수 있음
- 동시 상태 변경 주의 필요
