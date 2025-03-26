# ETCD 정보 확인하기

<br/>

## 문제 설명
`ssh cka9412`에서 etcd에 대한 다음 정보를 찾아서 `/opt/course/p1/etcd-info.txt`에 작성하세요:

1. 서버 프라이빗 키 위치
2. 서버 인증서 만료 날짜
3. 클라이언트 인증서 인증 활성화 여부

<br/>

## 해결 방법

### 1. 클러스터 노드 확인
```bash
k get node
NAME            STATUS   ROLES           AGE   VERSION
cka9412         Ready    control-plane   9d    v1.32.0
cka9412-node1   Ready    <none>          9d    v1.32.0
```

<br/>

### 2. etcd Pod 확인
```bash
# root 권한으로 전환
sudo -i

# etcd pod 확인
k -n kube-system get pod
NAME                                READY   STATUS    RESTARTS     AGE
etcd-cka9412                       1/1     Running   0            9d
...
```

<br/>

### 3. Static Pod 매니페스트 확인
```bash
# 매니페스트 파일 찾기
find /etc/kubernetes/manifests/
/etc/kubernetes/manifests/
/etc/kubernetes/manifests/etcd.yaml
...

# etcd 설정 확인
vim /etc/kubernetes/manifests/etcd.yaml
```

<br/>

### 4. etcd 설정 분석
```yaml
# /etc/kubernetes/manifests/etcd.yaml
spec:
  containers:
  - command:
    - etcd
    - --cert-file=/etc/kubernetes/pki/etcd/server.crt
    - --client-cert-auth=true                        # 클라이언트 인증 활성화됨
    - --key-file=/etc/kubernetes/pki/etcd/server.key # 서버 프라이빗 키 위치
```

<br/>

### 5. 인증서 만료일 확인
```bash
# 서버 인증서 만료일 확인
openssl x509 -noout -text -in /etc/kubernetes/pki/etcd/server.crt | grep Validity -A2
        Validity
            Not Before: Oct 29 14:14:27 2024 GMT
            Not After : Oct 29 14:19:27 2025 GMT
```

<br/>

### 6. 정보 저장
```bash
# 정보를 파일에 저장
cat > /opt/course/p1/etcd-info.txt << EOF
Server private key location: /etc/kubernetes/pki/etcd/server.key
Server certificate expiration date: Oct 29 14:19:27 2025 GMT
Is client certificate authentication enabled: yes
EOF
```

<br/>

## 주의사항
- etcd는 쿠버네티스의 핵심 데이터 저장소입니다
- 보안 관련 정보는 신중하게 다루어야 합니다
- Static Pod로 실행되는 etcd의 설정은 매니페스트 파일에서 확인할 수 있습니다
- 인증서 관련 작업은 root 권한이 필요합니다
