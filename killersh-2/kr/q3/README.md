# Kubelet 인증서 정보 확인

문제에서 kubeadm과 TLS bootstrapping을 사용하여 cka5248-node1 노드가 클러스터에 추가되었습니다.
다음 인증서들의 Issuer와 Extended Key Usage 값을 찾아야 합니다:

- Kubelet Client Certificate: kube-apiserver로의 outgoing 연결에 사용
- Kubelet Server Certificate: kube-apiserver로부터의 incoming 연결에 사용

찾은 정보는 /opt/course/3/certificate-info.txt 파일에 작성해야 합니다.

<br/>

## 1. 작업 디렉토리 생성
```bash
# 워커 노드로 접속
ssh cka5248-node1

# 결과 저장을 위한 디렉토리 생성
sudo mkdir -p /opt/course/3
```

<br/>

## 2. Kubelet 인증서 위치 확인
```bash
# kubelet 설정 파일 확인
sudo cat /var/lib/kubelet/config.yaml

# 일반적인 인증서 위치:
# Client Certificate: /var/lib/kubelet/pki/kubelet-client-current.pem
# Server Certificate: /var/lib/kubelet/pki/kubelet.crt
```

<br/>

## 3. 인증서 정보 추출
```bash
# Client Certificate 정보 확인
echo "Kubelet Client Certificate:" > /opt/course/3/certificate-info.txt
echo "------------------------" >> /opt/course/3/certificate-info.txt
sudo openssl x509 -in /var/lib/kubelet/pki/kubelet-client-current.pem -noout -issuer -ext extendedKeyUsage >> /opt/course/3/certificate-info.txt

# Server Certificate 정보 확인
echo -e "\nKubelet Server Certificate:" >> /opt/course/3/certificate-info.txt
echo "------------------------" >> /opt/course/3/certificate-info.txt
sudo openssl x509 -in /var/lib/kubelet/pki/kubelet.crt -noout -issuer -ext extendedKeyUsage >> /opt/course/3/certificate-info.txt
```

<br/>

## 4. 결과 확인
```bash
# 결과 파일 내용 확인
cat /opt/course/3/certificate-info.txt

# 파일 권한 확인
ls -l /opt/course/3/certificate-info.txt
```

<br/>

## 예상 결과 형식:
```bash

Kubelet Client Certificate:
issuer=CN=kubernetes
X509v3 Extended Key Usage:
TLS Web Client Authentication


Kubelet Server Certificate:
issuer=CN=kubernetes
X509v3 Extended Key Usage:
TLS Web Server Authentication
```


<br/>

## 주의사항:
1. 인증서 파일 경로가 다를 수 있으므로 kubelet 설정 확인 필요
2. 인증서 파일 접근을 위해 sudo 권한 필요
3. openssl 명령어가 설치되어 있어야 함
4. 결과 파일의 형식이 읽기 쉽도록 구성
