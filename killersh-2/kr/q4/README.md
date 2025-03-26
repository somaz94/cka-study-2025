# Pod와 Service 연동 구성

문제 요구사항:
1. ready-if-service-ready Pod 생성
   - nginx:1-alpine 이미지 사용
   - LivenessProbe: true 명령어 실행
   - ReadinessProbe: service-am-i-ready:80 접근 가능 여부 확인
2. am-i-ready Pod 생성
   - nginx:1-alpine 이미지 사용
   - label: id=cross-server-ready
3. service-am-i-ready 서비스가 두 번째 Pod를 엔드포인트로 가지도록 구성

<br/>

## 1. 첫 번째 Pod 생성
```bash
# Pod YAML 생성
kubectl run ready-if-service-ready --image=nginx:1-alpine --dry-run=client -o yaml > pod1.yaml

# YAML 파일 수정
cat << EOF > pod1.yaml
apiVersion: v1
kind: Pod
metadata:
  name: ready-if-service-ready
  labels:
    run: ready-if-service-ready
spec:
  containers:
  - name: ready-if-service-ready
    image: nginx:1-alpine
    livenessProbe:
      exec:
        command:
        - 'true'
    readinessProbe:
      exec:
        command:
        - sh
        - -c
        - 'wget -T2 -O- http://service-am-i-ready:80'
EOF

# Pod 생성
kubectl apply -f pod1.yaml

# 상태 확인 - Ready가 아닌 상태여야 함
kubectl get pod ready-if-service-ready
kubectl describe pod ready-if-service-ready
```

<br/>

## 2. 두 번째 Pod 생성
```bash
# Pod 생성
kubectl run am-i-ready \
  --image=nginx:1-alpine \
  --labels="id=cross-server-ready"

# Pod 상태 확인
kubectl get pod am-i-ready
```

<br/>

## 3. 서비스 연동 확인
```bash
# 서비스 상태 확인
kubectl describe svc service-am-i-ready

# 엔드포인트 확인
kubectl get endpoints service-am-i-ready

# 첫 번째 Pod가 Ready 상태가 되었는지 확인
kubectl get pod ready-if-service-ready
```

<br/>

## 예상 결과:
1. 처음에는 ready-if-service-ready Pod가 NotReady 상태
   - ReadinessProbe가 service-am-i-ready에 접근 실패
2. am-i-ready Pod 생성 후
   - service-am-i-ready의 엔드포인트로 등록됨
   - ready-if-service-ready Pod가 Ready 상태로 변경

<br/>

## 주의사항:
1. ReadinessProbe의 타임아웃 설정(wget -T2) 확인
2. 서비스의 selector가 정확히 설정되어 있는지 확인
3. Pod의 레이블이 서비스 selector와 일치하는지 확인
4. ReadinessProbe가 성공하기까지 시간 소요될 수 있음