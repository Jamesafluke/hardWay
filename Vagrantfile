$setup_user_script = <<-'SCRIPT'
  USER_NAME="jfluckiger"
  SSH_KEY_DATA="" # <--- !!! THE VAGRANTFILE WILL INJECT THIS !!!

  # Create user if they don't exist
  if ! id "$USER_NAME" &>/dev/null; then
    echo "Creating user $USER_NAME..."
    useradd -m -s /bin/bash "$USER_NAME"
    
    # Give them passwordless sudo capabilities (standard for admin/jumpbox users)
    echo "$USER_NAME ALL=(ALL) NOPASSWD:ALL" > /etc/sudoers.d/$USER_NAME
    chmod 0441 /etc/sudoers.d/$USER_NAME
  fi

  # Set up SSH directory and authorized_keys file
  USER_HOME="/home/$USER_NAME"
  mkdir -p "$USER_HOME/.ssh"
  echo "$SSH_KEY_DATA" > "$USER_HOME/.ssh/authorized_keys"
  
  # Set proper permissions so SSH doesn't reject them
  chmod 700 "$USER_HOME/.ssh"
  chmod 600 "$USER_HOME/.ssh/authorized_keys"
  chown -R "$USER_NAME:$USER_NAME" "$USER_HOME/.ssh"
  echo "User $USER_NAME and SSH key configured successfully."
SCRIPT

Vagrant.configure("2") do |config|
  pub_key_path = File.expand_path("~/.ssh/id_ed25519.pub")
  my_public_key = File.exist?(pub_key_path) ? File.read(pub_key_path).strip : ""
  
  config.vm.define "jumpbox" do |jumpbox|
    jumpbox.vm.box = "debian/bookworm64"
    jumpbox.vm.hostname = "jumpbox"
    jumpbox.vm.disk :disk, size: "10GB", primary: true
    jumpbox.vm.provider "virtualbox" do |vb|
      vb.name = "jumpbox"
      vb.memory = "2048"
      vb.customize ["modifyvm", :id, "--nested-hw-virt", "on"]
      vb.customize ["modifyvm", :id, "--vram", "16"]
      vb.customize ["modifyvm", :id, "--graphicscontroller", "vmsvga"]
    end

    script_with_key = $setup_user_script.gsub('SSH_KEY_DATA=""', "SSH_KEY_DATA=\"#{my_public_key}\"")
    jumpbox.vm.provision "shell", inline: script_with_key
  end 

  config.vm.define "server" do |server|
    server.vm.box = "debian/bookworm64"
    server.vm.hostname = "server"
    server.vm.disk :disk, size: "20GB", primary: true
    server.vm.provider "virtualbox" do |vb|
      vb.name = "server"
      vb.memory = "2048"
      vb.customize ["modifyvm", :id, "--nested-hw-virt", "on"]
      vb.customize ["modifyvm", :id, "--vram", "16"]
      vb.customize ["modifyvm", :id, "--graphicscontroller", "vmsvga"]
    end

    script_with_key = $setup_user_script.gsub('SSH_KEY_DATA=""', "SSH_KEY_DATA=\"#{my_public_key}\"")
    server.vm.provision "shell", inline: script_with_key
  end 

  config.vm.define "node0" do |node0|
    node0.vm.box = "debian/bookworm64"
    node0.vm.hostname = "node0"
    node0.vm.disk :disk, size: "20GB", primary: true
    node0.vm.provider "virtualbox" do |vb|
      vb.name = "node0"
      vb.memory = "2048"
      vb.customize ["modifyvm", :id, "--nested-hw-virt", "on"]
      vb.customize ["modifyvm", :id, "--vram", "16"]
      vb.customize ["modifyvm", :id, "--graphicscontroller", "vmsvga"]
    end

    script_with_key = $setup_user_script.gsub('SSH_KEY_DATA=""', "SSH_KEY_DATA=\"#{my_public_key}\"")
    node0.vm.provision "shell", inline: script_with_key
  end 

  config.vm.define "node1" do |node1|
    node1.vm.box = "debian/bookworm64"
    node1.vm.hostname = "node1"
    node1.vm.disk :disk, size: "20GB", primary: true
    node1.vm.provider "virtualbox" do |vb|
      vb.name = "node1"
      vb.memory = "2048"
      vb.customize ["modifyvm", :id, "--nested-hw-virt", "on"]
      vb.customize ["modifyvm", :id, "--vram", "16"]
      vb.customize ["modifyvm", :id, "--graphicscontroller", "vmsvga"]
    end

    script_with_key = $setup_user_script.gsub('SSH_KEY_DATA=""', "SSH_KEY_DATA=\"#{my_public_key}\"")
    node1.vm.provision "shell", inline: script_with_key
  end
end