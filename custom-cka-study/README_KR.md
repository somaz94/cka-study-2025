## q1: ETCD 백업 및 복원

노드 'cluster1-controlplane'에서 ETCD 백업 및 복원 시나리오를 수행합니다. 다음 정보를 사용합니다:

ETCD 엔드포인트: https://127.0.0.1:2379
CA 인증서: /etc/kubernetes/pki/etcd/ca.crt
서버 인증서: /etc/kubernetes/pki/etcd/server.crt
서버 키: /etc/kubernetes/pki/etcd/server.key

작업:
1. 제공된 인증서를 사용하여 현재 ETCD 데이터를 `/opt/backup/etcd-backup.db`로 백업합니다
2. '사전 복원'이라는 테스트 네임스페이스를 만들고 이미지 `nginx:1.22`를 사용하여 `nginx`를 배포합니다
3. 백업 지점에 ETCD 데이터 복원
4. 복원 후 테스트 리소스(이름 공간 및 배포)가 사라졌는지 확인합니다

<br/>

## q2: 멀티 컨테이너 파드 트러블슈팅

다음 요구사항을 만족하는 멀티 컨테이너 파드를 생성하고 문제를 해결하세요:

### 초기 설정
1. `multi-container-pod`라는 이름의 파드를 두 개의 컨테이너로 생성:
   - 컨테이너 1: nginx (이름: nginx)
   - 컨테이너 2: busybox (이름: log-monitor)

2. 공유 볼륨 구성:
   - `shared-data` 볼륨: nginx 컨테이너의 `/usr/share/nginx/html`에 마운트
   - `nginx-logs` 볼륨: 양쪽 컨테이너의 `/var/log/nginx`에 마운트

3. 로그 모니터링 설정:
   - `busybox` 컨테이너는 nginx 접근 로그를 지속적으로 모니터링
   - 사용할 명령어: `tail -f /var/log/nginx/access.log`

### 문제 해결 작업
1. 권한 문제 해결:
   - 두 컨테이너 모두 nginx 로그 파일에 접근 가능하도록 설정
   - 적절한 보안 컨텍스트(security context) 구성
   - 로그 모니터링이 정상 작동하는지 확인

### 검증 단계
1. 파드 실행 상태 확인:
```bash
kubectl get pod multi-container-pod
```

2. nginx 작동 테스트:
```bash
kubectl exec multi-container-pod -c nginx -- curl localhost
```

3. 로그 모니터링 확인:
```bash
kubectl logs multi-container-pod -c log-monitor
```

4. 공유 볼륨 확인:
```bash
kubectl exec multi-container-pod -c nginx -- ls /usr/share/nginx/html
kubectl exec multi-container-pod -c log-monitor -- ls /var/log/nginx
```

참고: 모든 컨테이너는 공유 리소스에 접근할 수 있는 적절한 권한으로 실행되어야 합니다.

<br/>

## q3: 노드 유지보수
클러스터의 워커 노드 중 하나를 유지보수하기 위한 작업을 수행하세요:
1. 노드를 스케줄 불가능 상태로 변경
2. 실행 중인 파드들을 안전하게 다른 노드로 이동
3. 시스템 업그레이드 시뮬레이션 (30초 대기)
4. 노드를 다시 스케줄 가능 상태로 변경

<br/>

## q4: 네트워크 정책 구성
다음 요구사항을 만족하는 네트워크 정책을 구성하세요:

## 초기 설정
1. 다음 네임스페이스들을 생성하세요:
   - frontend
   - backend
   - database

2. 각 네임스페이스에 테스트용 nginx 파드를 배포하세요:
   - frontend/frontend-pod
   - backend/backend-pod
   - database/db-pod

## 네트워크 정책 요구사항
1. frontend 네임스페이스:
   - backend 네임스페이스의 파드와만 통신 가능
   - 다른 모든 인그레스/이그레스 트래픽 차단
   - 레이블: role=frontend

2. backend 네임스페이스:
   - frontend에서 오는 트래픽 허용
   - database 네임스페이스로 나가는 트래픽만 허용
   - 레이블: role=backend

3. database 네임스페이스:
   - backend에서 오는 트래픽만 허용
   - 모든 이그레스 트래픽 차단
   - 레이블: role=database

## 검증 테스트
다음 테스트를 수행하여 정책이 올바르게 적용되었는지 확인하세요:

