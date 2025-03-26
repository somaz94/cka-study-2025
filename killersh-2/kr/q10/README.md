# PV, PVC 및 동적 프로비저닝 구성

문제 요구사항:
1. local-backup StorageClass 생성
   - provisioner: rancher.io/local-path
   - volumeBindingMode: WaitForFirstConsumer
   - PVC 삭제 시에도 PV 유지
2. backup Job을 PVC 사용하도록 수정
   - 50Mi 스토리지 요청
   - 새로운 StorageClass 사용

<br/>

## 1. StorageClass 생성
```bash
# 기존 StorageClass 확인
kubectl get sc

# 새로운 StorageClass 생성
cat << EOF > sc.yaml
apiVersion: storage.k8s.io/v1
kind: StorageClass
metadata:
  name: local-backup
provisioner: rancher.io/local-path
reclaimPolicy: Retain
volumeBindingMode: WaitForFirstConsumer
EOF

# StorageClass 적용
kubectl apply -f sc.yaml

# 생성 확인
kubectl get sc
```

<br/>

## 2. Job 수정
```bash
# 기존 Job 백업
cp /opt/course/10/backup.yaml /opt/course/10/backup.yaml.bak

# Job 및 PVC 수정
cat << EOF > /opt/course/10/backup.yaml
apiVersion: v1
kind: PersistentVolumeClaim
metadata:
  name: backup-pvc
  namespace: project-bern
spec:
  accessModes:
    - ReadWriteOnce
  resources:
    requests:
      storage: 50Mi
  storageClassName: local-backup
---
apiVersion: batch/v1
kind: Job
metadata:
  name: backup
  namespace: project-bern
spec:
  backoffLimit: 0
  template:
    spec:
      volumes:
        - name: backup
          persistentVolumeClaim:
            claimName: backup-pvc
      containers:
        - name: bash
          image: bash:5
          command:
            - bash
            - -c
            - |
              set -x
              touch /backup/backup-\$(date +%Y-%m-%d-%H-%M-%S).tar.gz
              sleep 15
          volumeMounts:
            - name: backup
              mountPath: /backup
      restartPolicy: Never
EOF
```

<br/>

## 3. 변경사항 적용 및 검증
```bash
# 기존 Job 삭제 (있다면)
kubectl -n project-bern delete job backup

# 새로운 설정 적용
kubectl apply -f /opt/course/10/backup.yaml

# 리소스 상태 확인
kubectl -n project-bern get job,pod,pvc,pv

# 선택사항: 실제 저장된 파일 확인
find /opt/local-path-provisioner
```

<br/>

## 예상 결과:
1. local-backup StorageClass 생성됨
2. PVC가 생성되고 PV에 바인딩됨
3. Job이 성공적으로 실행되어 백업 파일 생성
4. PVC 삭제 시에도 PV와 데이터 유지됨

<br/>

## 주의사항:
1. StorageClass의 reclaimPolicy를 Retain으로 설정
2. Job과 PVC가 같은 네임스페이스에 있어야 함
3. PVC 용량이 정확히 50Mi로 설정되어야 함
4. Job 재실행 시 이전 Job 삭제 필요
