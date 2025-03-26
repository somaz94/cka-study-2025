# Service CIDR 변경하기

<br/>

## 문제 설명
`ssh cka9412`에서 다음 작업을 수행하세요:

1. Namespace default에 `httpd:2-alpine` 이미지를 사용하는 Pod `check-ip` 생성
2. 포트 80으로 ClusterIP Service `check-ip-service` 생성하고 IP 기록
3. 클러스터의 Service CIDR을 `11.96.0.0/12`로 변경
4. 동일한 Pod를 가리키는 두 번째 Service `check-ip-service2` 생성
   - 새로운 Service는 새 CIDR 범위의 IP를 받아야 함

<br/>

## 해결 방법

### 1. Pod 생성 및 첫 번째 Service 노출
```bash
# Pod 생성
k run check-ip --image=httpd:2-alpine

# Service 생성
k expose pod check-ip --name check-ip-service --port 80

# Service IP 확인
k get svc
NAME               TYPE        CLUSTER-IP      EXTERNAL-IP   PORT(S)   AGE
check-ip-service   ClusterIP   10.109.84.110   <none>        80/TCP    13s
kubernetes         ClusterIP   10.96.0.1       <none>        443/TCP   9d
```

<br/>

### 2. kube-apiserver 설정 변경
```bash
# root 권한으로 전환
sudo -i

# kube-apiserver 매니페스트 수정
vim /etc/kubernetes/manifests/kube-apiserver.yaml

# 변경 내용
spec:
  containers:
  - command:
    - kube-apiserver
    ...
    - --service-cluster-ip-range=11.96.0.0/12     # CIDR 변경
    ...

# apiserver Pod 재시작 확인
watch crictl ps
kubectl -n kube-system get pod | grep api
```

<br/>

### 3. kube-controller-manager 설정 변경
```bash
# controller manager 매니페스트 수정
vim /etc/kubernetes/manifests/kube-controller-manager.yaml

# 변경 내용
spec:
  containers:
  - command:
    - kube-controller-manager
    ...
    - --service-cluster-ip-range=11.96.0.0/12     # CIDR 변경
    ...

# controller manager Pod 재시작 확인
watch crictl ps
kubectl -n kube-system get pod | grep controller
```

<br/>

### 4. 두 번째 Service 생성 및 확인
```bash
# 두 번째 Service 생성
k expose pod check-ip --name check-ip-service2 --port 80

# Service 및 Endpoints 확인
k get svc,ep
NAME                        TYPE        CLUSTER-IP      EXTERNAL-IP   PORT(S)   AGE
service/check-ip-service    ClusterIP   10.109.84.110   <none>        80/TCP    5m55s
service/check-ip-service2   ClusterIP   11.105.52.114   <none>        80/TCP    29s
service/kubernetes          ClusterIP   10.96.0.1       <none>        443/TCP   9d

NAME                          ENDPOINTS             AGE
endpoints/check-ip-service    10.44.0.3:80          5m55s
endpoints/check-ip-service2   10.44.0.3:80          29s
endpoints/kubernetes          192.168.100.21:6443   9d
```

<br/>

## 주의사항
- Service CIDR 변경은 클러스터의 핵심 구성요소를 수정하는 작업입니다
- kube-apiserver와 kube-controller-manager 모두 수정해야 합니다
- 변경 후 컴포넌트가 정상적으로 재시작될 때까지 기다려야 합니다
- 기존 Service의 IP는 변경되지 않으며, 새로 생성되는 Service만 새 CIDR 범위에서 IP를 할당받습니다
