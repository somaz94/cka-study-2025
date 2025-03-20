# Kubernetes 버전 업데이트 및 클러스터 조인

<br/>

## 문제 설명
`ssh cka3962`에서 다음 작업을 수행합니다:

동료가 `cka3962-node1` 노드가 오래된 Kubernetes 버전을 실행 중이며 아직 클러스터에 포함되지 않았다고 알려주었습니다.

1. 노드의 Kubernetes 버전을 컨트롤플레인과 동일한 버전으로 업데이트
2. kubeadm을 사용하여 노드를 클러스터에 추가

> ℹ️ cka3962에서 ssh cka3962-node1을 사용하여 워커 노드에 접속할 수 있습니다.

<br/>

## 해결 방법

### 1. 컨트롤플레인 버전 확인
```bash
k get node
```

결과:
```bash
NAME      STATUS   ROLES           AGE    VERSION
cka3962   Ready    control-plane   169m   v1.32.1
```

<br/>

### 2. 워커 노드 현재 버전 확인
```bash
ssh cka3962-node1
sudo -i

# 버전 확인
kubectl version
kubelet --version     # Kubernetes v1.31.5
kubeadm version      # v1.32.1
```

<br/>

### 3. kubelet과 kubectl 업데이트
```bash
# 패키지 업데이트
apt update

# 설치 가능한 버전 확인
apt show kubectl -a | grep 1.32

# kubeadm 버전 업데이트
sudo apt-mark unhold kubeadm && \
sudo apt-get update && sudo apt-get install -y kubeadm='1.32.1-1.1' && \
sudo apt-mark hold kubeadm

# kubelet 버전 업데이트
sudo apt-mark unhold kubelet kubectl && \
sudo apt-get update && sudo apt-get install -y kubelet=1.32.1-1.1 kubectl=1.32.1-1.1 && \
sudo apt-mark hold kubelet kubectl

# 확인
dpkg -l | grep 'kubeadm\|kubelet\|kubectl'
# kubeadm 1.32-1.1
# kubelet 1.32-1.1
# kubectl 1.32-1.1
```

<br/>

### 4. 클러스터에 노드 추가

컨트롤플레인에서:
```bash
# 새로운 토큰 생성
sudo kubeadm token create --print-join-command

# 토큰 목록 확인(kubeadm token list)
kubeadm token list

# hash 확인(--discovery-token-ca-cert-hash)
openssl x509 -pubkey -in /etc/kubernetes/pki/ca.crt | openssl rsa -pubin -outform der 2>/dev/null | openssl dgst -sha256 -hex | sed 's/^.* //'
```

<br/>

워커 노드에서:
```bash
# kubeadm join 명령어 실행
sudo kubeadm join 192.168.100.31:6443 --token pwq11h.uevwb20rt81e6whd \
  --discovery-token-ca-cert-hash sha256:cb299d7b2025adf683779793a4a0a2051ac7611da668f188770259b0da68376c

# kubelet 상태 확인
sudo systemctl restart kubelet
sudo systemctl status kubelet
```

> ℹ️ kubeadm join에 문제가 있다면 kubeadm reset을 먼저 실행해야 할 수 있습니다.

<br/>

### 5. 노드 상태 확인
```bash
k get node
```

결과:
```bash
NAME            STATUS   ROLES           AGE    VERSION
cka3962         Ready    control-plane   173m   v1.32.1
cka3962-node1   Ready    <none>          29s    v1.32.1
```

### node 문제 발생 시 제거
```bash
kubectl drain cka3962-node1 --ignore-daemonsets --delete-emptydir-data
kubectl delete node cka3962-node1
```

<br/>

### 노드 상태 확인
```bash
k get node
```

### 노드 접속 후 kubelet stop , key 
```bash
sudo systemctl stop kubelet
sudo rm -rf /etc/kubernetes/kubelet.conf
sudo rm -rf /etc/kubernetes/pki/*
```
- 다시 처음부터 위에 작업 반복

<br/>

## 주의사항
- 버전 업그레이드 전 컨트롤플레인 버전 확인
- kubeadm, kubelet, kubectl 모두 동일한 버전으로 업데이트
- 토큰 만료 시간 확인 (기본 23시간)
- 노드가 Ready 상태가 될 때까지 대기
