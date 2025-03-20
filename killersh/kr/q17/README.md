# Pod의 컨테이너 찾고 정보 확인하기

<br/>

## 문제 설명
`ssh cka2556`에서 다음 작업을 수행합니다:

Namespace `project-tiger`에서 다음 조건을 만족하는 Pod를 생성하고 관련 정보를 찾으세요:
- Pod 이름: tigers-reunite
- 이미지: httpd:2-alpine
- 레이블: pod=container, container=pod

1. Pod가 스케줄된 노드에 접속하여 해당 Pod의 containerd 컨테이너를 찾으세요
2. crictl 명령어를 사용하여:
   - 컨테이너 ID와 info.runtimeType을 `/opt/course/17/pod-container.txt`에 저장
   - 컨테이너 로그를 `/opt/course/17/pod-container.log`에 저장

<br/>

## 해결 방법

### 1. Pod 생성 및 노드 확인
```bash
# Pod 생성 (CLI 방식)
k -n project-tiger run tigers-reunite \
  --image=httpd:2-alpine \
  --labels "pod=container,container=pod"

# Pod 생성 (yaml 방식)
apiVersion: v1
kind: Pod
metadata:
  name: tigers-reunite
  namespace: project-tiger
  labels:
    pod: container
    container: pod
spec:
  containers:
  - name: tigers-reunite
    image: httpd:2-alpine


# Pod가 실행 중인 노드 확인
k -n project-tiger get pod -o wide
NAME            READY   STATUS    ...   NODE
tigers-reunite  1/1     Running   ...   cka2556-node1
```

<br/>

### 2. 노드에 접속하여 컨테이너 정보 확인
```bash
# 노드 접속
ssh cka2556-node1

# 컨테이너 ID 찾기
sudo crictl ps | grep tigers-reunite
ba62e5d465ff0   a7ccaadd632cf   2 minutes ago   Running   tigers-reunite   ...

# 컨테이너 런타임 타입 확인
sudo crictl inspect ba62e5d465ff0 | grep runtimeType
    "runtimeType": "io.containerd.runc.v2",

# 정보 저장
exit # 노드 접속 종료
echo "ba62e5d465ff0 io.containerd.runc.v2" > /opt/course/17/pod-container.txt
```

<br/>

### 3. 노드에 접속하여 컨테이너 로그 저장
```bash
# 노드 접속
ssh cka2556-node1

# 로그 확인
sudo crictl logs ba62e5d465ff0 

# 노드 접속 종료
exit

# 로그 저료
vi /opt/course/17/pod-container.log
# 확인인 로그

# 저장
: w장

```

<br/>

## 주의사항
- crictl은 컨테이너 런타임 인터페이스(CRI) 도구로, 컨테이너 관리에 사용됩니다
- 실제 시험에서는 docker 명령어를 사용할 수도 있습니다
- 로그가 많은 경우 scp를 사용하여 파일을 전송할 수 있습니다
- 컨테이너 ID는 시스템마다 다를 수 있습니다
