# 첫 번째로 종료될 Pod 찾기

<br/>

## 문제 설명
`ssh cka2556`에서 다음 작업을 수행합니다:

`project-c13` Namespace의 모든 Pod를 확인하고, 노드의 리소스(CPU 또는 메모리)가 부족할 때 가장 먼저 종료될 가능성이 있는 Pod의 이름을 찾습니다.

Pod 이름을 `/opt/course/4/pods-terminated-first.txt`에 작성하세요.

<br/>

## 해결 방법

### 1. 수동 접근 방식
```bash
# Pod의 리소스 요청 확인
k -n project-c13 describe pod | less -p Requests

# 또는 다음 명령어 사용
k -n project-c13 describe pod | grep -A 3 -E 'Requests|^Name:'
```

결과:
```bash
Deployment `c13-3cc-runner-heavy`의 Pod들이 리소스 요청을 지정하지 않았음을 확인
```

<br/>

### 2. 정답 작성
```bash
# /opt/course/4/pods-terminated-first.txt 내용
c13-3cc-runner-heavy-65588d7d6-djtv9map
c13-3cc-runner-heavy-65588d7d6-v8kf5map
c13-3cc-runner-heavy-65588d7d6-wwpb4map
```

<br/>

### 3. 자동화 방식 (참고)
```bash
# jsonpath를 사용한 Pod 이름과 리소스 요청 확인
k -n project-c13 get pod -o jsonpath="{range .items[*]} {.metadata.name}{.spec.containers[*].resources}{'\n'}"

# QoS 클래스 확인
k get pods -n project-c13 -o jsonpath="{range .items[*]}{.metadata.name} {.status.qosClass}{'\n'}"
```

<br/>

## 설명
- 노드의 CPU나 메모리가 한계에 도달하면, Kubernetes는 요청된 것보다 많은 리소스를 사용하는 Pod를 먼저 찾습니다.
- 컨테이너에 리소스 요청/제한이 설정되지 않은 경우, 기본적으로 요청된 것보다 많이 사용하는 것으로 간주됩니다.
- Kubernetes는 정의된 리소스와 제한에 따라 Pod에 Quality of Service 클래스를 할당합니다.
- BestEffort QoS 클래스를 가진 Pod(메모리나 CPU 제한/요청이 정의되지 않은)가 가장 먼저 종료됩니다.

<br/>

## 모범 사례
- 항상 리소스 요청과 제한을 설정하는 것이 좋습니다.
- 적절한 값을 모르는 경우:
  - Prometheus와 같은 메트릭 도구 사용
  - `kubectl top pod` 사용
  - `kubectl exec`로 컨테이너에 접속하여 `top` 등의 도구 사용
