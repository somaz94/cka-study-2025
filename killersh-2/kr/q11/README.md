# Secret 생성 및 Pod 마운트

문제 요구사항:
1. secret 네임스페이스 생성
2. busybox:1 이미지를 사용하는 secret-pod 생성
3. secret1을 /tmp/secret1에 읽기 전용으로 마운트
4. secret2를 환경 변수로 사용

<br/>

## 1. 네임스페이스 생성
```bash
# 네임스페이스 생성
kubectl create ns secret
```

<br/>

## 2. Secret1 생성
```bash
# 기존 Secret 파일 복사 및 수정
cp /opt/course/11/secret1.yaml 11_secret1.yaml

# 네임스페이스 수정
cat << EOF > 11_secret1.yaml
apiVersion: v1
data:
  halt: IyEgL2Jpbi9zaAo...
kind: Secret
metadata:
  name: secret1
  namespace: secret
EOF

# Secret1 생성
kubectl apply -f 11_secret1.yaml
```

<br/>

## 3. Secret2 생성
```bash
# 리터럴 값으로 Secret 생성
kubectl -n secret create secret generic secret2 \
  --from-literal=user=user1 \
  --from-literal=pass=1234
```

<br/>

## 4. Pod 생성
```bash
# Pod YAML 생성
cat << EOF > 11.yaml
apiVersion: v1
kind: Pod
metadata:
  name: secret-pod
  namespace: secret
spec:
  containers:
  - name: secret-pod
    image: busybox:1
    args:
    - sh
    - -c
    - sleep 1d
    env:
    - name: APP_USER
      valueFrom:
        secretKeyRef:
          name: secret2
          key: user
    - name: APP_PASS
      valueFrom:
        secretKeyRef:
          name: secret2
          key: pass
    volumeMounts:
    - name: secret1
      mountPath: /tmp/secret1
      readOnly: true
  volumes:
  - name: secret1
    secret:
      secretName: secret1
EOF

# Pod 생성
kubectl apply -f 11.yaml
```

<br/>

## 5. 검증
```bash
# 환경 변수 확인
kubectl -n secret exec secret-pod -- env | grep APP

# 마운트된 Secret 확인
kubectl -n secret exec secret-pod -- find /tmp/secret1

# Secret 내용 확인
kubectl -n secret exec secret-pod -- cat /tmp/secret1/halt
```

<br/>

## 예상 결과:
1. 환경 변수 확인 시:
   ```
   APP_USER=user1
   APP_PASS=1234
   ```

2. Secret1 마운트 확인:
   ```
   /tmp/secret1
   /tmp/secret1/halt
   ```

<br/>

## 주의사항:
1. 모든 리소스가 secret 네임스페이스에 생성되어야 함
2. Secret1은 읽기 전용으로 마운트되어야 함
3. Secret2의 키가 정확한 환경 변수 이름으로 매핑되어야 함
4. Pod가 정상적으로 실행 중이어야 함
