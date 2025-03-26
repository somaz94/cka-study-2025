# ETCD 버전 확인 및 스냅샷 생성

문제에서 다음 작업을 수행해야 합니다:
1. etcd 버전 정보를 /opt/course/7/etcd-version에 저장
2. etcd 스냅샷을 생성하여 /opt/course/7/etcd-snapshot.db에 저장

<br/>

## 1. 작업 디렉토리 생성
```bash
# 디렉토리 생성
sudo mkdir -p /opt/course/7
```

<br/>

## 2. ETCD 버전 확인
```bash
# ETCD Pod 확인
kubectl -n kube-system get pod | grep etcd

# ETCD 버전 확인 및 저장
kubectl -n kube-system exec etcd-cka2560 -- etcd --version > /opt/course/7/etcd-version

# 결과 확인
cat /opt/course/7/etcd-version
```

<br/>

## 3. ETCD 인증서 정보 확인
```bash
# ETCD 매니페스트 확인
cat /etc/kubernetes/manifests/etcd.yaml

# API 서버의 ETCD 연결 설정 확인
cat /etc/kubernetes/manifests/kube-apiserver.yaml | grep etcd
```

<br/>

## 4. ETCD 스냅샷 생성
```bash
# 스냅샷 생성
ETCDCTL_API=3 etcdctl snapshot save /opt/course/7/etcd-snapshot.db \
--cacert /etc/kubernetes/pki/etcd/ca.crt \
--cert /etc/kubernetes/pki/etcd/server.crt \
--key /etc/kubernetes/pki/etcd/server.key

# 결과 확인
ls -l /opt/course/7/etcd-snapshot.db