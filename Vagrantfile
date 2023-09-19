Vagrant.configure("2") do |config|
  config.vm.define "node1" do |node1_config|
    node1_config.vm.box = "ubuntu/jammy64"
    node1_config.vm.provision "ansible", playbook: "ubuntu.yml"
    node1_config.vm.network "private_network", type: "dhcp"
    node1_config.vm.provider "virtualbox" do |vb|
      vb.memory = "2048"
    end
  end
end

