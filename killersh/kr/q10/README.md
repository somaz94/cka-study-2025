# RBAC로 ServiceAccount 권한 설정하기

<br/>

## 문제 설명
`ssh cka3962`에서 다음 작업을 수행합니다:

Namespace `project-hamster`에 새로운 ServiceAccount `processor`를 생성하세요. Role과 RoleBinding도 같은 이름 `processor`로 생성하세요. 이 ServiceAccount는 해당 Namespace에서 Secret과 ConfigMap만 생성할 수 있어야 합니다.

<br/>

## RBAC 리소스 이해하기

### RBAC 조합
1. Role + RoleBinding (단일 Namespace에서 사용 가능, 단일 Namespace에 적용)
2. ClusterRole + ClusterRoleBinding (클러스터 전체에서 사용 가능, 클러스터 전체에 적용)
3. ClusterRole + RoleBinding (클러스터 전체에서 사용 가능, 단일 Namespace에 적용)
4. Role + ClusterRoleBinding (불가능: 단일 Namespace에서 사용 가능, 클러스터 전체에 적용)

<br/>

## 해결 방법

### 1. ServiceAccount 생성
```bash
k -n project-hamster create sa processor
```

### 2. Role 생성

명령어로 생성:
```bash
k -n project-hamster create role processor \
  --verb=create \
  --resource=secret \
  --resource=configmap
```

또는 YAML로 생성:
```yaml
# processor-role.yaml
apiVersion: rbac.authorization.k8s.io/v1
kind: Role
metadata:
  name: processor
  namespace: project-hamster
rules:
- apiGroups: [""]  # "" 는 core API group을 의미
  resources: ["secrets", "configmaps"]
  verbs: ["create"]
```

```bash
# YAML 적용
kubectl apply -f processor-role.yaml
```

<br/>

### 3. RoleBinding 생성
```bash
k -n project-hamster create rolebinding processor \
  --role processor \
  --serviceaccount project-hamster:processor
```

생성된 RoleBinding:
```yaml
apiVersion: rbac.authorization.k8s.io/v1
kind: RoleBinding
metadata:
  name: processor
  namespace: project-hamster
roleRef:
  apiGroup: rbac.authorization.k8s.io
  kind: Role
  name: processor
subjects:
- kind: ServiceAccount
  name: processor
  namespace: project-hamster
```

<br/>

### 4. 권한 확인
```bash
# Secret 생성 권한 확인
k -n project-hamster auth can-i create secret \
  --as system:serviceaccount:project-hamster:processor

# ConfigMap 생성 권한 확인
k -n project-hamster auth can-i create configmap \
  --as system:serviceaccount:project-hamster:processor

# Pod 생성 권한 확인 (없어야 함)
k -n project-hamster auth can-i create pod \
  --as system:serviceaccount:project-hamster:processor

# Secret 삭제 권한 확인 (없어야 함)
k -n project-hamster auth can-i delete secret \
  --as system:serviceaccount:project-hamster:processor

# ConfigMap 조회 권한 확인 (없어야 함)
k -n project-hamster auth can-i get configmap \
  --as system:serviceaccount:project-hamster:processor
```

<br/>

## 주의사항
- Role은 권한의 범위를 정의합니다 (어디서 사용 가능한지)
- RoleBinding은 권한의 적용을 정의합니다 (어디에 적용되는지)
- ServiceAccount 이름을 RoleBinding에서 정확히 지정해야 합니다
- 필요한 권한만 부여하는 것이 보안상 좋습니다
