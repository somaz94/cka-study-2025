# ETCD Backup and Restore

<br/>

## 해결 방법

### 1. ETCD 데이터 백업
```bash
# 백업 디렉토리 생성
mkdir -p /opt/backup

# ETCD 스냅샷 생성
ETCDCTL_API=3 etcdctl --endpoints=https://127.0.0.1:2379 \
  --cacert=/etc/kubernetes/pki/etcd/ca.crt \
  --cert=/etc/kubernetes/pki/etcd/server.crt \
  --key=/etc/kubernetes/pki/etcd/server.key \
  snapshot save /opt/backup/etcd-backup.db

# 백업 확인
ETCDCTL_API=3 etcdctl snapshot status /opt/backup/etcd-backup.db --write-out=table
```

<br/>

### 2. 테스트 리소스 생성
```bash
# 네임스페이스 생성
kubectl create namespace pre-restore

# Deployment YAML 생성
cat << EOF > nginx-deploy.yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: nginx
  namespace: pre-restore
  labels:
    app: nginx
spec:
  replicas: 3
  selector:
    matchLabels:
      app: nginx
  template:
    metadata:
      labels:
        app: nginx
    spec:
      containers:
      - name: nginx
        image: nginx:1.22
        ports:
        - containerPort: 80
EOF

# Deployment 생성
kubectl apply -f nginx-deploy.yaml

# 리소스 생성 확인
kubectl -n pre-restore get deploy,pod
```

<br/>

### 3. ETCD 데이터 복구
```bash
# root 권한으로 전환
sudo -i

# ETCD 서비스 중지 (매니페스트 파일 이동)
mv /etc/kubernetes/manifests/etcd.yaml /etc/kubernetes/

# 기존 ETCD 데이터 디렉토리 백업
mv /var/lib/etcd/member /var/lib/etcd/member.bak

# 스냅샷에서 복구
ETCDCTL_API=3 etcdctl snapshot restore /opt/backup/etcd-backup.db \
  --data-dir=/var/lib/etcd

# ETCD 권한 수정
chown -R etcd:etcd /var/lib/etcd

# ETCD 서비스 재시작 (매니페스트 파일 복구)
mv /etc/kubernetes/etcd.yaml /etc/kubernetes/manifests/
```

<br/>

### 4. 복구 확인
```bash
# ETCD Pod가 다시 시작될 때까지 대기
watch crictl ps | grep etcd

# 테스트 리소스가 삭제되었는지 확인
kubectl get ns pre-restore
kubectl -n pre-restore get deploy,pod
```

<br/>

## 주의사항
1. 백업 전 충분한 디스크 공간 확보
2. 복구 시 기존 ETCD 데이터 디렉토리 백업
3. ETCD 복구 후 권한 설정 확인
4. 모든 명령은 컨트롤 플레인 노드에서 실행
5. 필요한 경우 sudo 사용

<br/>

## 검증
- 백업 파일이 생성되었는지 확인
- 테스트 네임스페이스와 디플로이먼트가 성공적으로 생성되었는지 확인
- 복구 후 테스트 리소스가 없어졌는지 확인
- 클러스터의 다른 기능이 정상적으로 작동하는지 확인
