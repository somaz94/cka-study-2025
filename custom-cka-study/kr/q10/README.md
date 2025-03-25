# 클러스터 업그레이드 절차

<br/>

## 1. 업그레이드 계획 수립
```bash
# 현재 클러스터 버전 확인
echo "클러스터 업그레이드 계획" > /opt/upgrade-plan.txt
echo "----------------------" >> /opt/upgrade-plan.txt

echo -e "\n1. 현재 클러스터 상태:" >> /opt/upgrade-plan.txt
kubectl version --short >> /opt/upgrade-plan.txt
kubectl get nodes >> /opt/upgrade-plan.txt

# 업그레이드 가능 버전 확인
kubeadm upgrade plan >> /opt/upgrade-plan.txt

# 워크로드 상태 확인
echo -e "\n2. 업그레이드 전 워크로드 상태:" >> /opt/upgrade-plan.txt
kubectl get pods --all-namespaces >> /opt/upgrade-plan.txt
```

<br/>

## 2. 컨트롤 플레인 업그레이드
```bash
# kubeadm 업그레이드
apt-mark unhold kubeadm
apt-get update && apt-get install -y kubeadm=1.27.1-00
apt-mark hold kubeadm

# 업그레이드 계획 확인
kubeadm upgrade plan

# 컨트롤 플레인 노드 drain
kubectl drain control-plane-node --ignore-daemonsets

# 컨트롤 플레인 업그레이드 실행
kubeadm upgrade apply v1.27.1

# kubelet 및 kubectl 업그레이드
apt-mark unhold kubelet kubectl
apt-get install -y kubelet=1.27.1-00 kubectl=1.27.1-00
apt-mark hold kubelet kubectl

# kubelet 재시작
systemctl daemon-reload
systemctl restart kubelet

# 노드 uncordon
kubectl uncordon control-plane-node
```

<br/>

## 3. 워커 노드 업그레이드
```bash
# 각 워커 노드에 대해 순차적으로 실행
for node in $(kubectl get nodes -l '!node-role.kubernetes.io/control-plane' -o name); do
    echo "Upgrading $node..."
    
    # 노드 drain
    kubectl drain ${node#node/} --ignore-daemonsets --delete-emptydir-data
    
    # 워커 노드에서 실행할 명령어
    ssh ${node#node/} << EOF
        # kubeadm 업그레이드
        apt-mark unhold kubeadm
        apt-get update && apt-get install -y kubeadm=1.27.1-00
        apt-mark hold kubeadm
        
        # 노드 업그레이드
        kubeadm upgrade node
        
        # kubelet 및 kubectl 업그레이드
        apt-mark unhold kubelet kubectl
        apt-get install -y kubelet=1.27.1-00 kubectl=1.27.1-00
        apt-mark hold kubelet kubectl
        
        # kubelet 재시작
        systemctl daemon-reload
        systemctl restart kubelet
EOF
    
    # 노드 uncordon
    kubectl uncordon ${node#node/}
    
    # 노드 상태가 Ready가 될 때까지 대기
    kubectl wait --for=condition=Ready node/${node#node/} --timeout=300s
done
```

<br/>

## 4. 업그레이드 완료 확인 및 테스트
```bash
# 업그레이드 결과 기록
echo "클러스터 업그레이드 결과" > /opt/upgrade-result.txt
echo "----------------------" >> /opt/upgrade-result.txt

# 노드 버전 확인
echo -e "\n1. 업그레이드 후 노드 상태:" >> /opt/upgrade-result.txt
kubectl get nodes >> /opt/upgrade-result.txt

# 시스템 파드 상태 확인
echo -e "\n2. 시스템 파드 상태:" >> /opt/upgrade-result.txt
kubectl get pods -n kube-system >> /opt/upgrade-result.txt

# 워크로드 상태 확인
echo -e "\n3. 워크로드 상태:" >> /opt/upgrade-result.txt
kubectl get pods --all-namespaces >> /opt/upgrade-result.txt

# 기본 기능 테스트
echo -e "\n4. 기능 테스트:" >> /opt/upgrade-result.txt

# 파드 생성 테스트
kubectl run test-pod --image=nginx
kubectl get pod test-pod -o wide >> /opt/upgrade-result.txt

# 서비스 생성 테스트
kubectl expose pod test-pod --port=80 --type=ClusterIP
kubectl get svc test-pod >> /opt/upgrade-result.txt

# 테스트 리소스 정리
kubectl delete pod test-pod
kubectl delete svc test-pod
```

<br/>

## 주의사항:
1. 업그레이드 전 모든 중요 데이터 백업
2. 업그레이드는 한 번에 한 단계씩만 수행
3. 각 노드 업그레이드 전 워크로드 drain 필수
4. 업그레이드 중 서비스 중단 시간 최소화
5. 모든 컴포넌트의 버전 호환성 확인
6. 업그레이드 실패 시 롤백 계획 준비
