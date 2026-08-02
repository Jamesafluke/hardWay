# IP Address Definitions
jumpbox_ip = "10.240.0.10"
server_ip  = "10.240.0.11"
node0_ip   = "10.240.0.20"
node1_ip   = "10.240.0.21"

$setup_hosts_script = <<-'SCRIPT'
  cat <<'EOF' >> /etc/hosts

# Kubernetes Cluster Hosts
10.240.0.10 jumpbox.kubernetes.local jumpbox
10.240.0.11 server.kubernetes.local server
10.240.0.20 node0.kubernetes.local node0
10.240.0.21 node1.kubernetes.local node1
EOF
SCRIPT

$setup_user_script = <<-'SCRIPT'
  USER_NAME="jfluckiger"
  SSH_KEY_DATA="" # Injected dynamically by Vagrant

  # Create user if missing
  if ! id "$USER_NAME" &>/dev/null; then
    echo "Creating user $USER_NAME..."
    useradd -m -s /bin/bash "$USER_NAME"
    
    # Configure passwordless sudo
    echo "$USER_NAME ALL=(ALL) NOPASSWD:ALL" > /etc/sudoers.d/$USER_NAME
    chmod 0441 /etc/sudoers.d/$USER_NAME
  fi

  # Configure SSH access
  USER_HOME="/home/$USER_NAME"
  mkdir -p "$USER_HOME/.ssh"
  echo "$SSH_KEY_DATA" > "$USER_HOME/.ssh/authorized_keys"
  
  # Lock down file permissions
  chmod 700 "$USER_HOME/.ssh"
  chmod 600 "$USER_HOME/.ssh/authorized_keys"
  chown -R "$USER_NAME:$USER_NAME" "$USER_HOME/.ssh"
  
  echo "User $USER_NAME and SSH key configured successfully."
SCRIPT

Vagrant.configure("2") do |config|
  # Load SSH Public Key from host
  pub_key_path = File.expand_path("~/.ssh/id_ed25519.pub")
  my_public_key = File.exist?(pub_key_path) ? File.read(pub_key_path).strip : ""

  # Generate full user script with SSH key
  script_with_key = $setup_user_script.gsub('SSH_KEY_DATA=""', "SSH_KEY_DATA=\"#{my_public_key}\"")

  # --- GLOBAL PROVISIONERS (Runs on ALL VMs automatically) ---
  config.vm.provision "shell", inline: $setup_hosts_script
  config.vm.provision "shell", inline: script_with_key

config.vm.define "jumpbox" do |jumpbox|
    jumpbox.vm.box = "debian/bookworm64"
    jumpbox.vm.hostname = "jumpbox.kubernetes.local"
    jumpbox.vm.network "private_network", ip: jumpbox_ip
    jumpbox.vm.network "forwarded_port", guest: 22, host: 2220, id: "ssh", auto_correct: false
    jumpbox.vm.disk :disk, size: "10GB", primary: true

    jumpbox.vm.provider "virtualbox" do |vb|
      vb.name = "jumpbox"
      vb.memory = "2048"
      vb.customize ["modifyvm", :id, "--nested-hw-virt", "on"]
      vb.customize ["modifyvm", :id, "--vram", "16"]
      vb.customize ["modifyvm", :id, "--graphicscontroller", "vmsvga"]
    end

    # --- COPY PRIVATE KEY TO JUMPBOX ---
    jumpbox.vm.provision "file", 
      source: File.expand_path("C:/Users/james/.ssh/id_ed25519"), 
      destination: "/home/jfluckiger/.ssh/id_ed25519"

    jumpbox.vm.provision "shell", inline: <<-SCRIPT
      chown jfluckiger:jfluckiger /home/jfluckiger/.ssh/id_ed25519
      chmod 600 /home/jfluckiger/.ssh/id_ed25519
      echo "Private key copied and permissions set."
    SCRIPT
  end

  config.vm.define "server" do |server|
    server.vm.box = "debian/bookworm64"
    server.vm.hostname = "server.kubernetes.local"
    server.vm.network "private_network", ip: server_ip
    server.vm.network "forwarded_port", guest: 22, host: 2221, id: "ssh", auto_correct: false
    server.vm.disk :disk, size: "20GB", primary: true

    server.vm.provider "virtualbox" do |vb|
      vb.name = "server"
      vb.memory = "2048"
      vb.customize ["modifyvm", :id, "--nested-hw-virt", "on"]
      vb.customize ["modifyvm", :id, "--vram", "16"]
      vb.customize ["modifyvm", :id, "--graphicscontroller", "vmsvga"]
    end
  end 

  config.vm.define "node0" do |node0|
    node0.vm.box = "debian/bookworm64"
    node0.vm.hostname = "node0.kubernetes.local"
    node0.vm.network "private_network", ip: node0_ip 
    node0.vm.network "forwarded_port", guest: 22, host: 2222, id: "ssh", auto_correct: false
    node0.vm.disk :disk, size: "20GB", primary: true

    node0.vm.provider "virtualbox" do |vb|
      vb.name = "node0"
      vb.memory = "2048"
      vb.customize ["modifyvm", :id, "--nested-hw-virt", "on"]
      vb.customize ["modifyvm", :id, "--vram", "16"]
      vb.customize ["modifyvm", :id, "--graphicscontroller", "vmsvga"]
    end
  end 

  config.vm.define "node1" do |node1|
    node1.vm.box = "debian/bookworm64"
    node1.vm.hostname = "node1.kubernetes.local"
    node1.vm.network "private_network", ip: node1_ip
    node1.vm.network "forwarded_port", guest: 22, host: 2223, id: "ssh", auto_correct: false
    node1.vm.disk :disk, size: "20GB", primary: true

    node1.vm.provider "virtualbox" do |vb|
      vb.name = "node1"
      vb.memory = "2048"
      vb.customize ["modifyvm", :id, "--nested-hw-virt", "on"]
      vb.customize ["modifyvm", :id, "--vram", "16"]
      vb.customize ["modifyvm", :id, "--graphicscontroller", "vmsvga"]
    end
  end
end