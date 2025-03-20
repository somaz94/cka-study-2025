# 노드당 하나의 Pod만 실행하는 Deployment 구성

<br/>

## 문제 설명
`ssh cka2556`에서 다음 작업을 수행합니다:

Namespace `project-tiger`에 다음 조건을 만족하는 Deployment를 생성하세요:
- 이름: deploy-important
- 레플리카: 3개
- Deployment와 Pod에 레이블: id=very-important
- 첫 번째 컨테이너: container1 (nginx:1-alpine)
- 두 번째 컨테이너: container2 (google/pause)
- 워커 노드당 하나의 Pod만 실행되어야 함 (topologyKey: kubernetes.io/hostname 사용)

<br/>

## 해결 방법

### 1. podAntiAffinity 사용
```yaml
# 12.yaml (podAntiAffinity 방식)
apiVersion: apps/v1
kind: Deployment
metadata:
  name: deploy-important
  namespace: project-tiger
  labels:
    id: very-important
spec:
  replicas: 3
  selector:
    matchLabels:
      id: very-important
  template:
    metadata:
      labels:
        id: very-important
    spec:
      containers:
      - name: container1
        image: nginx:1-alpine
      - name: container2
        image: google/pause
      affinity:
        podAntiAffinity:
          requiredDuringSchedulingIgnoredDuringExecution:
          - labelSelector:
              matchExpressions:
              - key: id
                operator: In
                values:
                - very-important
            topologyKey: kubernetes.io/hostname
```

<br/>

### 2. topologySpreadConstraints 사용
```yaml
# 12.yaml (topologySpreadConstraints 방식)
apiVersion: apps/v1
kind: Deployment
metadata:
  name: deploy-important
  namespace: project-tiger
  labels:
    id: very-important
spec:
  replicas: 3
  selector:
    matchLabels:
      id: very-important
  template:
    metadata:
      labels:
        id: very-important
    spec:
      containers:
      - name: container1
        image: nginx:1-alpine
      - name: container2
        image: google/pause
      topologySpreadConstraints:
      - maxSkew: 1
        topologyKey: kubernetes.io/hostname
        whenUnsatisfiable: DoNotSchedule
        labelSelector:
          matchLabels:
            id: very-important
```

<br/>

### 3. 배포 및 확인
```bash
# Deployment 생성
k -f 12.yaml create

# Deployment 상태 확인
k -n project-tiger get deploy -l id=very-important
NAME               READY   UP-TO-DATE   AVAILABLE   AGE
deploy-important   2/3     3            2           19s

# Pod 배포 상태 확인
k -n project-tiger get pod -o wide -l id=very-important
NAME                                READY   STATUS    ...   NODE
deploy-important-78f98b75f9-5s6js   0/2     Pending   ...   <none>
deploy-important-78f98b75f9-657hx   2/2     Running   ...   cka2556-node1
deploy-important-78f98b75f9-9bz8q   2/2     Running   ...   cka2556-node2
```

<br/>

## 주의사항
- podAntiAffinity나 topologySpreadConstraints 둘 중 하나를 선택하여 사용
- 워커 노드가 2개이고 레플리카가 3개이므로 1개의 Pod는 Pending 상태가 됨
- 이는 DaemonSet과 유사하지만 고정된 레플리카 수를 가진 Deployment를 사용하는 방식
- Pending Pod의 상태는 kubectl describe로 확인 가능
