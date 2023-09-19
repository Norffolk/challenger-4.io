# How to Provision VMs quickly with Vagrant and Ansible
## 1. Install Vagrant:
```
wget -O- https://apt.releases.hashicorp.com/gpg | sudo gpg --dearmor -o /usr/share/keyrings/hashicorp-archive-keyring.gpg
echo "deb [signed-by=/usr/share/keyrings/hashicorp-archive-keyring.gpg] https://apt.releases.hashicorp.com $(lsb_release -cs) main" | sudo tee /etc/apt/sources.list.d/hashicorp.list
sudo apt update && sudo apt install vagrant
```
## 2. Install Ansible:
```
sudo apt update && sudo apt install software-properties-common && sudo add-apt-repository --yes --update ppa:ansible/ansible && sudo apt install ansible
```
**3. use mkdir to create a directory for vagrant/ansible:**
```
mkdir ubuntu_jammy
```
**4. First, we need choose the OS on this site:**

``
https://app.vagrantup.com/boxes/search
``

*for example "ubuntu/jammy64"*

**5. To create a Vagrantfile with ubuntu/jammy64 package, use this command:**
```
vagrant init ubuntu/jammy64
```
**6. Now we can edit the Vagrantfile to the custom config:**
```
Vagrant.configure("2") do |config|
  config.vm.define "node1" do |node1_config|
    node1_config.vm.box = "ubuntu/jammy64"
    node1_config.vm.hostname = "node1"
    node1_config.vm.provision "ansible", playbook: "ubuntu.yml"
    node1_config.vm.network "private_network", type: "dhcp"
    node1_config.vm.provider "virtualbox" do |vb|
      vb.memory = "2048"
    end
  end
end
```
*config.vm.define "node1" do |node1_config| = define the VM name if you want create another configs in the same Vagrantfile*

*node1_config.vm.box = "ubuntu/jammy64" = the OS package we choose*

*node1_config.vm.hostname = "node1" = name of VM*

*node1_config.vm.provision "ansible", playbook: "ubuntu.yml" = the line means the node1 will be provisioned by Ansible, and then the playbook that will be used*

*node1_config.vm.network "private_network", type: "dhcp" = this line assigns a private network configured to use DHCP (Dynamic Host Configuration Protocol) and assign an IP address automatically*

*node1_config.vm.provider "virtualbox" do |vb| = the provider will be used*

*vb.memory = "2048" = the RAM size of VM*

**7. After this we will create a playbook for the ansible in the same directory:**
```
---
- name: test
  hosts: all
  gather_facts: true
  become: true
  handlers:
    - name: restart_sshd
      service:
        name: sshd
        state: restarted
  tasks:
    - name: atualizar o cache de pacotes
      apt:
        update_cache: yes
    - name: install
      package:
        state: latest
        name:
          - bash-completion
          - vim
          - nano
          - nginx
          - curl
          - htop
    - name: Run Grafana installation script
      script: /home/jcz/ubuntu_jammy/grafana.sh
    - name: Create jcz1 user
      user:
        name: jcz1
        state: present
        password: "{{ 'Password1' | password_hash('sha512') }}"
        update_password: on_create
        shell: /bin/bash
    - name: Edit SSHD Config
      lineinfile:
        path: /etc/ssh/sshd_config
        regexp: '^PasswordAuthentication '
        insertafter: '#PasswordAuthentication'
        line: 'PasswordAuthentication yes'
      notify: restart_sshd
    - name: Add sudo rights for jcz1
      copy:
        dest: /etc/sudoers.d/jcz1
        content: "jcz1 ALL=(root) NOPASSWD: ALL"
        backup: true
```

*this is my playbook*

*in the playbook i need use the script to install grafana-server, therefore i will create the grafana.sh in the same directory of ansible and Vagrantfile:

```
sudo apt-get install -y software-properties-common wget

sudo wget -q -O /usr/share/keyrings/grafana.key https://apt.grafana.com/gpg.key

echo "deb [signed-by=/usr/share/keyrings/grafana.key] https://apt.grafana.com stable main" | sudo tee -a /etc/apt/sources.list.d/grafana.list

echo "deb [signed-by=/usr/share/keyrings/grafana.key] https://apt.grafana.com beta main" | sudo tee -a /etc/apt/sources.list.d/grafana.list

sudo apt update -y

sudo apt install grafana -y
```
**8. So we need get a permission of executable for the grafana.sh:**
```
sudo chmod +x grafana.sh
```
**9. Now we can run:**
```
vagrant up
```
*to destroy VM = vagrant destroy*





