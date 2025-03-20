# 모든 노드에 DaemonSet 배포하기

<br/>

## 문제 설명
`ssh cka2556`에서 다음 작업을 수행합니다:

Namespace `project-tiger`에 다음 조건을 만족하는 DaemonSet을 생성하세요:
- 이름: ds-important
- 이미지: httpd:2-alpine
- 레이블: 
  - id=ds-important
  - uuid=18426a0b-5f59-4e10-923f-c0e078e82462
- 리소스 요청:
  - CPU: 10 millicore
  - 메모리: 10 mebibyte
- 모든 노드(컨트롤플레인 포함)에서 실행되어야 함

<br/>

## 해결 방법

### 1. DaemonSet YAML 작성
```yaml
# 11.yaml
apiVersion: apps/v1
kind: DaemonSet
metadata:
  name: ds-important
  namespace: project-tiger
  labels:
    id: ds-important
    uuid: 18426a0b-5f59-4e10-923f-c0e078e82462
spec:
  selector:
    matchLabels:
      id: ds-important
      uuid: 18426a0b-5f59-4e10-923f-c0e078e82462
  template:
    metadata:
      labels:
        id: ds-important
        uuid: 18426a0b-5f59-4e10-923f-c0e078e82462
    spec:
      containers:
      - name: ds-important
        image: httpd:2-alpine
        resources:
          requests:
            cpu: 10m
            memory: 10Mi
      tolerations:                                  # 컨트롤플레인 노드 실행을 위한 설정
      - effect: NoSchedule
        key: node-role.kubernetes.io/control-plane
```

<br/>

### 2. DaemonSet 생성 및 확인
```bash
# DaemonSet 생성
k -f 11.yaml create

# DaemonSet 상태 확인
k -n project-tiger get ds
NAME           DESIRED   CURRENT   READY   UP-TO-DATE   AVAILABLE   NODE SELECTOR   AGE
ds-important   3         3         3       3            3           <none>          8s

# Pod 배포 상태 확인
k -n project-tiger get pod -l id=ds-important -o wide
NAME                 READY   STATUS    ...    NODE            ...
ds-important-26456   1/1     Running   ...    cka2556-node2   ...
ds-important-wnt5p   1/1     Running   ...    cka2556         ...
ds-important-wrbjd   1/1     Running   ...    cka2556-node1   ...
```

<br/>

## 주의사항
- DaemonSet은 각 노드에 하나의 Pod만 배포합니다
- 컨트롤플레인 노드에 Pod을 배포하려면 toleration 설정이 필요합니다
- 레이블은 selector와 template 모두에 정확히 지정해야 합니다
- 리소스 요청은 millicore(m)와 mebibyte(Mi) 단위를 사용합니다
