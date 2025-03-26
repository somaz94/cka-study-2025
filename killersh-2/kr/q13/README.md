# 멀티 컨테이너 Pod와 공유 볼륨 구성

문제 요구사항:
1. multi-container-playground Pod 생성
2. 세 개의 컨테이너가 공유하는 임시 볼륨 구성
3. c1: 노드 이름을 환경변수로 가짐
4. c2: 매초 date 명령어 결과를 파일에 기록
5. c3: 기록된 파일을 실시간으로 출력

<br/>

## 1. Pod YAML 생성
```bash
# 기본 Pod YAML 생성
kubectl run multi-container-playground --image=nginx:1-alpine --dry-run=client -o yaml > 13.yaml

# YAML 수정
cat << EOF > 13.yaml
apiVersion: v1
kind: Pod
metadata:
  name: multi-container-playground
spec:
  containers:
  - name: c1
    image: nginx:1-alpine
    env:
    - name: MY_NODE_NAME
      valueFrom:
        fieldRef:
          fieldPath: spec.nodeName
    volumeMounts:
    - name: vol
      mountPath: /vol
      
  - name: c2
    image: busybox:1
    command: ["sh", "-c", "while true; do date >> /vol/date.log; sleep 1; done"]
    volumeMounts:
    - name: vol
      mountPath: /vol
      
  - name: c3
    image: busybox:1
    command: ["sh", "-c", "tail -f /vol/date.log"]
    volumeMounts:
    - name: vol
      mountPath: /vol
      
  volumes:
  - name: vol
    emptyDir: {}
EOF
```

<br/>

## 2. Pod 생성 및 확인
```bash
# Pod 생성
kubectl apply -f 13.yaml

# Pod 상태 확인
kubectl get pod multi-container-playground
```

<br/>

## 3. 기능 검증
```bash
# c1 컨테이너의 환경변수 확인
kubectl exec multi-container-playground -c c1 -- env | grep MY
# 예상 결과: MY_NODE_NAME=cka3200

# c3 컨테이너의 로그 확인 (c2가 기록한 내용)
kubectl logs multi-container-playground -c c3
# 예상 결과:
# Tue Nov 5 13:41:33 UTC 2024
# Tue Nov 5 13:41:34 UTC 2024
# ...
```

<br/>

## 구성 설명:
1. 볼륨:
   - emptyDir: Pod 생명주기와 함께하는 임시 볼륨
   - 모든 컨테이너에 /vol로 마운트

2. 컨테이너:
   - c1: 노드 이름을 환경변수로 설정
   - c2: 날짜를 파일에 지속적으로 기록
   - c3: 파일 내용을 실시간으로 모니터링

<br/>

## 주의사항:
1. 볼륨은 emptyDir 타입 사용 (임시, 비지속성)
2. 모든 컨테이너가 같은 볼륨을 마운트
3. c2와 c3의 명령어가 정확해야 함
4. 환경변수 이름이 정확히 MY_NODE_NAME
