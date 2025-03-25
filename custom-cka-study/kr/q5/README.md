# 사용자 인증 및 권한 관리 구성

<br/>

## 1. 인증서 생성
```bash
# 디렉토리 생성
mkdir -p /opt/certificates

# 개인키 생성
openssl genrsa -out /opt/certificates/developer-admin.key 2048

# CSR 생성
openssl req -new -key /opt/certificates/developer-admin.key \
  -out /opt/certificates/developer-admin.csr \
  -subj "/CN=developer-admin/O=devops-team"

# CSR을 base64로 인코딩
CSR=$(cat /opt/certificates/developer-admin.csr | base64 | tr -d '\n')

# CertificateSigningRequest 생성
cat <<EOF | kubectl apply -f -
apiVersion: certificates.k8s.io/v1
kind: CertificateSigningRequest
metadata:
  name: developer-admin-csr
spec:
  groups:
  - devops-team
  request: $CSR
  signerName: kubernetes.io/kube-apiserver-client
  expirationSeconds: 31536000  # 365일
  usages:
  - client auth
EOF

# CSR 승인
kubectl certificate approve developer-admin-csr

# 인증서 추출
kubectl get csr developer-admin-csr -o jsonpath='{.status.certificate}' | base64 -d > /opt/certificates/developer-admin.crt
```

<br/>

## 2. 네임스페이스 및 RBAC 설정
```yaml
# 네임스페이스 생성
apiVersion: v1
kind: Namespace
metadata:
  name: development
---
# Role 생성
apiVersion: rbac.authorization.k8s.io/v1
kind: Role
metadata:
  name: developer-admin-role
  namespace: development
rules:
- apiGroups: ["apps", ""]
  resources: ["deployments", "pods", "services"]
  verbs: ["*"]
- apiGroups: [""]
  resources: ["configmaps", "secrets"]
  verbs: ["get", "list", "watch"]
---
# RoleBinding 생성
apiVersion: rbac.authorization.k8s.io/v1
kind: RoleBinding
metadata:
  name: developer-admin-binding
  namespace: development
subjects:
- kind: User
  name: developer-admin
  apiGroup: rbac.authorization.k8s.io
roleRef:
  kind: Role
  name: developer-admin-role
  apiGroup: rbac.authorization.k8s.io
```

<br/>

## 3. kubeconfig 설정
```bash
# 클러스터 정보 가져오기
CLUSTER_CA=$(kubectl config view --raw -o jsonpath='{.clusters[0].cluster.certificate-authority-data}')
CLUSTER_SERVER=$(kubectl config view --raw -o jsonpath='{.clusters[0].cluster.server}')

# kubeconfig 파일 생성
cat <<EOF > /opt/certificates/developer-admin.kubeconfig
apiVersion: v1
kind: Config
clusters:
- cluster:
    certificate-authority-data: ${CLUSTER_CA}
    server: ${CLUSTER_SERVER}
  name: kubernetes
contexts:
- context:
    cluster: kubernetes
    namespace: development
    user: developer-admin
  name: developer-context
current-context: developer-context
users:
- name: developer-admin
  user:
    client-certificate: /opt/certificates/developer-admin.crt
    client-key: /opt/certificates/developer-admin.key
EOF
```

<br/>

## 4. 권한 테스트
```bash
# 테스트 결과 파일 생성
echo "Developer Admin 권한 테스트 결과" > /opt/developer-admin-tests.txt
echo "--------------------------------" >> /opt/developer-admin-tests.txt

# Deployment 생성 테스트
echo -e "\n1. Deployment 생성 테스트:" >> /opt/developer-admin-tests.txt
kubectl --kubeconfig=/opt/certificates/developer-admin.kubeconfig \
  create deployment test-deploy --image=nginx -n development >> /opt/developer-admin-tests.txt 2>&1

# ConfigMap 읽기 테스트
echo -e "\n2. ConfigMap 읽기 테스트:" >> /opt/developer-admin-tests.txt
kubectl --kubeconfig=/opt/certificates/developer-admin.kubeconfig \
  get configmaps -n development >> /opt/developer-admin-tests.txt 2>&1

# Secret 읽기 테스트
echo -e "\n3. Secret 읽기 테스트:" >> /opt/developer-admin-tests.txt
kubectl --kubeconfig=/opt/certificates/developer-admin.kubeconfig \
  get secrets -n development >> /opt/developer-admin-tests.txt 2>&1

# Namespace 생성 테스트
echo -e "\n4. Namespace 생성 테스트 (실패 예상):" >> /opt/developer-admin-tests.txt
kubectl --kubeconfig=/opt/certificates/developer-admin.kubeconfig \
  create namespace test-ns >> /opt/developer-admin-tests.txt 2>&1
```

<br/>

## 주의사항:
1. 인증서 생성 시 올바른 CN과 그룹 설정 확인
2. 인증서 파일의 권한 설정 (600)
3. kubeconfig 파일의 경로가 절대 경로인지 확인
4. RBAC 규칙이 정확하게 설정되었는지 검증
