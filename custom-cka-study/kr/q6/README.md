# 리소스 관리 및 스케줄링 구성

<br/>

## 1. GPU 노드 레이블링 및 파드 생성
```bash
# GPU 노드에 레이블 추가
kubectl label nodes <node-name> gpu=true

# GPU 파드 생성
cat <<EOF | kubectl apply -f -
apiVersion: v1
kind: Pod
metadata:
  name: gpu-pod
spec:
  containers:
  - name: gpu-container
    image: nvidia/cuda:latest
  nodeSelector:
    gpu: "true"
EOF
```

<br/>

## 2. 메모리 요구사항이 높은 파드 배포
```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: high-memory-app
spec:
  replicas: 3
  selector:
    matchLabels:
      app: high-memory-app
  template:
    metadata:
      labels:
        app: high-memory-app
    spec:
      topologySpreadConstraints:
      - maxSkew: 1
        topologyKey: kubernetes.io/hostname
        whenUnsatisfiable: DoNotSchedule
        labelSelector:
          matchLabels:
            app: high-memory-app
      containers:
      - name: high-memory-container
        image: nginx
        resources:
          requests:
            memory: "1Gi"
          limits:
            memory: "2Gi"
```

<br/>

## 3. Anti-affinity 규칙을 사용한 파드 배포
```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: webapp
spec:
  replicas: 3
  selector:
    matchLabels:
      app: webapp
  template:
    metadata:
      labels:
        app: webapp
    spec:
      affinity:
        podAntiAffinity:
          requiredDuringSchedulingIgnoredDuringExecution:
          - labelSelector:
              matchExpressions:
              - key: app
                operator: In
                values:
                - webapp
            topologyKey: "kubernetes.io/hostname"
      containers:
      - name: webapp
        image: nginx
```

<br/>

## 4. 모니터링 설정 및 검증
```bash
# metrics-server 설치
kubectl apply -f https://github.com/kubernetes-sigs/metrics-server/releases/latest/download/components.yaml

# metrics-server 작동 확인
kubectl get apiservice v1beta1.metrics.k8s.io

# 배포 상태 확인
kubectl get pods -o wide

# 리소스 사용량 모니터링
kubectl top nodes
kubectl top pods --all-namespaces
```

<br/>

## 검증 테스트
```bash
# GPU 파드 검증
echo "1. GPU 파드 배포 검증:" > /opt/scheduling-test.txt
kubectl get pod gpu-pod -o wide >> /opt/scheduling-test.txt

# 메모리 파드 분산 검증
echo -e "\n2. 메모리 파드 분산 검증:" >> /opt/scheduling-test.txt
kubectl get pods -l app=high-memory-app -o wide >> /opt/scheduling-test.txt

# Anti-affinity 검증
echo -e "\n3. Anti-affinity 규칙 검증:" >> /opt/scheduling-test.txt
kubectl get pods -l app=webapp -o wide >> /opt/scheduling-test.txt

# 노드 리소스 사용량
echo -e "\n4. 노드 리소스 사용량:" >> /opt/scheduling-test.txt
kubectl top nodes >> /opt/scheduling-test.txt
```

<br/>

## 주의사항:
1. GPU 노드가 실제로 존재하는지 확인
2. metrics-server가 정상적으로 작동하는지 확인
3. 충분한 노드 리소스가 있는지 확인
4. 노드 레이블이 올바르게 설정되었는지 확인
