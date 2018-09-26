# -*- mode: ruby -*-
# vi: set ft=ruby :

Vagrant.configure(2) do |config|

  # Use SSH-Key of host machine
  config.ssh.forward_agent = true

  # Disable SharedFolder by default.
  config.vm.synced_folder ".", "/vagrant", disabled: true

  # General Provider Settings
  config.vm.provider "virtualbox" do |vb|
    # Disable GUI --> No GUI Window will pop up on start, could be opened on virtual Box app via double click on machine.
    vb.gui = false

    # special VirtualBox Customizations:
    vb.customize ["modifyvm", :id,
                  "--nicpromisc1", "allow-all",
                  "--nicpromisc2", "allow-all",
                  "--cpuexecutioncap", "50",
                  # Make a Structure within Virtualbox to group Machines
                  "--groups", "/Vagrant/LMU/Jenkins"
                 ]
  end

  config.vm.define "jenkinsmaster1", primary: true, autostart: true do |node|
    node.vm.box = "debian/stretch64"
    node.vm.hostname = "jenkinsmaster1"

    node.vm.provider "virtualbox" do |vb|
      vb.name = "JenkinsMaster1"
      vb.memory = 8192
      vb.cpus = 8
    end
    node.vm.network :public_network, ip: "192.168.5.100", netmask: "255.255.255.0"
  end

  (1..10).each do |i|
    config.vm.define "jenkinsslave#{i}", primary: false, autostart: false do |node|
      node.vm.box = "debian/stretch64"
      node.vm.hostname = "jenkinsslave#{i}"

      node.vm.provider "virtualbox" do |vb|
        vb.memory = 2048
        vb.cpus = 2
      end
      node.vm.network "private_network", ip: "192.168.5.#{110 + i}", netmask: "255.255.255.0"
    end
  end

  config.vm.provision "bootstrap", type: "shell" do |s|
    s.inline = "echo Bootstrap Machine"
  end
  config.vm.provision "ssh-key", type: "shell", privileged: false do |s|
    ssh_pub_key = File.readlines("#{Dir.home}/.ssh/id_rsa.pub").first.strip
    s.inline = <<-SHELL
      echo #{ssh_pub_key} >> ~/.ssh/authorized_keys
    SHELL
  end
  config.vm.provision "base-preseed", type: "ansible" do |ansible|
    ansible.compatibility_mode = "2.0"
    ansible.playbook = "ansible/base-preseed.yml"
    #ansible.verbose = "vvv"
  end
  config.vm.provision "application", type: "ansible" do |ansible| # run: "never"
    ansible.compatibility_mode = "2.0"
    ansible.playbook = "ansible/jenkins.yml"
    ansible.groups = {
      "jenkinsmasters" => ["jenkinsmaster1",],
      "jenkinsslaves" => ["jenkinsslave1"],
    }
    #ansible.verbose = "vvvv"
    ansible.verbose = "vvv"
    #ansible.verbose = "vv"
    #ansible.verbose = "v"
    #ansible.verbose = ""
    #ansible.start_at_task = ""
    #ansible.stop_at_task = ""
    #ansible.limit = "all"
    #ansible.tags = ["setup", "configuration", "update"]
    #ansible.skip_tags = ["update"]
    ansible.extra_vars = {
      ansible_connection: 'ssh',
      ansible_ssh_args: '-o ForwardAgent=yes',
      ansible_ssh_private_key_file: ['~/.ssh/id_rsa']
    }
  end

end
