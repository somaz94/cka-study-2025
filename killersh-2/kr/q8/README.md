# 컨트롤플레인 컴포넌트 조사

문제에서 컨트롤플레인 컴포넌트들의 실행/설치 방식을 확인하고, DNS 애플리케이션의 정보를 찾아야 합니다.

<br/>

## 1. Kubelet 확인
```bash
# systemd 서비스 확인
ps aux | grep kubelet

# systemd 관련 파일 확인
find /usr/lib/systemd | grep kube

# 서비스 상태 확인
service kubelet status
```

<br/>

## 2. Static Pod 확인
```bash
# 매니페스트 디렉토리 확인
ls /etc/kubernetes/manifests/

# 결과:
# - kube-apiserver.yaml
# - kube-controller-manager.yaml
# - kube-scheduler.yaml
# - etcd.yaml
```

<br/>

## 3. DNS 애플리케이션 확인
```bash
# kube-system 네임스페이스의 Pod 확인
kubectl -n kube-system get pod -o wide

# DNS 관련 Deployment 확인
kubectl -n kube-system get deploy
# 결과: coredns Deployment 확인
```

<br/>

## 4. 결과 파일 작성
```bash
# 디렉토리 생성
mkdir -p /opt/course/8

# 결과 파일 작성
cat << EOF > /opt/course/8/controlplane-components.txt
kubelet: process
kube-apiserver: static-pod
kube-scheduler: static-pod
kube-controller-manager: static-pod
etcd: static-pod
dns: pod coredns
EOF
```

<br/>

## 컴포넌트 유형 설명:
1. kubelet: systemd 프로세스로 실행
2. kube-apiserver: Static Pod로 실행 (/etc/kubernetes/manifests/)
3. kube-scheduler: Static Pod로 실행
4. kube-controller-manager: Static Pod로 실행
5. etcd: Static Pod로 실행
6. DNS(coredns): 일반 Pod로 실행 (Deployment로 관리)

<br/>

## 주의사항:
1. kubelet은 유일하게 systemd 프로세스로 실행
2. Static Pod는 노드 이름이 접미사로 붙음 (-cka8448)
3. coredns는 Deployment로 관리되는 일반 Pod
4. 결과 파일 형식을 정확히 지켜야 함
