# -*- mode: ruby -*-
# vi: set ft=ruby :

VAGRANTFILE_API_VERSION = "2"

# Define clusters
CLUSTERS = {
  "operations" => {
    "control" => { ip: "192.168.116.149" },
    "worker"  => { ip: "192.168.116.150" }
  },
  "production-us-1" => {
    "control" => { ip: "192.168.116.159" },
    "worker"  => { ip: "192.168.116.160" }
  }
}

Vagrant.configure(VAGRANTFILE_API_VERSION) do |config|
  config.vm.box = "bento/ubuntu-20.04-arm64"

  # SSH settings
  config.ssh.insert_key = false

  CLUSTERS.each do |cluster_name, nodes|
    nodes.each do |role, opts|
      node_name = "#{cluster_name}-#{role}"
      config.vm.define node_name do |node|
        node.vm.hostname = node_name
        node.vm.network "private_network", ip: opts[:ip]

        node.vm.provider "virtualbox" do |vb|
          vb.memory = "4096"
          vb.cpus = 2
        end

        node.vm.provision "shell", inline: <<-SHELL
          #!/bin/bash
          set -eux

          # Disable swap
          sudo swapoff -a
          sudo sed -i '/ swap / s/^/#/' /etc/fstab

          # Enable IP forwarding
          sudo sysctl -w net.ipv4.ip_forward=1
          echo "net.ipv4.ip_forward=1" | sudo tee -a /etc/sysctl.conf

          # Load br_netfilter and set required sysctls for Flannel
          sudo modprobe br_netfilter
          cat <<EOF | sudo tee /etc/sysctl.d/k8s.conf
net.bridge.bridge-nf-call-ip6tables = 1
net.bridge.bridge-nf-call-iptables = 1
EOF
          sudo sysctl --system

          # Install dependencies
          sudo apt-get update
          sudo apt-get install -y apt-transport-https ca-certificates curl gnupg lsb-release software-properties-common

          # Install containerd
          sudo apt-get install -y containerd
          sudo systemctl enable --now containerd

          # Add Kubernetes apt repo (v1.31 stable)
          sudo mkdir -p /etc/apt/keyrings
          curl -fsSL https://pkgs.k8s.io/core:/stable:/v1.31/deb/Release.key | sudo gpg --dearmor -o /etc/apt/keyrings/kubernetes-apt-keyring.gpg
          echo "deb [signed-by=/etc/apt/keyrings/kubernetes-apt-keyring.gpg] https://pkgs.k8s.io/core:/stable:/v1.31/deb/ /" | sudo tee /etc/apt/sources.list.d/kubernetes.list

          sudo apt-get update
          sudo apt-get install -y kubelet kubeadm kubectl kubernetes-cni cri-tools
          sudo apt-mark hold kubelet kubeadm kubectl

          # Initialize control-plane nodes
          if [[ "#{role}" == "control" ]]; then
            sudo kubeadm init --pod-network-cidr=10.244.0.0/16 --apiserver-advertise-address=#{opts[:ip]} --kubernetes-version=v1.31.13

            # Configure kubectl for vagrant user
            mkdir -p $HOME/.kube
            sudo cp -i /etc/kubernetes/admin.conf $HOME/.kube/config
            sudo chown $(id -u):$(id -g) $HOME/.kube/config

            # Install Flannel CNI
            kubectl apply -f https://raw.githubusercontent.com/flannel-io/flannel/master/Documentation/kube-flannel.yml

            # Output join command for workers
            kubeadm token create --print-join-command > /vagrant/#{cluster_name}-join.sh
          fi

          # Join worker nodes
          if [[ "#{role}" == "worker" ]]; then
            until [[ -f /vagrant/#{cluster_name}-join.sh ]]; do sleep 5; done
            sudo bash /vagrant/#{cluster_name}-join.sh
          fi
        SHELL
      end
    end
  end
end
