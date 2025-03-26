# 스케줄러 중지 및 수동 스케줄링

문제 요구사항:
1. kube-scheduler 임시 중지
2. manual-schedule Pod 생성 및 수동 스케줄링
3. kube-scheduler 재시작 및 정상 작동 확인

<br/>

## 1. kube-scheduler 중지
```bash
# 컨트롤플레인 노드 확인
kubectl get node
# cka5248 (control-plane)
# cka5248-node1

# 스케줄러 상태 확인
sudo -i
kubectl -n kube-system get pod | grep schedule

# 스케줄러 매니페스트 이동으로 중지
cd /etc/kubernetes/manifests/
mv kube-scheduler.yaml ..

# 스케줄러 중지 확인
watch crictl ps
kubectl -n kube-system get pod | grep schedule
```

<br/>

## 2. Pod 생성 및 수동 스케줄링
```bash
# Pod 생성
kubectl run manual-schedule --image=httpd:2-alpine

# 스케줄링 되지 않은 상태 확인
kubectl get pod manual-schedule -o wide
# STATUS: Pending, NODE: <none>

# Pod YAML 수정을 위해 추출
kubectl get pod manual-schedule -o yaml > 9.yaml

# nodeName 필드 추가
# 9.yaml 수정
spec:
  nodeName: cka5248    # 컨트롤플레인 노드 이름 추가

# 수정된 Pod 적용
kubectl replace -f 9.yaml --force

# Pod 상태 확인
kubectl get pod manual-schedule -o wide
```

<br/>

## 3. 스케줄러 재시작 및 검증
```bash
# 스케줄러 매니페스트 복구
cd /etc/kubernetes/manifests/
mv ../kube-scheduler.yaml .

# 스케줄러 실행 확인
kubectl -n kube-system get pod | grep schedule

# 두 번째 Pod 생성
kubectl run manual-schedule2 --image=httpd:2-alpine

# Pod 배포 상태 확인
kubectl get pod -o wide | grep schedule
```

<br/>

## 예상 결과:
1. manual-schedule Pod: cka5248 (컨트롤플레인) 노드에서 실행
2. manual-schedule2 Pod: cka5248-node1 (워커) 노드에서 실행

<br/>

## 주의사항:
1. 스케줄러 중지는 매니페스트 파일 이동으로 수행
2. 수동 스케줄링은 nodeName 필드 사용
3. 컨트롤플레인 노드에 Pod 수동 배치 가능 (taint/toleration 무시)
4. 스케줄러 재시작 후 정상 작동 확인 필요
