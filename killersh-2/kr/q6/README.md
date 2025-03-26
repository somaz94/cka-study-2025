# Kubelet 문제 해결

현재 상황:
1. cka1024 노드의 kubelet이 실행되지 않음
2. kubelet을 복구하고 노드를 Ready 상태로 만들어야 함
3. nginx:1-alpine 이미지를 사용하는 success Pod 생성 필요

<br/>

## 1. Kubelet 상태 확인
```bash
# 노드 상태 확인
kubectl get node
# 결과: 연결 거부 에러 발생

# kubelet 프로세스 확인
sudo -i
ps aux | grep kubelet

# kubelet 서비스 상태 확인
service kubelet status
# 결과: inactive (dead) 상태
```

<br/>

## 2. Kubelet 실행 경로 확인
```bash
# kubelet 실행 파일 위치 확인
which kubelet
# 결과: /usr/bin/kubelet

# 서비스 설정의 실행 경로 확인
cat /usr/lib/systemd/system/kubelet.service.d/10-kubeadm.conf
# 문제: /usr/local/bin/kubelet로 잘못된 경로 설정됨
```

<br/>

## 3. Kubelet 설정 수정
```bash
# 설정 파일 수정
vim /usr/lib/systemd/system/kubelet.service.d/10-kubeadm.conf
# ExecStart=/usr/local/bin/kubelet를
# ExecStart=/usr/bin/kubelet로 수정

# systemd 재로드 및 kubelet 재시작
systemctl daemon-reload
service kubelet restart

# 상태 확인
service kubelet status
ps aux | grep kubelet
```

<br/>

## 4. 노드 상태 확인
```bash
# 노드 상태 확인 (Ready 상태가 될 때까지 대기)
kubectl get nodes
```

<br/>

## 5. Success Pod 생성
```bash
# Pod 생성
kubectl run success --image=nginx:1-alpine

# Pod 상태 확인
kubectl get pod success -o wide
```

<br/>

## 문제 해결 단계:
1. kubelet이 inactive 상태임을 확인
2. kubelet 실행 파일의 실제 위치(/usr/bin/kubelet) 확인
3. 서비스 설정의 잘못된 경로(/usr/local/bin/kubelet)를 수정
4. systemd 재로드 및 서비스 재시작
5. 노드가 Ready 상태가 된 후 Pod 생성

<br/>

## 주의사항:
1. kubelet 서비스 설정 파일의 경로가 정확해야 함
2. systemd 재로드가 필요함
3. 노드가 Ready 상태가 될 때까지 기다려야 함
4. 노드에 taint가 없으므로 추가 toleration 불필요
