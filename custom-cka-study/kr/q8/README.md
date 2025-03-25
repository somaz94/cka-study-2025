# 스토리지 구성

<br/>

## 1. StorageClass 생성
```yaml
# 기본 StorageClass 생성
apiVersion: storage.k8s.io/v1
kind: StorageClass
metadata:
  name: fast-storage
  annotations:
    storageclass.kubernetes.io/is-default-class: "true"
provisioner: kubernetes.io/gce-pd  # 클라우드 환경에 맞게 변경
parameters:
  type: pd-ssd
  fstype: ext4
reclaimPolicy: Delete
allowVolumeExpansion: true
```

<br/>

## 2. 스테이트풀셋 배포
```yaml
# PVC 생성
apiVersion: v1
kind: PersistentVolumeClaim
metadata:
  name: data-pvc
spec:
  accessModes:
    - ReadWriteOnce
  storageClassName: fast-storage
  resources:
    requests:
      storage: 10Gi
---
# 스테이트풀셋 배포
apiVersion: apps/v1
kind: StatefulSet
metadata:
  name: mysql
spec:
  serviceName: mysql
  replicas: 3
  selector:
    matchLabels:
      app: mysql
  template:
    metadata:
      labels:
        app: mysql
    spec:
      containers:
      - name: mysql
        image: mysql:5.7
        env:
        - name: MYSQL_ROOT_PASSWORD
          value: "password123"
        ports:
        - containerPort: 3306
        volumeMounts:
        - name: data
          mountPath: /var/lib/mysql
  volumeClaimTemplates:
  - metadata:
      name: data
    spec:
      accessModes: [ "ReadWriteOnce" ]
      storageClassName: fast-storage
      resources:
        requests:
          storage: 10Gi
```

<br/>

## 3. 볼륨 스냅샷 구성
```yaml
# 볼륨 스냅샷 클래스 생성
apiVersion: snapshot.storage.k8s.io/v1
kind: VolumeSnapshotClass
metadata:
  name: csi-snapshot-class
driver: kubernetes.io/gce-pd  # 클라우드 환경에 맞게 변경
deletionPolicy: Delete
---
# 볼륨 스냅샷 생성
apiVersion: snapshot.storage.k8s.io/v1
kind: VolumeSnapshot
metadata:
  name: mysql-snapshot-0
spec:
  volumeSnapshotClassName: csi-snapshot-class
  source:
    persistentVolumeClaimName: data-mysql-0
---
# 스냅샷으로부터 PVC 복구
apiVersion: v1
kind: PersistentVolumeClaim
metadata:
  name: mysql-restore-pvc
spec:
  accessModes:
    - ReadWriteOnce
  storageClassName: fast-storage
  resources:
    requests:
      storage: 10Gi
  dataSource:
    name: mysql-snapshot-0
    kind: VolumeSnapshot
    apiGroup: snapshot.storage.k8s.io
```

<br/>

## 4. 스토리지 확장 테스트
```yaml
# PVC 크기 확장
apiVersion: v1
kind: PersistentVolumeClaim
metadata:
  name: data-mysql-0
spec:
  accessModes:
    - ReadWriteOnce
  storageClassName: fast-storage
  resources:
    requests:
      storage: 20Gi  # 크기를 20Gi로 확장
```

<br/>

## 5. 구성 검증
```bash
# 테스트 결과 파일 생성
echo "스토리지 구성 테스트 결과" > /opt/storage-test.txt
echo "------------------------" >> /opt/storage-test.txt

# StorageClass 확인
echo -e "\n1. StorageClass 상태:" >> /opt/storage-test.txt
kubectl get sc >> /opt/storage-test.txt

# PVC 및 PV 상태 확인
echo -e "\n2. PVC/PV 상태:" >> /opt/storage-test.txt
kubectl get pvc,pv >> /opt/storage-test.txt

# 스테이트풀셋 상태 확인
echo -e "\n3. 스테이트풀셋 상태:" >> /opt/storage-test.txt
kubectl get statefulset mysql >> /opt/storage-test.txt
kubectl get pods -l app=mysql >> /opt/storage-test.txt

# 스냅샷 상태 확인
echo -e "\n4. 볼륨 스냅샷 상태:" >> /opt/storage-test.txt
kubectl get volumesnapshot >> /opt/storage-test.txt

# 스토리지 확장 확인
echo -e "\n5. 스토리지 확장 상태:" >> /opt/storage-test.txt
kubectl get pvc data-mysql-0 -o jsonpath='{.status.capacity.storage}' >> /opt/storage-test.txt
```

<br/>

## 데이터 지속성 테스트
```bash
# MySQL 데이터 생성
kubectl exec mysql-0 -- mysql -uroot -ppassword123 <<EOF
CREATE DATABASE test;
USE test;
CREATE TABLE messages (message VARCHAR(255));
INSERT INTO messages VALUES ('Hello, persistent storage!');
EOF

# 파드 재시작 후 데이터 확인
kubectl delete pod mysql-0
sleep 30
kubectl exec mysql-0 -- mysql -uroot -ppassword123 -e "SELECT * FROM test.messages;" >> /opt/storage-test.txt
```

<br/>

## 주의사항:
1. 클라우드 환경에 맞는 스토리지 프로비저너 사용
2. 볼륨 스냅샷 기능이 클러스터에서 활성화되어 있는지 확인
3. 스토리지 확장 기능이 지원되는지 확인
4. 데이터 백업 전 애플리케이션 정지 고려
