# 클러스터 정보 조사

<br/>

## 문제 설명
ssh cka8448 서버에 접속하여 다음 클러스터 정보를 찾아내야 합니다:

1. 컨트롤플레인 노드는 몇 개가 있나요?
2. 워커 노드는 몇 개가 있나요?
3. Service CIDR은 무엇인가요?
4. 어떤 네트워킹(또는 CNI 플러그인)이 구성되어 있으며, 해당 설정 파일은 어디에 있나요?
5. cka8448에서 실행되는 스태틱 파드는 어떤 접미사를 가지나요?

답변은 `/opt/course/14/cluster-info` 파일에 다음 형식으로 작성하세요:
```bash
# /opt/course/14/cluster-info
1: [답변]
2: [답변]
3: [답변]
4: [답변]
5: [답변]
```

<br/>

## 1. 컨트롤플레인 및 워커 노드 수 확인
```bash
kubectl get node
NAME      STATUS   ROLES           AGE   VERSION
cka8448   Ready    control-plane   71m   v1.32.1
```
- 컨트롤플레인 노드: 1개
- 워커 노드: 0개

<br/>

## 2. Service CIDR 확인
```bash
sudo cat /etc/kubernetes/manifests/kube-apiserver.yaml | grep service-cluster-ip-range
    - --service-cluster-ip-range=10.96.0.0/12
```

<br/>

## 3. CNI 플러그인 확인
```bash
# CNI 설정 파일 찾기
sudo find /etc/cni/net.d/
/etc/cni/net.d/
/etc/cni/net.d/.kubernetes-cni-keep
/etc/cni/net.d/10-weave.conflist
/etc/cni/net.d/87-podman-bridge.conflist

# Weave 설정 확인
sudo cat /etc/cni/net.d/10-weave.conflist
{
    "cniVersion": "0.3.0",
    "name": "weave",
    "plugins": [
        {
            "name": "weave",
            "type": "weave-net",
            "hairpinMode": true
        },
        {
            "type": "portmap",
            "capabilities": {"portMappings": true},
            "snat": true
        }
    ]
}
```

<br/>

## 4. Static Pod 접미사
- Static Pod는 노드 이름을 접미사로 가짐
- 이 경우 `-cka8448`가 접미사

## 5. 결과 파일 작성
```bash
cat << EOF > /opt/course/14/cluster-info
1: 1
2: 0
3: 10.96.0.0/12
4: Weave, /etc/cni/net.d/10-weave.conflist
5: -cka8448
EOF
```

<br/>

## 설명:
1. 컨트롤플레인 노드는 1개만 있고 워커 노드는 없음
2. Service CIDR은 kube-apiserver 매니페스트에서 확인
3. Weave CNI가 설치되어 있으며 설정은 /etc/cni/net.d/10-weave.conflist에 위치
4. Static Pod는 노드 이름(-cka8448)을 접미사로 가짐
