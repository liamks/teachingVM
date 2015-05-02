Vagrant.configure(2) do |config|
  config.vm.box = "ubuntu/trusty64"
  config.vm.define "TeachingVM"

  config.vm.network "forwarded_port", guest: 80, host: 5010

  # Ruby on Rails default port
  config.vm.network "forwarded_port", guest: 3000, host: 3001


  config.vm.synced_folder "~/Documents", "/home/vagrant/documents"

  config.vm.provider "virtualbox" do |vb|
    vb.memory = "2048"
    vb.cpus = "2"
  end

  config.vm.provision :ansible do |ansible|
    ansible.groups = {
      "dev" => ["TeachingVM"]
    }

    ansible.playbook = "provisioner/teaching-vm.yml"
  end
end