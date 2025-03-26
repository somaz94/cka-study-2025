# 클러스터 이벤트 로깅 실습

<br/>

## 문제 설명
ssh cka6016 서버에서 다음 작업을 수행하세요:

1. 전체 클러스터의 최신 이벤트를 시간순(metadata.creationTimestamp)으로 보여주는 kubectl 명령어를 `/opt/course/15/cluster_events.sh` 파일에 작성하세요.

2. kube-proxy Pod를 삭제하고 이로 인해 발생한 이벤트를 `/opt/course/15/pod_kill.log` 파일에 기록하세요.

3. kube-proxy Pod의 containerd 컨테이너를 수동으로 종료하고 발생한 이벤트를 `/opt/course/15/container_kill.log` 파일에 기록하세요.

<br/>

## 1. 클러스터 이벤트 조회 명령어 작성

```bash
# /opt/course/15/cluster_events.sh 파일 생성
cat << 'EOF' > /opt/course/15/cluster_events.sh
kubectl get events -A --sort-by=.metadata.creationTimestamp
EOF

# 실행 권한 부여
chmod +x /opt/course/15/cluster_events.sh
```

<br/>

## 2. kube-proxy Pod 삭제 및 이벤트 로깅

```bash
# 현재 실행 중인 kube-proxy Pod 확인
kubectl -n kube-system get pod -l k8s-app=kube-proxy -owide

# Pod 삭제
kubectl -n kube-system delete pod kube-proxy-lf2fs

# 발생한 이벤트를 파일에 저장
cat << 'EOF' > /opt/course/15/pod_kill.log
kube-system   12s         Normal    Killing             pod/kube-proxy-lf2fs                    Stopping container kube-proxy
kube-system   12s         Normal    SuccessfulCreate    daemonset/kube-proxy                    Created pod: kube-proxy-wb4tb
kube-system   11s         Normal    Scheduled           pod/kube-proxy-wb4tb                    Successfully assigned kube-system/kube-proxy-wb4tb to cka6016
kube-system   11s         Normal    Pulled              pod/kube-proxy-wb4tb                    Container image "registry.k8s.io/kube-proxy:v1.32.1" already present on machine
kube-system   11s         Normal    Created             pod/kube-proxy-wb4tb                    Created container kube-proxy
kube-system   11s         Normal    Started             pod/kube-proxy-wb4tb                    Started container kube-proxy
default       10s         Normal    Starting            node/cka6016  
EOF
```

<br/>

## 3. containerd 컨테이너 수동 종료 및 이벤트 로깅

```bash
# root 권한으로 전환
sudo -i

# kube-proxy 컨테이너 ID 확인
crictl ps | grep kube-proxy

# 컨테이너 강제 종료
crictl rm --force 2fd052f1fcf78

# 새로운 컨테이너가 생성되었는지 확인
crictl ps | grep kube-proxy

# 발생한 이벤트를 파일에 저장
cat << 'EOF' > /opt/course/15/container_kill.log
kube-system   21s         Normal    Created             pod/kube-proxy-wb4tb                    Created container kube-proxy
kube-system   21s         Normal    Started             pod/kube-proxy-wb4tb                    Started container kube-proxy
default       90s         Normal    Starting            node/cka6016                            
default       20s         Normal    Starting            node/cka6016         
EOF
```

<br/>

## 주요 차이점 설명:
1. Pod 삭제 시:
   - DaemonSet 컨트롤러가 새로운 Pod 생성
   - Pod 스케줄링, 컨테이너 생성 등 더 많은 이벤트 발생

2. 컨테이너 수동 종료 시:
   - Pod 객체는 그대로 유지
   - 컨테이너만 재생성되어 더 적은 이벤트 발생
   - kubelet이 자동으로 새 컨테이너 생성

<br/>

## 참고사항:
- crictl 대신 docker 명령어도 사용 가능
- 모든 이벤트는 시간순으로 정렬됨
- 실제 환경에서는 노드에 따라 SSH 접속이 필요할 수 있음
