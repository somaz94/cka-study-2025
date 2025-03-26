# 네임스페이스와 API 리소스

<br/>

## 문제 설명
ssh cka3200 서버에서 다음 작업을 수행하세요:

1. 모든 네임스페이스 범위의 쿠버네티스 리소스(Pod, Secret, ConfigMap 등)의 이름을 `/opt/course/16/resources.txt` 파일에 작성하세요.

2. project-* 네임스페이스 중에서 가장 많은 Role이 정의된 네임스페이스를 찾아 해당 네임스페이스 이름과 Role 개수를 `/opt/course/16/crowded-namespace.txt` 파일에 작성하세요.

<br/>

## 1. 네임스페이스 범위의 리소스 목록 작성

```bash
# 네임스페이스 범위의 리소스 조회 및 파일 저장
kubectl api-resources --namespaced -o name > /opt/course/16/resources.txt
```

결과 파일 내용:
```plaintext
# /opt/course/16/resources.txt
bindings
configmaps
endpoints
events
limitranges
persistentvolumeclaims
pods
podtemplates
replicationcontrollers
resourcequotas
secrets
serviceaccounts
services
controllerrevisions.apps
daemonsets.apps
deployments.apps
replicasets.apps
statefulsets.apps
localsubjectaccessreviews.authorization.k8s.io
horizontalpodautoscalers.autoscaling
cronjobs.batch
jobs.batch
leases.coordination.k8s.io
endpointslices.discovery.k8s.io
events.events.k8s.io
ingresses.networking.k8s.io
networkpolicies.networking.k8s.io
poddisruptionbudgets.policy
rolebindings.rbac.authorization.k8s.io
roles.rbac.authorization.k8s.io
csistoragecapacities.storage.k8s.io
```

<br/>

## 2. project-* 네임스페이스의 Role 개수 확인

```bash
# 각 project-* 네임스페이스의 Role 개수 확인
kubectl -n project-jinan get role --no-headers | wc -l
# 결과: 0

kubectl -n project-miami get role --no-headers | wc -l
# 결과: 300

kubectl -n project-melbourne get role --no-headers | wc -l
# 결과: 2

kubectl -n project-seoul get role --no-headers | wc -l
# 결과: 10

kubectl -n project-toronto get role --no-headers | wc -l
# 결과: 0

# 결과 파일 작성
echo "project-miami with 300 roles" > /opt/course/16/crowded-namespace.txt
```

<br/>

## 주요 설명:
1. API 리소스 조회:
   - `--namespaced`: 네임스페이스 범위의 리소스만 표시
   - `-o name`: 리소스 이름만 출력
   - 결과를 resources.txt 파일에 저장

2. Role 개수 확인:
   - `--no-headers`: 헤더 행 제외하고 출력
   - `wc -l`: 행 수 계산
   - project-miami가 300개로 가장 많은 Role 보유
   - 결과를 crowded-namespace.txt 파일에 저장

<br/>

## 참고사항:
- kubectl api-resources 명령어로 모든 리소스 목록 확인 가능
- -h 옵션으로 추가 도움말 확인 가능
