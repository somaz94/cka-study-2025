# 컨트롤플레인 노드에 Pod 스케줄링

문제 요구사항:
1. default 네임스페이스에 httpd:2-alpine 이미지로 Pod 생성
2. Pod 이름: pod1, 컨테이너 이름: pod1-container
3. 컨트롤플레인 노드에만 스케줄링되도록 구성
4. 노드에 새로운 레이블 추가하지 않음

<br/>

## 1. 컨트롤플레인 노드 확인
```bash
# 노드 확인
kubectl get node

# 컨트롤플레인 노드의 taint 확인
kubectl describe node cka5248 | grep Taint -A1
# 결과: node-role.kubernetes.io/control-plane:NoSchedule

# 노드 레이블 확인
kubectl get node cka5248 --show-labels
```

<br/>

## 2. Pod YAML 생성
```bash
# 기본 Pod YAML 생성
kubectl run pod1 --image=httpd:2-alpine --dry-run=client -o yaml > 12.yaml

# YAML 수정 - 방법 1: nodeSelector와 tolerations 사용
cat << EOF > 12.yaml
apiVersion: v1
kind: Pod
metadata:
  name: pod1
  labels:
    run: pod1
spec:
  containers:
  - name: pod1-container
    image: httpd:2-alpine
  tolerations:
  - effect: NoSchedule
    key: node-role.kubernetes.io/control-plane
  nodeSelector:
    node-role.kubernetes.io/control-plane: ""
EOF

# 또는 방법 2: nodeName 직접 지정
cat << EOF > 12.yaml
apiVersion: v1
kind: Pod
metadata:
  name: pod1
  labels:
    run: pod1
spec:
  containers:
  - name: pod1-container
    image: httpd:2-alpine
  nodeName: cka5248
EOF
```

<br/>

## 3. Pod 생성 및 확인
```bash
# Pod 생성
kubectl apply -f 12.yaml

# Pod 상태 및 노드 배치 확인
kubectl get pod pod1 -o wide
```

<br/>

## 구성 설명:
1. 방법 1: nodeSelector와 tolerations 사용
   - tolerations: 컨트롤플레인 노드의 NoSchedule taint를 허용
   - nodeSelector: 컨트롤플레인 노드에만 스케줄링되도록 강제
   - 장점: 유연한 스케줄링 정책 구현 가능
   - 단점: 설정이 더 복잡함

2. 방법 2: nodeName 사용
   - 특정 노드에 직접 스케줄링
   - 장점: 설정이 간단함
   - 단점: 유연성이 떨어지고 노드 장애 시 대응이 어려움

<br/>

## 예상 결과:
```bash
NAME   READY   STATUS    RESTARTS   AGE   IP       NODE     
pod1   1/1     Running   0          3s    <none>   cka5248
```

<br/>

## 주의사항:
1. toleration만으로는 불충분 (워커 노드에도 스케줄링 가능)
2. nodeSelector를 추가하여 컨트롤플레인 노드에만 스케줄링
3. 기존 노드 레이블만 사용
4. 컨테이너 이름이 정확히 pod1-container여야 함
5. nodeName 사용 시 직접적인 노드 지정으로 인한 유연성 감소 고려
