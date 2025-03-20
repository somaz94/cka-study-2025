# StatefulSet 스케일 다운

<br/>

## 문제 설명
`ssh cka3962`에서 다음 작업을 수행합니다:

`project-h800` Namespace에 `o3db-*`라는 이름의 두 개의 Pod가 있습니다. Project H800 관리팀이 리소스 절약을 위해 이를 하나의 replica로 축소하도록 요청했습니다.

<br/>

## 해결 방법

### 1. Pod 상태 확인
```bash
# Pod 목록 확인
k -n project-h800 get pod | grep o3db
```

결과:
```bash
o3db-0                                  1/1     Running   0          6d19h
o3db-1                                  1/1     Running   0          6d19h
```

<br/>

### 2. 리소스 타입 확인
```bash
# Pod를 관리하는 리소스 확인 (Deployment, DaemonSet, StatefulSet)
k -n project-h800 get deploy,ds,sts | grep o3db
```

결과:
```bash
statefulset.apps/o3db   2/2     6d19h
```

<br/>

### 3. Pod 레이블 확인 (선택사항)
```bash
# Pod의 레이블 확인
k -n project-h800 get pod --show-labels | grep o3db
```

결과:
```bash
o3db-0    1/1  Running  0  6d19h   app=nginx,apps.kubernetes.io/pod-index=0,controller-revision-hash=o3db-5fbd4bb9cc,statefulset.kubernetes.io/pod-name=o3db-0
o3db-1    1/1  Running  0  6d19h   app=nginx,apps.kubernetes.io/pod-index=1,controller-revision-hash=o3db-5fbd4bb9cc,statefulset.kubernetes.io/pod-name=o3db-1
```

<br/>

### 4. StatefulSet 스케일 다운
```bash
# StatefulSet을 1개의 replica로 축소
k -n project-h800 scale sts o3db --replicas 1

# 상태 확인
k -n project-h800 get sts o3db
```

결과:
```bash
NAME   READY   AGE
o3db   1/1     6d19h
```

<br/>

## 결론
StatefulSet의 replica 수를 2개에서 1개로 줄여 Project H800 관리팀의 요청을 성공적으로 수행했습니다.