1. frontend -> backend 통신 테스트
2. frontend -> database 통신 차단 확인
3. backend -> database 통신 테스트
4. database -> 외부 통신 차단 확인

테스트는 다음 명령어를 사용하여 수행:
```bash
kubectl exec -it <pod-name> -n <namespace> -- wget -qO- --timeout=2 http://<target-pod-ip>
```

모든 통신 테스트 결과를 `/opt/network-policy-test.txt` 파일에 기록하세요.

<br/>

## q5: 사용자 인증 및 권한 관리

새로운 개발팀 관리자를 위한 인증서 기반 인증을 구성하세요.

### 초기 설정
1. 다음 정보로 새로운 사용자 인증서를 생성하세요:
   - 사용자 이름: developer-admin
   - 그룹: devops-team
   - 인증서 유효기간: 365일
   - 인증서 저장 위치: `/opt/certificates/developer-admin.crt`
   - 개인키 저장 위치: `/opt/certificates/developer-admin.key`

2. `development` 네임스페이스를 생성하고 다음 권한을 부여하세요:
   - Deployment, Pod, Service 리소스에 대한 모든 권한
   - ConfigMap과 Secret에 대한 읽기 권한
   - Namespace 리소스에 대한 접근 불가

### kubeconfig 설정
1. 다음 설정으로 kubeconfig 파일을 생성하세요:
   - 파일 위치: `/opt/certificates/developer-admin.kubeconfig`
   - Context 이름: `developer-context`
   - Cluster 이름: `kubernetes`
   - 네임스페이스: `development`

### 권한 테스트
다음 작업을 수행하여 권한이 올바르게 설정되었는지 확인하세요:

1. 새로운 kubeconfig로 인증 테스트:
```bash
kubectl --kubeconfig=/opt/certificates/developer-admin.kubeconfig get pods -n development
```

2. 다음 권한들을 테스트하고 결과를 /opt/developer-admin-tests.txt에 기록:
   - Deployment 생성 가능 여부
   - ConfigMap 읽기 가능 여부
   - Secret 읽기 가능 여부
   - Namespace 생성 불가 여부

### 예상 결과물
1. 인증서 파일들:
   - /opt/certificates/developer-admin.crt
   - /opt/certificates/developer-admin.key
2. kubeconfig 파일:
   - /opt/certificates/developer-admin.kubeconfig
3. 테스트 결과 파일:
   - /opt/developer-admin-tests.txt

참고: 모든 인증서는 클러스터의 CA로 서명되어야 하며, 적절한 권한으로 파일이 생성되어야 합니다.

<br/>

## q6: 리소스 관리 및 스케줄링
다음 요구사항을 만족하는 파드 스케줄링을 구성하세요:
1. GPU 레이블이 있는 노드에만 특정 파드가 스케줄되도록 설정
2. 메모리 요구사항이 높은 파드들이 특정 노드에 분산되도록 구성
3. 특정 파드들이 서로 다른 노드에 배포되도록 anti-affinity 규칙 설정
4. 노드 리소스 사용량 모니터링 구성

<br/>

## q7: 서비스 및 인그레스 구성
다음 요구사항을 만족하는 서비스 구성을 수행하세요:
1. NodePort 서비스로 애플리케이션 노출
2. 인그레스 컨트롤러 설정
3. 도메인 기반 라우팅 규칙 구성
4. SSL/TLS 인증서 적용

<br/>

## q8: 스토리지 구성
다음 요구사항을 만족하는 스토리지 구성을 수행하세요:
1. 동적 프로비저닝을 위한 StorageClass 생성
2. PVC를 사용하는 스테이트풀셋 배포
3. 볼륨 스냅샷 생성 및 복구
4. 스토리지 확장 테스트

<br/>

## q9: 로깅 및 모니터링
클러스터의 모니터링 시스템을 구성하세요:
1. metrics-server 설치 및 구성
2. HorizontalPodAutoscaler 설정
3. 노드 및 파드 메트릭 수집 확인
4. 리소스 사용량 기반 자동 스케일링 테스트

<br/>

## q10: 클러스터 업그레이드
클러스터 업그레이드 절차를 수행하세요:
1. 현재 버전에서 한 단계 높은 버전으로 업그레이드 계획 수립
2. 컨트롤 플레인 컴포넌트 업그레이드
3. 워커 노드 순차적 업그레이드
4. 업그레이드 완료 후 기능 테스트
