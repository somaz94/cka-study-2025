# 노드 유지보수

<br/>

## 해결 방법

### 1. 현재 노드 상태 확인
```bash
# 노드 목록 확인
kubectl get nodes

# 노드에서 실행 중인 파드 확인
kubectl get pods --all-namespaces -o wide | grep <node-name>
```

<br/>

### 2. 노드 스케줄 불가능 설정 및 파드 이동
```bash
# 노드를 스케줄 불가능 상태로 변경
kubectl cordon <node-name>

# 노드의 파드들을 다른 노드로 안전하게 이동
kubectl drain <node-name> \
  --ignore-daemonsets \
  --delete-emptydir-data \
  --force
```

<br/>

### 3. 시스템 업그레이드 시뮬레이션
```bash
# 실제 환경에서는 여기서 업그레이드 작업 수행
echo "시스템 업그레이드 시뮬레이션 시작..."
sleep 30
echo "시스템 업그레이드 시뮬레이션 완료"
```

<br/>

### 4. 노드 재활성화
```bash
# 노드를 다시 스케줄 가능 상태로 변경
kubectl uncordon <node-name>

# 노드 상태 확인
kubectl get nodes
```

<br/>

### 5. 검증
```bash
# 노드 상태 확인
kubectl describe node <node-name> | grep Taint

# 새로운 파드가 노드에 스케줄되는지 확인
kubectl run test-pod --image=nginx
kubectl get pod test-pod -o wide
```

<br/>

## 주요 옵션 설명
- `--ignore-daemonsets`: DaemonSet이 관리하는 파드는 무시
- `--delete-emptydir-data`: emptyDir 볼륨을 사용하는 파드 삭제 허용
- `--force`: 관리되지 않는 파드도 삭제

<br/>

## 주의사항
1. drain 전에 중요 워크로드의 백업 확인
2. PodDisruptionBudget 설정 확인
3. 충분한 리소스가 있는 다른 노드 확보
4. 실제 업그레이드 작업 시 충분한 시간 확보
5. 문제 발생 시 빠른 롤백 계획 수립
