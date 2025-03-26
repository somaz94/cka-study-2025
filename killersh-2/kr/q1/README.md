# DNS FQDN 설정 문제 해결

문제에서 lima-control 네임스페이스의 Deployment 컨트롤러가 다양한 클러스터 내부 엔드포인트와 DNS FQDN 값을 사용하여 통신하고 있습니다.

<br/>

## 1. 현재 상태 확인
```bash
# Pod의 hostname과 subdomain 설정 확인
kubectl -n lima-workload get pod section100 -o yaml

# 예상되는 Pod 설정
apiVersion: v1
kind: Pod
metadata:
  name: section100
  namespace: lima-workload
spec:
  hostname: section100    # hostname 설정
  subdomain: section     # subdomain 설정
  containers:
  - name: pod
    image: httpd:2-alpine

# Service 확인
kubectl -n lima-workload get svc section
```

<br/>

## 2. DNS FQDN 값 확인 및 테스트
```bash
# DNS_1: kubernetes 서비스
nslookup kubernetes.default.svc.cluster.local
# 결과: 10.96.0.1

# DNS_2: department 헤드리스 서비스
nslookup department.lima-workload.svc.cluster.local
# 결과: 10.32.0.2, 10.32.0.9 (여러 Pod IP 반환)

# DNS_3: section100 파드 (hostname + subdomain 사용)
nslookup section100.section.lima-workload.svc.cluster.local
# 결과: 10.32.0.10

# DNS_4: IP 기반 파드
nslookup 1-2-3-4.kube-system.pod.cluster.local
# 결과: 1.2.3.4
```

<br/>

## 3. ConfigMap 업데이트
```yaml
apiVersion: v1
kind: ConfigMap
metadata:
  name: control-config
  namespace: lima-control
data:
  DNS_1: "kubernetes.default.svc.cluster.local"
  DNS_2: "department.lima-workload.svc.cluster.local"
  DNS_3: "section100.section.lima-workload.svc.cluster.local"
  DNS_4: "1-2-3-4.kube-system.pod.cluster.local"
```

<br/>

## 4. 변경사항 적용
```bash
# ConfigMap 업데이트 후 디플로이먼트 재시작
kubectl -n lima-control rollout restart deploy controller

# 상태 및 로그 확인
kubectl -n lima-control get pods
kubectl -n lima-control logs -f controller-xxxxx
```

<br/>

## DNS FQDN 형식 설명:
1. 일반 서비스: `<service-name>.<namespace>.svc.cluster.local`
2. 헤드리스 서비스: `<service-name>.<namespace>.svc.cluster.local`
   - 서비스 IP 대신 Pod IP들이 반환됨
3. Pod (hostname + subdomain 사용): `<hostname>.<subdomain>.<namespace>.svc.cluster.local`
   - Pod의 spec에 hostname과 subdomain이 설정되어 있어야 함
   - subdomain과 동일한 이름의 Service가 있어야 함
4. IP 기반 Pod: `<ip-with-dashes>.<namespace>.pod.cluster.local`

<br/>

## 주의사항:
1. DNS_3의 경우 Pod의 hostname과 subdomain 설정이 필요
2. 헤드리스 서비스는 여러 Pod IP를 반환할 수 있음
3. ConfigMap 업데이트 후 Pod 재시작 필요
4. 모든 FQDN은 cluster.local로 끝나야 함