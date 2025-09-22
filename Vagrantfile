Vagrant.configure("2") do |config|
  config.vm.box = "bento/ubuntu-20.04-arm64"

  config.vm.provider "vmware_desktop" do |v|
    v.memory = "4096"
    v.cpus = 2
  end

  # Common Kubernetes setup script
  kube_setup_script = <<-SHELL
    sudo apt-get update
    sudo apt-get install -y apt-transport-https ca-certificates curl gpg lsb-release

    # Add Kubernetes repo (v1.31)
    sudo mkdir -p /etc/apt/keyrings
    curl -fsSL https://pkgs.k8s.io/core:/stable:/v1.31/deb/Release.key \
      | sudo gpg --dearmor -o /etc/apt/keyrings/kubernetes-apt-keyring.gpg
    echo "deb [signed-by=/etc/apt/keyrings/kubernetes-apt-keyring.gpg] \
      https://pkgs.k8s.io/core:/stable:/v1.31/deb/ /" \
      | sudo tee /etc/apt/sources.list.d/kubernetes.list

    sudo apt-get update
    sudo DEBIAN_FRONTEND=noninteractive apt-get install -y kubelet kubeadm kubectl containerd
    sudo apt-mark hold kubelet kubeadm kubectl
    sudo systemctl enable --now containerd
  SHELL

  #############################
  # Cluster 1: operations
  #############################

  config.vm.define "operations-control" do |node|
    node.vm.network "private_network", type: "dhcp"
    node.vm.hostname = "operations-control"
    node.vm.provision "shell", inline: <<-SHELL
      #{kube_setup_script}

      # Disable swap and enable IP forwarding
      sudo swapoff -a
      sudo sed -i '/ swap / s/^/#/' /etc/fstab
      sudo sysctl -w net.ipv4.ip_forward=1
      echo "net.ipv4.ip_forward=1" | sudo tee -a /etc/sysctl.conf

      # Initialize Kubernetes cluster
      sudo kubeadm init --pod-network-cidr=10.244.0.0/16

      # Configure kubeconfig
      mkdir -p $HOME/.kube
      sudo cp -i /etc/kubernetes/admin.conf $HOME/.kube/config
      sudo chown $(id -u):$(id -g) $HOME/.kube/config

      # Install Flannel CNI
      kubectl apply -f https://raw.githubusercontent.com/coreos/flannel/master/Documentation/kube-flannel.yml

      # Generate join command for worker
      kubeadm token create --print-join-command > /vagrant/operations-join.sh
    SHELL
  end

  config.vm.define "operations-worker" do |node|
    node.vm.network "private_network", type: "dhcp"
    node.vm.hostname = "operations-worker"
    node.vm.provision "shell", inline: kube_setup_script

    node.vm.provision "shell", inline: <<-SHELL
      # Disable swap and enable IP forwarding
      sudo swapoff -a
      sudo sed -i '/ swap / s/^/#/' /etc/fstab
      sudo sysctl -w net.ipv4.ip_forward=1
      echo "net.ipv4.ip_forward=1" | sudo tee -a /etc/sysctl.conf

      # Wait for join command
      attempts=0
      while [ ! -f /vagrant/operations-join.sh ]; do
        echo "Waiting for operations join command..."
        sleep 5
        attempts=$((attempts+1))
        if [ $attempts -gt 60 ]; then
          echo "Timeout waiting for join file! Check operations-control logs."
          exit 1
        fi
      done

      sudo bash /vagrant/operations-join.sh
    SHELL
  end

  #############################
  # Cluster 2: production-us-1
  #############################

  config.vm.define "production-us-1-control" do |node|
    node.vm.network "private_network", type: "dhcp"
    node.vm.hostname = "production-us-1-control"
    node.vm.provision "shell", inline: <<-SHELL
      #{kube_setup_script}

      # Disable swap and enable IP forwarding
      sudo swapoff -a
      sudo sed -i '/ swap / s/^/#/' /etc/fstab
      sudo sysctl -w net.ipv4.ip_forward=1
      echo "net.ipv4.ip_forward=1" | sudo tee -a /etc/sysctl.conf

      # Initialize cluster with unique pod-network CIDR
      sudo kubeadm init --pod-network-cidr=10.245.0.0/16

      # Configure kubeconfig
      mkdir -p $HOME/.kube
      sudo cp -i /etc/kubernetes/admin.conf $HOME/.kube/config
      sudo chown $(id -u):$(id -g) $HOME/.kube/config

      # Install Flannel CNI
      kubectl apply -f https://raw.githubusercontent.com/coreos/flannel/master/Documentation/kube-flannel.yml

      # Generate join command for worker
      kubeadm token create --print-join-command > /vagrant/production-us-1-join.sh
    SHELL
  end

  config.vm.define "production-us-1-worker" do |node|
    node.vm.network "private_network", type: "dhcp"
    node.vm.hostname = "production-us-1-worker"
    node.vm.provision "shell", inline: kube_setup_script

    node.vm.provision "shell", inline: <<-SHELL
      # Disable swap and enable IP forwarding
      sudo swapoff -a
      sudo sed -i '/ swap / s/^/#/' /etc/fstab
      sudo sysctl -w net.ipv4.ip_forward=1
      echo "net.ipv4.ip_forward=1" | sudo tee -a /etc/sysctl.conf

      # Wait for join command
      attempts=0
      while [ ! -f /vagrant/production-us-1-join.sh ]; do
        echo "Waiting for production-us-1 join command..."
        sleep 5
        attempts=$((attempts+1))
        if [ $attempts -gt 60 ]; then
          echo "Timeout waiting for join file! Check production-us-1-control logs."
          exit 1
        fi
      done

      sudo bash /vagrant/production-us-1-join.sh
    SHELL
  end
end
