# -*- mode: ruby -*-
# vi: set ft=ruby :
ENV['VAGRANT_SERVER_URL'] = 'https://vagrant.elab.pro'
ENV['VAGRANT_DISABLE_VBOXSYMLINKCREATE'] = '1'  # Добавляем для VirtualBox

BoxName = "bento/ubuntu-24.04"
Provider = "virtualbox"
SSH_PUB_KEY = "./lab.pub"

VirtMachine = Struct.new(:name, :ip, :cpus, :memory)
ansible = VirtMachine.new("ansible-host", "10.10.10.20", 2, 2048)
worker = VirtMachine.new("task-service", "10.10.10.30", 4, 2048)
vms = [ansible, worker]

Vagrant.configure("2") do |config|
  vms.each do |i|
    config.vm.define "#{i.name}" do |cfg|
      cfg.vm.box = "#{BoxName}"
      cfg.vm.hostname = "#{i.name}.local"
      
      # Указываем, что нужно создать второй интерфейс (eth1)
      cfg.vm.network "private_network", virtualbox__intnet: "vagrant-network", auto_config: false
      
      cfg.vm.provision "file", source: "#{SSH_PUB_KEY}", destination: "~/.ssh/ansible_key.pub"
      cfg.vm.provision "shell", inline: <<-SHELL
        mkdir -p /home/vagrant/.ssh
        cat /home/vagrant/.ssh/ansible_key.pub >> /home/vagrant/.ssh/authorized_keys
        chmod 600 /home/vagrant/.ssh/authorized_keys
        
        # Настройка сети через netplan (полностью заменяем config.vm.network)
        cat > /etc/netplan/60-vagrant.yaml <<EOF
network:
  version: 2
  ethernets:
    eth0:
      dhcp4: true
    eth1:
      addresses: [${i.ip}/24]
EOF
        
        # Применяем настройки сети
        netplan apply
        
        # Перезапускаем сеть (на всякий случай)
        systemctl restart systemd-networkd
      SHELL
      
      cfg.vm.provider "#{Provider}" do |v|
        v.cpus = "#{i.cpus}"
        v.memory = "#{i.memory}"
        v.name = "#{i.name}"
      end
    end
  end
end