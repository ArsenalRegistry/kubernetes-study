# Master Node에 Kubernetes 설치(10.100.0.105 에서 진행)



#### Swap disabled


swapoff -a && sed -i '/swap/s/^/#/' /etc/fstab


#### docker 삭제


for pkg in docker.io docker-doc docker-compose podman-docker containerd runc; do sudo apt-get remove $pkg; done



#### 컨테이너 런타임 설치


sudo apt-get update


sudo apt-get install ca-certificates curl gnupg



#### Docker GPG 키 추가


sudo install -m 0755 -d /etc/apt/keyrings
curl -fsSL https://download.docker.com/linux/ubuntu/gpg | sudo gpg --dearmor -o /etc/apt/keyrings/docker.gpg
sudo chmod a+r /etc/apt/keyrings/docker.gpg

4) Docker 레포지토리 설정
echo \
  "deb [arch="$(dpkg --print-architecture)" signed-by=/etc/apt/keyrings/docker.gpg] https://download.docker.com/linux/ubuntu \
  "$(. /etc/os-release && echo "$VERSION_CODENAME")" stable" | \
  sudo tee /etc/apt/sources.list.d/docker.list > /dev/null

5) 컨테이너 런타임 구성요소 설치

### 패키지 업데이트
sudo apt-get update

### Docker Engine 및 Container Runtime 구성 요소 설치
sudo apt-get install docker-ce docker-ce-cli containerd.io docker-buildx-plugin docker-compose-plugin

6) kubernetes 구성요소 설치 (패키지 업데이트 및 필수 전체 패키지 설치)
sudo apt-get update
sudo apt-get install -y apt-transport-https ca-certificates curl

7) Google Cloud 공개 서명 키 다운로드
curl -fsSL https://dl.k8s.io/apt/doc/apt-key.gpg | sudo gpg --dearmor -o /etc/apt/keyrings/kubernetes-archive-keyring.gpg

8) k8s 레포지토리 추가
echo "deb [signed-by=/etc/apt/keyrings/kubernetes-archive-keyring.gpg] https://apt.kubernetes.io/ kubernetes-xenial main" | sudo tee /etc/apt/sources.list.d/kubernetes.list

9) 패키지 업데이트 후 kubeadm, kubectl, kubelet 설치

=> 패키지 저장소 업데이트가 안됨(URL 변경)

# 기존 Kubernetes 소스 리스트 파일 삭제 또는 변경
sudo rm /etc/apt/sources.list.d/kubernetes.list

# Kubernetes 소스 리스트를 다시 추가
curl -fsSL https://pkgs.k8s.io/core:/stable:/v1.30/deb/Release.key | sudo gpg --dearmor -o /etc/apt/keyrings/kubernetes-apt-keyring.gpg

echo 'deb [signed-by=/etc/apt/keyrings/kubernetes-apt-keyring.gpg] https://pkgs.k8s.io/core:/stable:/v1.30/deb/ /' | sudo tee /etc/apt/sources.list.d/kubernetes.list

# 패키지 리스트 업데이트
sudo apt-get update

# 다운로드
sudo apt-get install -y kubelet kubeadm kubectl

# containerd 구성 파일 생성
containerd config default | tee /etc/containerd/config.toml

# vim을 통해 config 파일 오픈
vim /etc/containerd/config.toml

# config.toml

...

114 [plugins."io.containerd.grpc.v1.cri".containerd.runtimes.runc.options]
115		BinaryName = ""
116 	CriuImagePath = ""
117 	CriuPath = ""
118 	CriuWorkPath = ""
119 	IoGid = 0
120 	IoUid = 0
121 	NoNewKeyring = false
122 	NoPivotRoot = false
123 	Root = ""
124 	ShimCgroup = ""
125 	SystemdCgroup = false # true로 변경

# 편집한 내용을 저장 후 containerd 데몬과 kubelet 데몬을 재시작하여 변경사항을 반영

systemctl restart containerd
systemctl restart kubelet

****** Masternode control plane 구성

#### kubeadm init 명령어 결과로 나온 join 명령어 저장 필수 => 이후 Wokrer Node 조인에서 사용

kubeadm init

mkdir -p $HOME/.kube
sudo cp -i /etc/kubernetes/admin.conf $HOME/.kube/config
sudo chown $(id -u):$(id -g) $HOME/.kube/config

## CNI 설치

kubectl apply -f https://docs.projectcalico.org/manifests/calico.yaml

## Master 노드 설치 확인

kubectl get nodes

############# Worker Node에서는 kubeadm init 전까지 진행하고 아래 조인 명령어 입력

kubeadm join 10.100.0.105:6443 --token 2h0fum.jea9l5knwfn593sz --discovery-token-ca-cert-hash sha256:a416aa1649020ae7eb926cbcf8726755e37c328409d1ed69b4a2e951e0e31818

kubectl get nodes로 Master Node Ready 상태 확인

Worker Node(10.100.0.106 에서 진행)

위에 순서대로 kubeadm init 전까지 진행하고

위에서 저장해놨던 조인 명령어 사용해서 Worker Node 조인
