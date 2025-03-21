# 네트워크 정책 구성

<br/>

## 1. 네임스페이스 생성 및 레이블 지정
```bash
# 네임스페이스 생성
kubectl create namespace frontend
kubectl create namespace backend
kubectl create namespace database

# 네임스페이스에 레이블 추가
kubectl label namespace frontend role=frontend
kubectl label namespace backend role=backend
kubectl label namespace database role=database

# 테스트용 파드 생성
kubectl run frontend-pod --image=nginx -n frontend
kubectl run backend-pod --image=nginx -n backend
kubectl run db-pod --image=nginx -n database
```

<br/>

## 2. 네트워크 정책 생성

### Frontend 네트워크 정책
```yaml
apiVersion: networking.k8s.io/v1
kind: NetworkPolicy
metadata:
  name: frontend-policy
  namespace: frontend
spec:
  policyTypes:
  - Ingress
  - Egress
  ingress: []  # 모든 인그레스 트래픽 차단
  egress:
  - to:
    - namespaceSelector:
        matchLabels:
          role: backend
```

### Backend 네트워크 정책
```yaml
apiVersion: networking.k8s.io/v1
kind: NetworkPolicy
metadata:
  name: backend-policy
  namespace: backend
spec:
  policyTypes:
  - Ingress
  - Egress
  ingress:
  - from:
    - namespaceSelector:
        matchLabels:
          role: frontend
  egress:
  - to:
    - namespaceSelector:
        matchLabels:
          role: database
```

### Database 네트워크 정책
```yaml
apiVersion: networking.k8s.io/v1
kind: NetworkPolicy
metadata:
  name: database-policy
  namespace: database
spec:
  policyTypes:
  - Ingress
  - Egress
  ingress:
  - from:
    - namespaceSelector:
        matchLabels:
          role: backend
  egress: []  # 모든 이그레스 트래픽 차단
```

<br/>

## 3. 정책 적용 및 테스트
```bash
# 정책 적용
kubectl apply -f frontend-policy.yaml
kubectl apply -f backend-policy.yaml
kubectl apply -f database-policy.yaml

# Pod IP 확인
FRONTEND_POD_IP=$(kubectl get pod frontend-pod -n frontend -o jsonpath='{.status.podIP}')
BACKEND_POD_IP=$(kubectl get pod backend-pod -n backend -o jsonpath='{.status.podIP}')
DB_POD_IP=$(kubectl get pod db-pod -n database -o jsonpath='{.status.podIP}')

# 테스트 수행 및 결과 기록
echo "네트워크 정책 테스트 결과" > /opt/network-policy-test.txt
echo "------------------------" >> /opt/network-policy-test.txt

# Frontend -> Backend 테스트
echo "1. Frontend -> Backend 테스트:" >> /opt/network-policy-test.txt
kubectl exec -n frontend frontend-pod -- wget -qO- --timeout=2 http://$BACKEND_POD_IP >> /opt/network-policy-test.txt 2>&1

# Frontend -> Database 테스트
echo "2. Frontend -> Database 테스트 (should fail):" >> /opt/network-policy-test.txt
kubectl exec -n frontend frontend-pod -- wget -qO- --timeout=2 http://$DB_POD_IP >> /opt/network-policy-test.txt 2>&1

# Backend -> Database 테스트
echo "3. Backend -> Database 테스트:" >> /opt/network-policy-test.txt
kubectl exec -n backend backend-pod -- wget -qO- --timeout=2 http://$DB_POD_IP >> /opt/network-policy-test.txt 2>&1

# Database -> 외부 테스트
echo "4. Database -> 외부 테스트 (should fail):" >> /opt/network-policy-test.txt
kubectl exec -n database db-pod -- wget -qO- --timeout=2 http://google.com >> /opt/network-policy-test.txt 2>&1
```

<br/>

## 주요 변경사항:
1. podSelector: {} 추가하여 네임스페이스의 모든 파드에 정책 적용
2. 명시적인 ingress/egress 규칙 설정
3. 빈 배열([])을 사용하여 모든 트래픽 차단
4. 테스트 결과를 파일에 기록

<br/>

## 주의사항:
1. 네트워크 정책은 CNI 플러그인이 지원해야 함
2. 정책 적용 전 기존 연결 확인
3. 테스트 시 timeout 설정 필요
4. 실패한 연결 시도도 기록해야 함
