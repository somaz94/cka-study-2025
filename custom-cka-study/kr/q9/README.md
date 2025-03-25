# 로깅 및 모니터링 구성

<br/>

## 1. metrics-server 설치 및 구성
```bash
# metrics-server 설치
kubectl apply -f https://github.com/kubernetes-sigs/metrics-server/releases/latest/download/components.yaml

# metrics-server 배포 확인
kubectl get deployment metrics-server -n kube-system

# metrics-server 작동 확인
kubectl get --raw "/apis/metrics.k8s.io/v1beta1/nodes" | jq
```

<br/>

## 2. 테스트 애플리케이션 및 HPA 설정
```yaml
# 테스트용 애플리케이션 배포
apiVersion: apps/v1
kind: Deployment
metadata:
  name: php-apache
spec:
  selector:
    matchLabels:
      run: php-apache
  replicas: 1
  template:
    metadata:
      labels:
        run: php-apache
    spec:
      containers:
      - name: php-apache
        image: k8s.gcr.io/hpa-example
        ports:
        - containerPort: 80
        resources:
          limits:
            cpu: 500m
          requests:
            cpu: 200m
---
# 서비스 생성
apiVersion: v1
kind: Service
metadata:
  name: php-apache
  labels:
    run: php-apache
spec:
  ports:
  - port: 80
  selector:
    run: php-apache
---
# HPA 설정
apiVersion: autoscaling/v2
kind: HorizontalPodAutoscaler
metadata:
  name: php-apache
spec:
  scaleTargetRef:
    apiVersion: apps/v1
    kind: Deployment
    name: php-apache
  minReplicas: 1
  maxReplicas: 10
  metrics:
  - type: Resource
    resource:
      name: cpu
      target:
        type: Utilization
        averageUtilization: 50
```

<br/>

## 3. 메트릭 수집 확인
```bash
# 모니터링 결과 파일 생성
echo "클러스터 모니터링 테스트 결과" > /opt/monitoring-test.txt
echo "----------------------------" >> /opt/monitoring-test.txt

# 노드 메트릭 확인
echo -e "\n1. 노드 메트릭:" >> /opt/monitoring-test.txt
kubectl top nodes >> /opt/monitoring-test.txt

# 파드 메트릭 확인
echo -e "\n2. 파드 메트릭:" >> /opt/monitoring-test.txt
kubectl top pods --all-namespaces >> /opt/monitoring-test.txt

# HPA 상태 확인
echo -e "\n3. HPA 초기 상태:" >> /opt/monitoring-test.txt
kubectl get hpa php-apache >> /opt/monitoring-test.txt
```

<br/>

## 4. 자동 스케일링 테스트
```bash
# 부하 생성을 위한 컨테이너 실행
kubectl run -i --tty load-generator --rm --image=busybox:1.28 --restart=Never -- /bin/sh -c "while sleep 0.01; do wget -q -O- http://php-apache; done"

# 테스트 결과 기록
echo -e "\n4. 부하 테스트 중 HPA 상태:" >> /opt/monitoring-test.txt
kubectl get hpa php-apache --watch >> /opt/monitoring-test.txt &
HPA_PID=$!

# 1분 대기
sleep 60

# 파드 스케일링 상태 확인
echo -e "\n5. 스케일링 후 파드 상태:" >> /opt/monitoring-test.txt
kubectl get pods | grep php-apache >> /opt/monitoring-test.txt

# HPA 모니터링 중단
kill $HPA_PID

# 부하 테스트 종료
kubectl delete pod load-generator --force

# 최종 상태 확인
echo -e "\n6. 부하 테스트 후 메트릭:" >> /opt/monitoring-test.txt
kubectl top pods | grep php-apache >> /opt/monitoring-test.txt
```

<br/>

## 5. 모니터링 대시보드 설정 (선택사항)
```yaml
# Prometheus 설치 (선택사항)
apiVersion: v1
kind: Namespace
metadata:
  name: monitoring
---
# Prometheus Operator 설치를 위한 helm 명령어
helm repo add prometheus-community https://prometheus-community.github.io/helm-charts
helm repo update
helm install prometheus prometheus-community/kube-prometheus-stack \
  --namespace monitoring \
  --create-namespace
```

<br/>

## 주의사항:
1. metrics-server가 정상적으로 작동하는지 확인
2. 리소스 요청(requests)과 제한(limits)이 적절히 설정되었는지 확인
3. HPA 설정의 타겟 메트릭이 현실적인 값인지 확인
4. 테스트 중 클러스터 리소스 상태 모니터링
