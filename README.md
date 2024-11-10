# CKA 대비 VMWare 기반 쿠버네티스 클러스터 구성


### 클러스터 구성

Cluster	  Member	              CNI	      IP Range        Description
k8s	      1 master, 2 workers	  flannel	  10.244.0.0/16   k8s cluster
hk8s	    1 master, 2 workers	  calico	  192.168.0.0/16  k8s cluster
bk8s	    1 master, 1 worker	  flannel	  10.245.0.0/16   k8s cluster
wk8s	    1 master, 2 workers	  flannel	  10.246.0.0/16   k8s cluster
ek8s    	1 master, 2 workers	  flannel 	10.247.0.0/16   k8s cluster
ik8s	    1 master, 1 base node	loopback	-               k8s cluster – worker node inactive

자세한 사항은 [구성 설명] 참고

---
### 실행 방법
```bash
vagrant up
vagrant status

vagrant ssh <vm_name>
vagrant halt <vm_name>      # 모든/특정 가상머신 종료
vagrant suspend <vm_name>   # 모든/특정 가상머신 중지
# restart 없음 -> halt & up

vagrant snapshot save <snapshot_name> --all         # 모든 가상머신 스냅샷
vagrant snapshot save <vm_name> <snapshot_name>     # 특정 가상머신 스냅샷
vagrant snapshot restore <vm_name> <snapshot_name>  # 특정 가상머신 스냅샷 복원

vagrant destroy <vm_name>   # 특정 가상머신 삭제
vagrant destroy -f          # 모든 가상머신 삭제
rm -rf .vagrant

vagrant reload              # Vagrant 네트워크 설정 초기화

```
https://developer.hashicorp.com/vagrant/docs/v2.4.1/share
https://portal.cloud.hashicorp.com/vagrant/discover?architectures=amd64&providers=vmware_desktop&query=ubuntu




---


#### 구성 설명
- 각 클러스터의 마스터 노드에서 kubeadm init 명령어를 실행
  고유한 --pod-network-cidr 값을 지정해 네트워크 대역이 겹치지 않도록 설정
- flannel과 calico CNI 설정을 각 클러스터에 맞게 적용
- ik8s 클러스터의 base 노드는 네트워크 연결이 없도록 루프백으로 설정하여 worker node inactive 상태를 유지합니다.
- 각 클러스터에 고유한 네트워크 대역을 할당하여 네트워크가 겹치지 않도록 설정해야 합니다.
- 각 클러스터의 --pod-network-cidr 설정을 다르게 지정하여 서로 충돌하지 않게 합니다.


#### [참고] CNI별 기본 네트워크 대역
- Calico  : 192.168.0.0/16
- flannel : 10.244.0.0/16

```bash
# Calico 네트워크 구성 예시
- name: CALICO_IPV4POOL_CIDR
  value: "192.168.0.0/16"


# Flannel 네트워크 구성 예시
net-conf.json: |
  {
    "Network": "10.244.0.0/16",
    "Backend": {
      "Type": "vxlan"
    }
  }

# ---
sudo kubeadm init --pod-network-cidr=10.244.0.0/16
sudo kubeadm init --pod-network-cidr=192.168.0.0/16
```

---
## Vagrant 2.4.2 에러 - conflicting dependencies logger (= 1.6.0) and logger (= 1.6.1)

결론적으로 2.4.2 버전은 롤백이 필요해 보인다는 이야기도 나왔다.

### 참고자료
https://stackoverflow.com/questions/18908708/installing-rubygems-in-windows
https://rubyinstaller.org/downloads/
https://developer.hashicorp.com/vagrant/install/vmware

### 증상
# 아래 두 명령어 같은 에러 발생
# vagrant plugin install vagrant-vmware-desktop
# vagrant plugin install vagrant-vmware-desktop --plugin-clean-sources --plugin-source https://rubygems.org

```powershell
PS C:\Users\Sungmin> vagrant plugin install vagrant-vmware-desktop
Installing the 'vagrant-vmware-desktop' plugin. This can take a few minutes...
Vagrant failed to properly resolve required dependencies. These
errors can commonly be caused by misconfigured plugin installations
or transient network issues. The reported error is:

conflicting dependencies logger (= 1.6.0) and logger (= 1.6.1)
  Activated logger-1.6.1
  which does not match conflicting dependency (= 1.6.0)

  Conflicting dependency chains:
    logger (= 1.6.1), 1.6.1 activated

  versus:
    logger (= 1.6.0)

  Gems matching logger (= 1.6.0):
    logger-1.6.0
```


### 해결 방법 - 2.4.1 버전 설치
1. 2.4.2 버전 삭제
2. vmware 플러그인 설치
  - 아래 링크 참고
  - https://developer.hashicorp.com/vagrant/docs/providers/vmware/installation

