# Pod 정렬 스크립트 작성

문제에서 kubectl sorting을 사용하여 두 개의 bash 스크립트를 작성해야 합니다:

1. /opt/course/5/find_pods.sh:
   - 모든 네임스페이스의 Pod를 AGE(metadata.creationTimestamp) 기준으로 정렬

2. /opt/course/5/find_pods_uid.sh:
   - 모든 네임스페이스의 Pod를 metadata.uid 기준으로 정렬

<br/>

## 1. 작업 디렉토리 생성
```bash
# 디렉토리 생성
mkdir -p /opt/course/5
```

<br/>

## 2. AGE 기준 정렬 스크립트 작성
```bash
# find_pods.sh 스크립트 생성
cat << 'EOF' > /opt/course/5/find_pods.sh
#!/bin/bash
kubectl get pods --sort-by=.metadata.creationTimestamp -A
EOF

# 실행 권한 부여
chmod +x /opt/course/5/find_pods.sh

# 테스트
/opt/course/5/find_pods.sh
```

<br/>

## 3. UID 기준 정렬 스크립트 작성
```bash
# find_pods_uid.sh 스크립트 생성
cat << 'EOF' > /opt/course/5/find_pods_uid.sh
#!/bin/bash
kubectl get pods --sort-by=.metadata.uid -A
EOF

# 실행 권한 부여
chmod +x /opt/course/5/find_pods_uid.sh

# 테스트
/opt/course/5/find_pods_uid.sh
```

<br/>

## 4. 스크립트 검증
```bash
# 스크립트 파일 확인
ls -l /opt/course/5/find_pods.sh /opt/course/5/find_pods_uid.sh

# 스크립트 내용 확인
cat /opt/course/5/find_pods.sh
cat /opt/course/5/find_pods_uid.sh

# 실행 결과 확인
bash /opt/course/5/find_pods.sh
bash /opt/course/5/find_pods_uid.sh
```

<br/>

## 선택적: UID를 포함한 상세 출력
```bash
# AGE 정렬 스크립트에 상세 출력 추가
cat << 'EOF' > /opt/course/5/find_pods.sh
#!/bin/bash
kubectl get pods -A --sort-by=.metadata.creationTimestamp \
  -o custom-columns=NAMESPACE:.metadata.namespace,NAME:.metadata.name,AGE:.metadata.creationTimestamp
EOF

# UID 정렬 스크립트에 상세 출력 추가
cat << 'EOF' > /opt/course/5/find_pods_uid.sh
#!/bin/bash
kubectl get pods -A --sort-by=.metadata.uid \
  -o custom-columns=NAMESPACE:.metadata.namespace,NAME:.metadata.name,UID:.metadata.uid
EOF
```

<br/>

## 주의사항:
1. 스크립트 파일에 실행 권한이 있는지 확인
2. 스크립트가 올바른 셔뱅(#!/bin/bash)을 포함하는지 확인
3. kubectl 명령어가 정상적으로 작동하는지 확인
4. 정렬 필드 이름이 정확한지 확인
   - `.metadata.creationTimestamp`
   - `.metadata.uid`
