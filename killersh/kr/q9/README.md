# Pod 내부에서 K8s API 접근하기

<br/>

## 문제 설명
`ssh cka9412`에서 다음 작업을 수행합니다:

Namespace `project-swan`에 ServiceAccount `secret-reader`가 있습니다. 이 ServiceAccount를 사용하는 `nginx:1-alpine` 이미지의 Pod `api-contact`를 생성하세요.

Pod에 접속하여 curl로 Kubernetes API에서 모든 Secret을 수동으로 조회하고, 결과를 `/opt/course/9/result.json`에 저장하세요.

<br/>

## 해결 방법

### 1. ServiceAccount 권한 확인 및 설정
```bash
# ServiceAccount 존재 확인
k get sa -n project-swan secret-reader

# Secret 조회 권한 확인
k auth can-i get secret --as system:serviceaccount:project-swan:secret-reader
```

필요한 경우 ClusterRole과 ClusterRoleBinding 생성:
```yaml
# secret-reader-role.yaml
apiVersion: rbac.authorization.k8s.io/v1
kind: ClusterRole
metadata:
  name: secret-reader-clusterrole
rules:
- apiGroups: [""]
  resources: ["secrets"]
  verbs: ["get", "list"]
---
apiVersion: rbac.authorization.k8s.io/v1
kind: ClusterRoleBinding
metadata:
  name: secret-reader-clusterrolebinding
subjects:
- kind: ServiceAccount
  name: secret-reader
  namespace: project-swan
roleRef:
  kind: ClusterRole
  name: secret-reader-clusterrole
  apiGroup: rbac.authorization.k8s.io
```

### 2. Pod 생성
```yaml
# 9.yaml
apiVersion: v1
kind: Pod
metadata:
  name: api-contact
  namespace: project-swan
spec:
  serviceAccountName: secret-reader
  containers:
  - name: nginx
    image: nginx:1-alpine
    command: ["/bin/sh", "-c", "sleep 3600"]  # Pod 유지를 위해 추가(선택)
```

```bash
# Pod 생성
k apply -f 9.yaml

# Pod 상태 확인
k get pods -n project-swan api-contact
```

### 3. Pod 내부에서 K8s API 접근
```bash
# Pod에 접속
k -n project-swan exec api-contact -it -- sh

# API 서버 및 ServiceAccount 정보 설정
APISERVER=https://kubernetes.default.svc
SERVICEACCOUNT=/var/run/secrets/kubernetes.io/serviceaccount

# 현재 Pod의 네임스페이스 읽기
NAMESPACE=$(cat ${SERVICEACCOUNT}/namespace)

# ServiceAccount 토큰 읽기
TOKEN=$(cat ${SERVICEACCOUNT}/token)

# 내부 인증 기관(CA) 참조
CACERT=${SERVICEACCOUNT}/ca.crt

# 토큰을 사용하여 API 탐색
curl --cacert ${CACERT} --header "Authorization: Bearer ${TOKEN}" -X GET ${APISERVER}/api/v1/secrets
```

### 4. 결과 저장
```bash
# Pod 내부에서 결과 저장
curl --cacert ${CACERT} --header "Authorization: Bearer ${TOKEN}" \
  -X GET ${APISERVER}/api/v1/secrets > result.json

# 결과 파일 복사
exit
k -n project-swan exec api-contact -it -- cat result.json > /opt/course/9/result.json
```

## 주의사항
- ServiceAccount에 적절한 Secret 조회 권한이 있어야 합니다
- Pod이 종료되지 않도록 command를 추가합니다
- kubernetes.default.svc는 K8s API 서비스의 내부 DNS 이름입니다
- API 요청 시 적절한 인증서와 토큰을 사용하는 것이 중요합니다
