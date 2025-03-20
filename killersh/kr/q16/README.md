# CoreDNS 설정 업데이트하기

<br/>

## 문제 설명
`ssh cka5774`에서 다음 작업을 수행합니다:

CoreDNS 설정을 다음과 같이 업데이트하세요:

1. 기존 설정 YAML을 `/opt/course/16/coredns_backup.yaml`에 백업하세요
2. `SERVICE.NAMESPACE.custom-domain`이 `SERVICE.NAMESPACE.cluster.local`과 동일하게 작동하도록 CoreDNS 설정을 업데이트하세요

테스트는 `busybox:1` 이미지를 사용하는 Pod에서 다음 명령어로 수행할 수 있습니다:
```bash
nslookup kubernetes.default.svc.cluster.local
nslookup kubernetes.default.svc.custom-domain
```

<br/>

## 해결 방법

### 1. 현재 상태 확인
```bash
# CoreDNS Pod 확인
k -n kube-system get deploy,pod
NAME                             READY   UP-TO-DATE   AVAILABLE   AGE
deployment.apps/coredns          2/2     2            2           42h

NAME                                  READY   STATUS    RESTARTS      AGE
pod/coredns-74f75f8b69-c4z47          1/1     Running   0             42h
pod/coredns-74f75f8b69-wsnfr          1/1     Running   0             42h
```

<br/>

### 2. CoreDNS ConfigMap 백업
```bash
# ConfigMap 백업
k -n kube-system get cm coredns -oyaml > /opt/course/16/coredns_backup.yaml
```

### 3. CoreDNS 설정 업데이트
```yaml
# CoreDNS ConfigMap 수정
apiVersion: v1
data:
  Corefile: |
    .:53 {
        errors
        health {
           lameduck 5s
        }
        ready
        kubernetes custom-domain cluster.local in-addr.arpa ip6.arpa {  # custom-domain 추가
           pods insecure
           fallthrough in-addr.arpa ip6.arpa
           ttl 30
        }
        prometheus :9153
        forward . /etc/resolv.conf {
           max_concurrent 1000
        }
        cache 30 {
           disable success cluster.local
           disable denial cluster.local
        }
        loop
        reload
        loadbalance
    }
```

<br/>

### 4. 설정 적용 및 테스트
```bash
# CoreDNS Deployment 재시작
k -n kube-system rollout restart deploy coredns

# Pod 생성 CLI (CLI 방식)
k run bb --image=busybox:1 -- sh -c 'sleep 1d'

# Pod 생성 yaml (yaml 방식)
apiVersion: v1
kind: Pod
metadata:
  name: bb
spec:
  containers:
  - name: busybox
    image: busybox:1
    command: ["sh", "-c", "sleep 1d"]


k exec -it bb -- sh

# DNS 조회 테스트
/ # nslookup kubernetes.default.svc.custom-domain
Server:         10.96.0.10
Address:        10.96.0.10:53
Name:   kubernetes.default.svc.custom-domain
Address: 10.96.0.1

/ # nslookup kubernetes.default.svc.cluster.local
Server:         10.96.0.10
Address:        10.96.0.10:53
Name:   kubernetes.default.svc.cluster.local
Address: 10.96.0.1
```

<br/>

### 5. 백업에서 복구 방법
```bash
# 변경사항 확인
k diff -f /opt/course/16/coredns_backup.yaml

# 백업에서 복구
k delete -f /opt/course/16/coredns_backup.yaml
k apply -f /opt/course/16/coredns_backup.yaml
k -n kube-system rollout restart deploy coredns
```

<br/>

## 주의사항
- CoreDNS 설정 변경 전 반드시 백업을 수행하세요
- 설정 변경 후 CoreDNS Deployment를 재시작해야 합니다
- 설정 오류 시 DNS 해석이 작동하지 않을 수 있으므로 신중하게 수정하세요
- 백업 파일이 있어야만 복구가 가능합니다
