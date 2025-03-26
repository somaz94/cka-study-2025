# Kube-Proxy iptables 규칙 확인하기

<br/>

## 문제 설명
`ssh cka2556`에서 kube-proxy가 올바르게 작동하는지 확인하기 위해 다음 작업을 수행하세요:

1. Namespace `project-hamster`에서 `nginx:1-alpine` 이미지를 사용하는 Pod `p2-pod` 생성
2. Pod를 클러스터 내부에서 port 3000->80으로 노출하는 Service `p2-service` 생성
3. 노드 `cka2556`에서 생성된 Service에 대한 iptables 규칙을 `/opt/course/p2/iptables.txt`에 저장
4. Service를 삭제하고 iptables 규칙이 제거되었는지 확인

<br/>

## 해결 방법

### 1. Pod 생성
```bash
# Pod 생성
k -n project-hamster run p2-pod --image=nginx:1-alpine
```

<br/>

### 2. Service 생성
```bash
# Service 생성
k -n project-hamster expose pod p2-pod \
  --name p2-service \
  --port 3000 \
  --target-port 80

# 리소스 확인
k -n project-hamster get pod,svc,ep
NAME                 READY   STATUS    RESTARTS   AGE
pod/p2-pod           1/1     Running   0          2m31s

NAME                 TYPE        CLUSTER-IP       EXTERNAL-IP   PORT(S)    AGE
service/p2-service   ClusterIP   10.105.128.247   <none>        3000/TCP   1s

NAME                   ENDPOINTS       AGE
endpoints/p2-service   10.44.0.31:80   1s
```

<br/>

### 3. kube-proxy 설정 확인 (선택사항)
```bash
# kube-proxy 컨테이너 확인
sudo -i
crictl ps | grep kube-proxy
67cccaf8310a1   505d571f5fd56   9 days ago      Running    kube-proxy ...

# kube-proxy 로그 확인
crictl logs 67cccaf8310a1
I1029 14:10:23.984360       1 server_linux.go:66] "Using iptables proxy"
```

<br/>

### 4. iptables 규칙 확인 및 저장
```bash
# iptables 규칙 확인
iptables-save | grep p2-service
-A KUBE-SEP-55IRFJIRWHLCQ6QX -s 10.44.0.31/32 -m comment --comment "project-hamster/p2-service" -j KUBE-MARK-MASQ
-A KUBE-SEP-55IRFJIRWHLCQ6QX -p tcp -m comment --comment "project-hamster/p2-service" -m tcp -j DNAT --to-destination 10.44.0.31:80
-A KUBE-SERVICES -d 10.105.128.247/32 -p tcp -m comment --comment "project-hamster/p2-service cluster IP" -m tcp --dport 3000 -j KUBE-SVC-U5ZRKF27Y7YDAZTN
-A KUBE-SVC-U5ZRKF27Y7YDAZTN ! -s 10.244.0.0/16 -d 10.105.128.247/32 -p tcp -m comment --comment "project-hamster/p2-service cluster IP" -m tcp --dport 3000 -j KUBE-MARK-MASQ
-A KUBE-SVC-U5ZRKF27Y7YDAZTN -m comment --comment "project-hamster/p2-service -> 10.44.0.31:80" -j KUBE-SEP-55IRFJIRWHLCQ6QX

# 규칙을 파일에 저장
iptables-save | grep p2-service > /opt/course/p2/iptables.txt
```

<br/>

### 5. Service 삭제 및 규칙 제거 확인
```bash
# Service 삭제
k -n project-hamster delete svc p2-service

# iptables 규칙이 제거되었는지 확인
iptables-save | grep p2-service
# 출력이 없어야 함
```

<br/>

## 주의사항
- 쿠버네티스 Service는 기본 설정에서 모든 노드의 iptables 규칙으로 구현됩니다
- Service가 생성, 수정, 삭제되거나 Endpoints가 변경될 때마다 kube-apiserver는 각 노드의 kube-proxy에 연락하여 iptables 규칙을 업데이트합니다
- iptables 규칙 확인은 root 권한이 필요합니다
