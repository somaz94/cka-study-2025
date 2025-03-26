# Static Pod와 NodePort 서비스 구성

문제에서 다음 작업을 수행해야 합니다:
1. controlplane 노드에 Static Pod 생성
   - 이름: my-static-pod
   - 네임스페이스: default
   - 이미지: nginx:1-alpine
   - 리소스 요청: CPU 10m, 메모리 20Mi
2. NodePort 서비스 생성하여 Static Pod 노출
   - 이름: static-pod-service
   - 포트: 80

<br/>

## 1. Static Pod 매니페스트 생성
```bash
# Static Pod 디렉토리 확인
sudo find / -name "manifests" | grep kubelet

# 일반적인 위치: /etc/kubernetes/manifests/
# Static Pod 매니페스트 생성
cat << EOF | sudo tee /etc/kubernetes/manifests/my-static-pod.yaml
apiVersion: v1
kind: Pod
metadata:
  name: my-static-pod
  namespace: default
spec:
  containers:
  - name: nginx
    image: nginx:1-alpine
    resources:
      requests:
        cpu: "10m"
        memory: "20Mi"
    ports:
    - containerPort: 80
EOF
```

<br/>

## 2. Static Pod 확인
```bash
# Pod 생성 확인
kubectl get pods -n default
# 결과: my-static-pod-<node-name> 형태로 표시됨

# Pod 상세 정보 확인
kubectl describe pod my-static-pod-<node-name> -n default
```

<br/>

## 3. NodePort 서비스 생성
```bash
# 서비스 생성
cat << EOF | kubectl apply -f -
apiVersion: v1
kind: Service
metadata:
  name: static-pod-service
  namespace: default
spec:
  type: NodePort
  ports:
  - port: 80
    targetPort: 80
  selector:
    kubernetes.io/hostname: $(hostname)  # Static Pod가 실행 중인 노드 선택
EOF
```

<br/>

## 4. 검증
```bash
# 서비스 확인
kubectl get svc static-pod-service
# NodePort 번호 확인

# 엔드포인트 확인
kubectl get endpoints static-pod-service

# 노드 IP 확인
kubectl get nodes -o wide

# 서비스 접근 테스트
curl 192.168.100.31:<NODE_PORT>
```

<br/>

## 주의사항:
1. Static Pod 매니페스트는 올바른 디렉토리에 위치해야 함
2. kubelet이 매니페스트를 감지하고 Pod를 생성할 때까지 대기 필요
3. 서비스의 selector가 Static Pod를 정확히 선택해야 함
4. NodePort 서비스의 포트 범위는 30000-32767
