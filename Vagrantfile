
# vi: set ft=ruby :

Vagrant.configure("2") do |config|
  # Base box
  config.vm.box = "geerlingguy/ubuntu2004"
  config.vm.hostname = "yolo-dev"

  # Create a private network
  config.vm.network "private_network", type: "dhcp"

  # VirtualBox provider config
  config.vm.provider "virtualbox" do |vb|
    vb.memory = "2048"
    vb.cpus = 2
  end

  # Ansible provisioning
  config.vm.provision "ansible" do |ansible|
    ansible.playbook = "playbook.yml"
    ansible.inventory_path = "hosts"
    ansible.become = true
    ansible.verbose = "vv"
    ansible.compatibility_mode = "2.0" 
  end
end