Vagrant.configure(2) do |config|
  config.vm.box = "ubuntu/trusty64"

  config.vm.network "forwarded_port", guest: 80, host: 5010

  config.vm.provider "virtualbox" do |vb|
    vb.memory = "2048"
    vb.cpus = "2"
  end

  config.vm.provision :ansible do |ansible|
      ansible.groups = {
        "dev" => ["default"],
        "dev" => ["default"]
    }

    ansible.playbook = "provisioner/teaching-vm.yml"
  end
end