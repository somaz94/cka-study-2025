# MinIO Operator, CRD Config, Helm Install

<br/>

## 문제 설명
`ssh cka7968`에서 다음 작업을 수행합니다:

MinIO Operator를 Helm을 사용하여 `minio` Namespace에 설치하고, Tenant CRD를 구성 및 생성:

1. Namespace `minio` 생성
2. Helm chart `minio/operator`를 새 Namespace에 설치 (Helm Release 이름: `minio-operator`)
3. `/opt/course/2/minio-tenant.yaml`의 `Tenant` 리소스에 `features` 아래 `enableSFTP: true` 추가
4. `/opt/course/2/minio-tenant.yaml`에서 `Tenant` 리소스 생성

> ℹ️ MinIO가 제대로 실행될 필요는 없습니다. Helm Chart와 Tenant 리소스를 요청대로 설치하면 충분합니다.

<br/>

## 주요 개념
# Helm Chart: Kubernetes YAML 템플릿 파일들을 단일 패키지로 결합, Values로 커스터마이징 가능
# Helm Release: Chart의 설치된 인스턴스
# Helm Values: Release 생성 시 Chart의 YAML 템플릿 파일 커스터마이징 가능
# Operator: Kubernetes API와 통신하고 CRD와 작동할 수 있는 Pod
# CRD: Kubernetes API의 확장인 Custom Resources

<br/>

## 해결 방법

### 1. Namespace 생성
```bash
# minio namespace 생성
k create ns minio
```

<br/>

### 2. Helm Chart 설치
```bash
# Helm 저장소 확인
helm repo list

# Helm 차트 검색
helm search repo

# minio namespace에 operator 설치
helm -n minio install minio-operator minio/operator

# 설치 확인
helm -n minio ls

# Pod 상태 확인
k -n minio get pod

# CRD 확인
k get crd
```

<br/>

### 3. Tenant 리소스 업데이트
```bash
# /opt/course/2/minio-tenant.yaml 파일 내용
apiVersion: minio.min.io/v2
kind: Tenant
metadata:
  name: tenant
  namespace: minio
  labels:
    app: minio
spec:
  features:
    bucketDNS: false
    enableSFTP: true                     # 추가된 부분
  image: quay.io/minio/minio:latest
  pools:
    - servers: 1
      name: pool-0
      volumesPerServer: 0
      volumeClaimTemplate:
        apiVersion: v1
        kind: persistentvolumeclaims
        metadata: { }
        spec:
          accessModes:
            - ReadWriteOnce
          resources:
            requests:
              storage: 10Mi
          storageClassName: standard
        status: { }
  requestAutoCert: true
```

<br/>

### 4. Tenant 리소스 생성
```bash
# Tenant 리소스 생성
k -f /opt/course/2/minio-tenant.yaml apply

# Tenant 상태 확인
k -n minio get tenant
```

<br/>

## 결론
이 시나리오에서는 Helm을 사용하여 operator를 설치하고, operator가 작동하는 CRD를 생성했습니다. 이는 Kubernetes에서 흔히 볼 수 있는 패턴입니다.
