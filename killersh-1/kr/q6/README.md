# Storage, PV, PVC, Pod Volume 구성하기

<br/>

## 문제 설명
`ssh cka7968`에서 다음 작업을 수행합니다:

1. PersistentVolume `safari-pv` 생성
   - 용량: 2Gi
   - accessMode: ReadWriteOnce
   - hostPath: /Volumes/Data
   - storageClassName: 정의하지 않음

2. Namespace `project-t230`에 PersistentVolumeClaim `safari-pvc` 생성
   - 요청 용량: 2Gi
   - accessMode: ReadWriteOnce
   - storageClassName: 정의하지 않음
   - PV와 올바르게 바인딩되어야 함

3. Namespace `project-t230`에 Deployment `safari` 생성
   - 볼륨 마운트 위치: /tmp/safari-data
   - 이미지: httpd:2-alpine

<br/>

## 해결 방법

### 1. PersistentVolume 생성
```yaml
# 6_pv.yaml
kind: PersistentVolume
apiVersion: v1
metadata:
  name: safari-pv
spec:
  capacity:
    storage: 2Gi
  accessModes:
    - ReadWriteOnce
  hostPath:
    path: "/Volumes/Data"
```

```bash
# PV 생성
k -f 6_pv.yaml create
```

> ⚠️ hostPath 볼륨 타입은 보안 위험이 있으므로 가능한 피해야 합니다. 
> hostPath 디렉토리의 데이터는 노드 간에 공유되지 않으며, 
> Pod가 스케줄된 노드에 따라 사용 가능한 데이터가 달라집니다.

<br/>

### 2. PersistentVolumeClaim 생성
```yaml
# 6_pvc.yaml
kind: PersistentVolumeClaim
apiVersion: v1
metadata:
  name: safari-pvc
  namespace: project-t230
spec:
  accessModes:
    - ReadWriteOnce
  resources:
    requests:
      storage: 2Gi
```

```bash
# PVC 생성
k -f 6_pvc.yaml create

# PV, PVC 상태 확인
k -n project-t230 get pv,pvc
```

<br/>

### 3. Deployment 생성
```yaml
# 6_dep.yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  labels:
    app: safari
  name: safari
  namespace: project-t230
spec:
  replicas: 1
  selector:
    matchLabels:
      app: safari
  template:
    metadata:
      labels:
        app: safari
    spec:
      volumes:                                    # 추가
      - name: data                                # 추가
        persistentVolumeClaim:                    # 추가
          claimName: safari-pvc                   # 추가
      containers:
      - image: httpd:2-alpine
        name: container
        volumeMounts:                             # 추가
        - name: data                              # 추가
          mountPath: /tmp/safari-data             # 추가
```

<br/>

```bash
# Deployment 생성
k -f 6_dep.yaml create

# 마운트 상태 확인
k -n project-t230 describe pod safari-b499cc5b9-x7d7h | grep -A2 Mounts:
```

<br/>

## 확인사항
- PV와 PVC가 올바르게 Bound 상태인지 확인
- Pod에 볼륨이 올바르게 마운트되었는지 확인
- hostPath 사용 시 보안 위험 고려
