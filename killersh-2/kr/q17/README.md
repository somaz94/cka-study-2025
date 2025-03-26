# 오퍼레이터, CRD, RBAC, Kustomize

<br/>

## 문제 설명
ssh cka6016 서버에서 다음 작업을 수행하세요:

`/opt/course/17/operator`에 있는 Kustomize 설정이 있습니다. 이 설정은 여러 CRD와 함께 작동하는 오퍼레이터를 설치합니다. 다음 명령어로 배포되었습니다:

```bash
kubectl kustomize /opt/course/17/operator/prod | kubectl apply -f -
```

Kustomize 기본 설정에서 다음 변경사항을 수행하세요:

1. 오퍼레이터가 특정 CRD를 나열해야 합니다. 로그를 확인하여 어떤 CRD가 필요한지 확인하고 Role operator-role의 권한을 조정하세요.
2. student4라는 이름의 새로운 Student 리소스를 추가하세요(이름과 설명은 자유롭게 지정).
3. 변경된 Kustomize 설정을 prod에 배포하세요.

<br/>

## 1. Kustomize 설정 구조 확인

```bash
# 디렉토리 구조 확인
cd /opt/course/17/operator
ls
# 결과:
# base  prod

# base 설정 확인
kubectl kustomize base
# 결과: CRD, ServiceAccount 등의 기본 리소스 확인

# prod 설정 확인
kubectl kustomize prod
# 결과: namespace가 operator-prod로 설정된 리소스 확인
```

<br/>

## 2. 오퍼레이터 로그 확인 및 문제 파악

```bash
# Pod 상태 확인
kubectl -n operator-prod get pod
NAME                        READY   STATUS    RESTARTS   AGE
operator-7f4f58d4d9-v6ftw   1/1     Running   0          6m9s

# 로그 확인
kubectl -n operator-prod logs operator-7f4f58d4d9-v6ftw
# 결과:
# + kubectl get students
# Error: students.education.killer.sh is forbidden...
# + kubectl get classes
# Error: classes.education.killer.sh is forbidden...
```

<br/>

## 3. Role 권한 조정

```bash
# Role 템플릿 생성
kubectl -n operator-prod create role operator-role \
  --verb list \
  --resource student \
  --resource class \
  -oyaml --dry-run=client

# base/rbac.yaml 수정
vim base/rbac.yaml
```

```yaml
# base/rbac.yaml
apiVersion: rbac.authorization.k8s.io/v1
kind: Role
metadata:
  name: operator-role
  namespace: default
rules:
- apiGroups:
  - education.killer.sh
  resources:
  - students
  - classes
  verbs:
  - list
---
apiVersion: rbac.authorization.k8s.io/v1
kind: RoleBinding
metadata:
  name: operator-rolebinding
  namespace: default
subjects:
  - kind: ServiceAccount
    name: operator
    namespace: default
roleRef:
  kind: Role
  name: operator-role
  apiGroup: rbac.authorization.k8s.io
```