# -*- mode: ruby -*-
# vi: set ft=ruby :


numberRnode=2
vmBox="bento/ubuntu-20.04"

Vagrant.configure("2") do |config|
  # master server
  config.vm.define "rmaster" do |rmaster|
    rmaster.vm.box = "#{vmBox}"
    rmaster.vm.hostname = "Rmaster"
    rmaster.vm.provision "docker"
    rmaster.vm.box_url = "#{vmBox}"
    rmaster.vm.network :private_network, ip: "192.168.56.200"
    rmaster.vm.provider :virtualbox do |v|
      v.customize ["modifyvm", :id, "--natdnshostresolver1", "on"]
      v.customize ["modifyvm", :id, "--memory", 4096]
      v.customize ["modifyvm", :id, "--name", "Rmaster"]
      v.customize ["modifyvm", :id, "--cpus", "2"]
    end
    rmaster.vm.provision "shell", inline: <<-SHELL
      sed -i 's/ChallengeResponseAuthentication no/ChallengeResponseAuthentication yes/g' /etc/ssh/sshd_config    
      service ssh restart
      if [ ! -f "/usr/bin/docker" ];then
        curl -s -fsSL https://get.docker.com | sh; 
      fi
      systemctl restart docker
      docker run --privileged -d --restart=unless-stopped -p 80:80 -p 443:443 rancher/rancher
      swapoff -a
      sed -i.bak -r 's/(.+ swap .+)/#\1/' /etc/fstab
    SHELL
  end

  # Noeuds rancher
  (1..numberRnode).each do |i|
    config.vm.define "rnode#{i}" do |rnode|
      rnode.vm.box = "#{vmBox}"
      rnode.vm.hostname = "Rnode#{i}"
      rnode.vm.provision "docker"
      rnode.vm.box_url = "#{vmBox}"
      rnode.vm.network "private_network", ip: "192.168.56.23#{i}"
      rnode.vm.provider :virtualbox do |v|
        v.customize ["modifyvm", :id, "--natdnshostresolver1", "on"]
        v.customize ["modifyvm", :id, "--memory", 4096]
        v.customize ["modifyvm", :id, "--name", "Rnode#{i}"]
        v.customize ["modifyvm", :id, "--cpus", "2"]
      end
      rnode.vm.provision "shell", inline: <<-SHELL
        sed -i 's/ChallengeResponseAuthentication no/ChallengeResponseAuthentication yes/g' /etc/ssh/sshd_config    
        service ssh restart
        swapoff -a
        sed -i.bak -r 's/(.+ swap .+)/#\1/' /etc/fstab
      SHELL
    end
  end
end
