# NetworkPolicy로 Pod 간 통신 제한하기

<br/>

## 문제 설명
`ssh cka7968`에서 다음 작업을 수행합니다:

해킹된 backend Pod에서 전체 클러스터에 접근할 수 있었던 보안 사고가 있었습니다. 이를 방지하기 위해 Namespace `project-snake`에 NetworkPolicy `np-backend`를 생성하여 `backend-*` Pod들이 다음 통신만 가능하도록 제한하세요:

- `db1-*` Pod들과 포트 `1111`로 연결
- `db2-*` Pod들과 포트 `2222`로 연결

Pod 레이블의 `app` 값을 정책에 사용하세요.

<br/>

## 해결 방법

### 1. 현재 상태 확인
```bash
# Pod 및 레이블 확인
k -n project-snake get pod -L app
NAME        READY   STATUS    RESTARTS   AGE   APP
backend-0   1/1     Running   0          8d    backend
db1-0       1/1     Running   0          8d    db1
db2-0       1/1     Running   0          8d    db2
vault-0     1/1     Running   0          8d    vault

# 현재 연결 테스트
k -n project-snake exec backend-0 -- curl -s 10.32.0.11:1111  # db1
database one

k -n project-snake exec backend-0 -- curl -s 10.32.0.12:2222  # db2
database two

k -n project-snake exec backend-0 -- curl -s 10.32.0.13:3333  # vault
vault secret storage  # 이 연결은 차단되어야 함
```

<br/>

### 2. NetworkPolicy 생성
```yaml
# 15_np.yaml
apiVersion: networking.k8s.io/v1
kind: NetworkPolicy
metadata:
  name: np-backend
  namespace: project-snake
spec:
  podSelector:
    matchLabels:
      app: backend    # backend Pod들에 적용
  policyTypes:
    - Egress         # 외부 통신 제한
  egress:
    # db1 규칙
  - to:
    - podSelector:
        matchLabels:
          app: db1  # db1 Pod 허용
    ports:
    - protocol: TCP
      port: 1111   # 포트 1111만 허용
    
    # db2 규칙
  - to:
    - podSelector:
        matchLabels:
          app: db2  # db2 Pod 허용
    ports:
    - protocol: TCP
      port: 2222   # 포트 2222만 허용
```

<br/>

### 3. 정책 적용 및 확인
```bash
# NetworkPolicy 생성
k -f 15_np.yaml create

# 연결 테스트
k -n project-snake exec backend-0 -- curl -s 10.32.0.11:1111  # db1
database one

k -n project-snake exec backend-0 -- curl -s 10.32.0.12:2222  # db2
database two

k -n project-snake exec backend-0 -- curl -s 10.32.0.13:3333  # 실패!
^C
```

<br/>

## 주의사항
- NetworkPolicy는 규칙이 명확해야 합니다
- 잘못된 구성 예시:
  ```yaml
  # 잘못된 예시 - 의도하지 않은 통신 허용
  egress:
    - to:
      - podSelector:
          matchLabels:
            app: db1
      - podSelector:
          matchLabels:
            app: db2
      ports:
      - protocol: TCP
        port: 1111
      - protocol: TCP
        port: 2222
  ```
- 이 잘못된 예시에서는 db2에 1111 포트로도 접근이 가능해집니다
- NetworkPolicy 생성 후 `kubectl describe`로 정책이 올바르게 해석되었는지 확인하는 것이 좋습니다
