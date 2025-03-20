# 인증서 유효 기간 확인하기

## 문제 설명
`ssh cka9412`에서 다음 작업을 수행합니다:

클러스터 인증서에 대해 다음 작업을 수행하세요:
1. openssl 또는 cfssl을 사용하여 kube-apiserver 서버 인증서의 유효 기간을 확인하고, 만료 날짜를 `/opt/course/14/expiration`에 기록하세요
2. kubeadm 명령어로 만료 날짜를 확인하여 두 방법이 동일한 결과를 보이는지 확인하세요
3. kube-apiserver 인증서를 갱신하는 kubeadm 명령어를 `/opt/course/14/kubeadm-renew-certs.sh`에 작성하세요

## 해결 방법

### 1. 인증서 파일 찾기
```bash
# root 권한으로 전환
sudo -i

# apiserver 관련 인증서 찾기
find /etc/kubernetes/pki | grep apiserver
/etc/kubernetes/pki/apiserver-etcd-client.key
/etc/kubernetes/pki/apiserver-kubelet-client.key
/etc/kubernetes/pki/apiserver-etcd-client.crt
/etc/kubernetes/pki/apiserver.key
/etc/kubernetes/pki/apiserver-kubelet-client.crt
/etc/kubernetes/pki/apiserver.crt
```

### 2. openssl로 인증서 만료일 확인
```bash
# openssl로 인증서 확인
openssl x509 -noout -text -in /etc/kubernetes/pki/apiserver.crt | grep Validity -A2
        Validity
            Not Before: Oct 29 14:14:27 2024 GMT
            Not After : Oct 29 14:19:27 2025 GMT

# 만료일 저장
echo "Oct 29 14:19:27 2025 GMT" > /opt/course/14/expiration
```

### 3. kubeadm으로 인증서 만료일 확인 저장
```bash
# kubeadm으로 인증서 확인
kubeadm certs check-expiration > /opt/course/14/expiration

# 결과 확인
cat /opt/course/14/expiration 
CERTIFICATE                EXPIRES                  RESIDUAL TIME   CERTIFICATE AUTHORITY   EXTERNALLY MANAGED
admin.conf                 Oct 29, 2025 14:19 UTC   356d            ca                      no
apiserver                  Oct 29, 2025 14:19 UTC   356d            ca                      no
apiserver-etcd-client      Oct 29, 2025 14:19 UTC   356d            etcd-ca                 no
apiserver-kubelet-client   Oct 29, 2025 14:19 UTC   356d            ca                      no
...
```

### 4. 인증서 갱신 명령어 저장
```bash
# 갱신 명령어 저장
echo "kubeadm certs renew apiserver" > /opt/course/14/kubeadm-renew-certs.sh
```

## 주의사항
- 인증서 확인과 갱신은 root 권한이 필요합니다
- openssl과 kubeadm 두 방법의 결과가 일치하는지 확인해야 합니다
- 인증서 갱신 후에는 kube-apiserver를 재시작해야 할 수 있습니다
- 프로덕션 환경에서는 인증서 갱신 전에 반드시 백업을 수행해야 합니다