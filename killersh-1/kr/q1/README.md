# Kubeconfig 정보 추출하기

<br/>

## 문제 설명
- /opt/course/1/kubeconfig 파일에서 다음 정보를 추출해야 합니다.
  1. 모든 context 이름을 /opt/course/1/contexts에 한 줄씩 저장
  2. 현재 context 이름을 /opt/course/1/current-context에 저장
  3. account-0027 사용자의 client-certificate를 base64 디코딩하여 /opt/course/1/cert에 저장

<br/>

## 해결 방법

### 1. Context 이름 추출하기

```bash
# 모든 context 이름 확인
kubectl --kubeconfig /opt/course/1/kubeconfig config get-contexts

# context 이름만 파일에 저장
kubectl --kubeconfig /opt/course/1/kubeconfig config get-contexts -oname > /opt/course/1/contexts
```

결과:
```bash
cluster-admin
cluster-w100
cluster-w200
```

<br/>

### 2. 현재 Context 확인하기

```bash
# 현재 context 확인 및 파일에 저장
kubectl --kubeconfig /opt/course/1/kubeconfig config current-context > /opt/course/1/current-context
```

결과:
```bash
cluster-w200
```

<br/>

### 3. Client-Certificate 추출하기

```bash
# 방법 1: 전체 설정 보기
kubectl --kubeconfig /opt/course/1/kubeconfig config view -o yaml --raw

# 방법 2: jsonpath를 사용하여 자동화
kubectl --kubeconfig /opt/course/1/kubeconfig config view --raw \
  -ojsonpath="{.users[0].user.client-certificate-data}" | base64 -d > /opt/course/1/cert
```

결과:
```bash
-----BEGIN CERTIFICATE-----
MIICvDCCAaQCFHYdjSZFKCyUCR1B2naXCg...
-----END CERTIFICATE-----
```

<br/>

## 참고 사항
- --raw 옵션은 민감한 인증서 정보를 볼 때 사용
- jsonpath를 사용하면 특정 정보만 정확하게 추출 가능
- base64 디코딩은 인증서 데이터를 읽을 수 있는 형태로 변환

<br/>

## 유용한 kubectl 명령어
```bash
# context 목록 보기
kubectl config get-contexts

# 현재 context 확인
kubectl config current-context

# 설정 보기 (yaml 형식)
kubectl config view -o yaml
```