# Node와 Pod 리소스 사용량 확인

<br/>

## 문제 설명
`ssh cka5774`에서 다음 작업을 수행합니다:

metrics-server가 클러스터에 설치되어 있습니다. `kubectl`을 사용하여 두 개의 bash 스크립트를 작성하세요:

1. `/opt/course/7/node.sh`: Node의 리소스 사용량을 보여주는 스크립트
2. `/opt/course/7/pod.sh`: Pod와 컨테이너의 리소스 사용량을 보여주는 스크립트

<br/>

## 해결 방법

### 1. kubectl top 명령어 확인
```bash
# kubectl top 도움말 확인
k top -h

# 결과:
Display resource (CPU/memory) usage. 
The top command allows you to see the resource consumption for nodes or pods.
This command requires Metrics Server to be correctly configured and working on the server.

Available Commands:
  node        Display resource (CPU/memory) usage of nodes
  pod         Display resource (CPU/memory) usage of pods
```

<br/>

### 2. Node 리소스 사용량 확인
```bash
# Node 리소스 사용량 테스트
k top node

# 결과:
NAME            CPU(cores)   CPU%   MEMORY(bytes)   MEMORY%
cka5774         104m         10%    1121Mi          60%
```

<br/>

### 3. node.sh 스크립트 작성
```bash
# /opt/course/7/node.sh
kubectl top node
```

<br/>

### 4. Pod 리소스 사용량 확인
```bash
# Pod top 명령어 옵션 확인
k top pod -h

# 주요 옵션:
--containers=false: If present, print usage of containers within a pod
```

<br/>

### 5. pod.sh 스크립트 작성
```bash
# /opt/course/7/pod.sh
kubectl top pod --containers=true --all-namespaces
```

<br/>

## 주의사항
- 스크립트에서는 alias(k)가 아닌 전체 명령어(kubectl)를 사용해야 합니다.
- metrics-server가 올바르게 설정되어 있어야 합니다.
- Pod 리소스 사용량 확인 시 --containers 옵션으로 컨테이너별 상세 정보를 볼 수 있습니다.
