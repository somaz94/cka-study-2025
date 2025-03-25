# 멀티 컨테이너 파드 문제 해결

<br/>

## 해결 방법

<br/>

### 1. 공유 볼륨을 가진 파드 생성
```yaml
apiVersion: v1
kind: Pod
metadata:
  name: multi-container-pod
spec:
  volumes:
  - name: shared-data
    emptyDir: {}
  - name: nginx-logs
    emptyDir: {}
  containers:
  - name: nginx
    image: nginx
    volumeMounts:
    - name: shared-data
      mountPath: /usr/share/nginx/html
    - name: nginx-logs
      mountPath: /var/log/nginx
  - name: log-monitor
    image: busybox
    command: ["/bin/sh", "-c"]
    args:
    - while true; do
        tail -f /var/log/nginx/access.log;
        sleep 30;
      done
    volumeMounts:
    - name: nginx-logs
      mountPath: /var/log/nginx
```

<br/>

### 2. 권한 문제 해결
```bash
# 기존 파드 삭제
kubectl delete pod multi-container-pod
```

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: multi-container-pod
spec:
  volumes:
  - name: shared-data
    emptyDir: {}
  - name: nginx-logs
    emptyDir: {}
  containers:
  - name: nginx
    image: nginx
    volumeMounts:
    - name: shared-data
      mountPath: /usr/share/nginx/html
    - name: nginx-logs
      mountPath: /var/log/nginx
    securityContext:
      runAsUser: 0
  - name: log-monitor
    image: busybox
    command: ["/bin/sh", "-c"]
    args:
    - while true; do
        tail -f /var/log/nginx/access.log;
        sleep 30;
      done
    volumeMounts:
    - name: nginx-logs
      mountPath: /var/log/nginx
    securityContext:
      runAsUser: 0
```

<br/>

### 3. 설정 확인
```bash
# 파드 상태 확인
kubectl get pod multi-container-pod

# 로그 모니터 컨테이너의 로그 확인
kubectl logs multi-container-pod -c log-monitor

# nginx 작동 확인
kubectl exec multi-container-pod -c nginx -- curl localhost

# 공유 볼륨 확인
kubectl exec multi-container-pod -c nginx -- ls /usr/share/nginx/html
kubectl exec multi-container-pod -c log-monitor -- ls /var/log/nginx
```

<br/>

## 주요 변경사항:
1. nginx 로그를 위한 별도의 볼륨 추가
2. 로그 모니터링을 위한 명령어 설정
3. 권한 문제 해결을 위한 securityContext 설정
4. 검증 단계 추가

<br/>

## 참고사항:
- 두 컨테이너 모두 nginx 로그에 접근하기 위해 root 권한 필요 (runAsUser: 0)
- 로그 모니터 컨테이너는 nginx 접근 로그를 지속적으로 감시
- shared-data 볼륨은 다른 공유 파일에도 사용 가능
- 두 컨테이너의 정상 작동 여부를 주기적으로 확인해야 함
