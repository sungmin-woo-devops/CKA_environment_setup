Vagrant.configure("2") do |config|
  # 공통 설정
  config.vm.box = "generic/ubuntu2204"
  
  # 클러스터별 구성
  # k8s(1m 2w, flannel), hk8s(1m 2w, calico), bk8s(1m 1w, flannel),
  # wk8s(1n 2w, flannel), ek8s(1m, 2w, flannel), ik8s(1m, 1base node, loopback, worker node inactive)
  
  # [1/6] k8s 클러스터 (1m 2w, flannel, 네트워크 대역: 10.244.0.0/16)
  config.vm.define "k8s-master" do |control_plane|
    control_plane.vm.hostname = "k8s-master"
    control_plane.vm.network "private_network", ip: "192.168.56.10"
    control_plane.vm.provider "vmware_desktop" do |vb|
      vb.memory = "4096"
      vb.cpus = 2
    end
  end
  (1..2).each do |i|
    config.vm.define "k8s-worker#{i}" do |worker|
      worker.vm.hostname = "k8s-worker#{i}"
      worker.vm.network "private_network", ip: "192.168.56.1#{i}"
      worker.vm.provider "vmware_desktop" do |vb|
        vb.memory = "2048"
        vb.cpus = 2
      end
    end
  end

  # [2/6] hk8s 클러스터 (1m 2w, calico, 네트워크 대역: 192.168.0.0/16)
  config.vm.define "hk8s-master" do |control_plane|
    control_plane.vm.hostname = "hk8s-master"
    control_plane.vm.network "private_network", ip: "192.168.56.20"
    control_plane.vm.provider "vmware_desktop" do |vb|
      vb.memory = "4096"
      vb.cpus = 2
    end
  end
  (1..2).each do |i|
    config.vm.define "hk8s-worker#{i}" do |worker|
      worker.vm.hostname = "hk8s-worker#{i}"
      worker.vm.network "private_network", ip: "192.168.56.2#{i}"
      worker.vm.provider "vmware_desktop" do |vb|
        vb.memory = "2048"
        vb.cpus = 2
      end
    end
  end

  # [3/6] bk8s 클러스터 (1m 1w, flannel, 네트워크 대역: 10.245.0.0/16)
  config.vm.define "bk8s-master" do |control_plane|
    control_plane.vm.hostname = "bk8s-master"
    control_plane.vm.network "private_network", ip: "192.168.56.30"
    control_plane.vm.provider "vmware_desktop" do |vb|
      vb.memory = "4096"
      vb.cpus = 2
    end
  end
  config.vm.define "bk8s-worker" do |worker|
    worker.vm.hostname = "bk8s-worker"
    worker.vm.network "private_network", ip: "192.168.56.31"
    worker.vm.provider "vmware_desktop" do |vb|
      vb.memory = "2048"
      vb.cpus = 2
    end
  end
  
  # [4/6] wk8s 클러스터 (1m 2w, flannel, 네트워크 대역: 10.246.0.0/16)
  config.vm.define "wk8s-master" do |control_plane|
    control_plane.vm.hostname = "wk8s-master"
    control_plane.vm.network "private_network", ip: "192.168.56.40"
    control_plane.vm.provider "vmware_desktop" do |vb|
      vb.memory = "4096"
      vb.cpus = 2
    end
  end  
  (1..2).each do |i|
    config.vm.define "wk8s-worker#{i}" do |worker|
      worker.vm.hostname = "wk8s-worker#{i}"
      worker.vm.network "private_network", ip: "192.168.56.4#{i}"
      worker.vm.provider "vmware_desktop" do |vb|
        vb.memory = "2048"
        vb.cpus = 2
      end
    end
  end

  # [5/6] ek8s 클러스터 (1m 2w, flannel, 네트워크 대역: 10.247.0.0/16)
  config.vm.define "ek8s-master" do |control_plane|
    control_plane.vm.hostname = "ek8s-master"
    control_plane.vm.network "private_network", ip: "192.168.56.50"
    control_plane.vm.provider "vmware_desktop" do |vb|
      vb.memory = "4096"
      vb.cpus = 2
    end
  end  
  (1..2).each do |i|
    config.vm.define "ek8s-worker#{i}" do |worker|
      worker.vm.hostname = "ek8s-worker#{i}"
      worker.vm.network "private_network", ip: "192.168.56.5#{i}"
      worker.vm.provider "vmware_desktop" do |vb|
        vb.memory = "2048"
        vb.cpus = 2
      end
    end
  end

  # [6/6] ik8s 클러스터 (1m, 1 base node, loopback, worker node inactive)
  config.vm.define "ik8s-master" do |control_plane|
    control_plane.vm.hostname = "ik8s-master"
    control_plane.vm.network "private_network", ip: "192.168.56.60"
    control_plane.vm.provider "vmware_desktop" do |vb|
      vb.memory = "4096"
      vb.cpus = 2
    end
  end
  # ik8s 기본 노드 (Base Node) - Loopback CNI 적용
  config.vm.define "ik8s-base" do |base|
    base.vm.hostname = "ik8s-base"
    base.vm.network "private_network", type: "dhcp"
    base.vm.provider "vmware_desktop" do |vb|
      vb.memory = "2048"
      vb.cpus = 2
    end
  end

  # # Ansible로 수행
  # # CNI 설치 스크립트
  # config.vm.provision "shell", inline: <<-SHELL
  #   if [ "$(hostname)" == "k8s-master" ]; then
  #     sudo kubeadm init --pod-network-cidr=10.244.0.0/16
  #     sudo kubectl apply -f https://raw.githubusercontent.com/coreos/flannel/master/Documentation/kube-flannel.yml
  #   elif [ "$(hostname)" == "hk8s-master" ]; then
  #     sudo kubeadm init --pod-network-cidr=192.168.0.0/16
  #     sudo kubectl apply -f https://docs.projectcalico.org/manifests/calico.yaml
  #   elif [ "$(hostname)" == "bk8s-master" ]; then
  #     sudo kubeadm init --pod-network-cidr=10.245.0.0/16
  #     sudo kubectl apply -f https://raw.githubusercontent.com/coreos/flannel/master/Documentation/kube-flannel.yml
  #   elif [ "$(hostname)" == "wk8s-master" ]; then
  #     sudo kubeadm init --pod-network-cidr=10.246.0.0/16
  #     sudo kubectl apply -f https://raw.githubusercontent.com/coreos/flannel/master/Documentation/kube-flannel.yml
  #   elif [ "$(hostname)" == "ek8s-master" ]; then
  #     sudo kubeadm init --pod-network-cidr=10.247.0.0/16
  #     sudo kubectl apply -f https://raw.githubusercontent.com/coreos/flannel/master/Documentation/kube-flannel.yml
  #   fi
  # SHELL
end
