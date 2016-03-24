# -*- mode: ruby -*-
# vi: set ft=ruby :

############### Definitions Comes Here #################
CLOUDSTACK_MGMT_IP = "192.168.151.2"
CLOUDSTACK_MGMT_HOSTNAME = "csmgmtv1"
CLOUDSTACK_AGENT_IP = "192.168.151.3"
CLOUDSTACK_AGENT_HOSTNAME = "csagentv1"
CLOUDSTACK_NETWORK_PREFIX = "192.168.151"
CLOUDSTACK_GATEWAY_IP = "192.168.151.1"
POD_START_IP = "192.168.151.70"
POD_END_IP = "192.168.151.90"
PUBLIC_START_IP = "192.168.151.91"
PUBLIC_END_IP = "192.168.151.121"

############### Write a inventory file, used by Ansible #####
############### ./Ansible/vagrant will be used in provision ##### 
File.open('./Ansible/vagrant' ,'w') do |f|
  f.write "[#{CLOUDSTACK_MGMT_HOSTNAME}]\n"
  f.write "#{CLOUDSTACK_MGMT_IP} ansible_ssh_user=vagrant ansible_ssh_private_key_file=~/.vagrant.d/insecure_private_key\n\n"
  f.write "[#{CLOUDSTACK_AGENT_HOSTNAME}]\n"
  f.write "#{CLOUDSTACK_AGENT_IP} ansible_ssh_user=vagrant ansible_ssh_private_key_file=~/.vagrant.d/insecure_private_key\n\n"
  f.write "[#{CLOUDSTACK_MGMT_HOSTNAME}:vars]\n"
  f.write "ansible_hostname=#{CLOUDSTACK_MGMT_HOSTNAME}\n"
  f.write "ansible_fqdn=#{CLOUDSTACK_MGMT_HOSTNAME}\n"
  f.write "MySQLRoot=root\n"
  f.write "MySQLMaxConnections=700\n"
  f.write "MySQLPass=engine123\n"
  f.write "CloudDBUser=cloud\n"
  f.write "CloudDBPass=engine123\n"
  f.write "ManagementIP=#{CLOUDSTACK_MGMT_IP}\n"
  f.write "AgentIP=#{CLOUDSTACK_AGENT_IP}\n"
  f.write "NetworkPrefix=#{CLOUDSTACK_NETWORK_PREFIX}\n"
  f.write "GatewayIP=#{CLOUDSTACK_GATEWAY_IP}\n"
  f.write "PodStartIP=#{POD_START_IP}\n"
  f.write "PodEndIP=#{POD_END_IP}\n"
  f.write "PublicStartIP=#{PUBLIC_START_IP}\n"
  f.write "PublicEndIP=#{PUBLIC_END_IP}\n"
  f.write "SysTemplateURLurl46=http://192.168.1.69/systemvm64template-4.6.0-xen.vhd.bz2\n\n"
  f.write "VhdutilURL=http://192.168.1.69/vhd-util\n\n"
  f.write "[#{CLOUDSTACK_AGENT_HOSTNAME}:vars]\n"
  f.write "AgentIP=#{CLOUDSTACK_AGENT_IP}\n"
  f.write "GatewayIP=#{CLOUDSTACK_GATEWAY_IP}\n"
end

############### Vagrant Configuration Here #############
Vagrant.configure(2) do |config|

  # vagrant issues #1673..fixes hang with configure_networks
  config.ssh.shell = "bash -c 'BASH_ENV=/etc/profile exec bash'"
  config.ssh.username = 'vagrant'
  config.ssh.password = 'vagrant'
  config.ssh.insert_key = 'true'
  config.vm.provider :libvirt do |domain|
    domain.nic_model_type = 'e1000'
    domain.memory = 384
    domain.nested = true
    domain.cpu_mode = 'host-passthrough'
  end

  # csmgmtv1 node. 
  # Add one networking, modify hostname, define memory, CPU cores.
  config.vm.define :csmgmtv1 do |csmgmtv1|
    csmgmtv1.vm.box = "centos67rootpasswd"
    csmgmtv1.vm.hostname = CLOUDSTACK_MGMT_HOSTNAME
    csmgmtv1.vm.network :private_network, :ip => CLOUDSTACK_MGMT_IP
    csmgmtv1.vm.provider :libvirt do |domain|
      domain.memory = 4096
      domain.cpus = 2
      domain.nested = true
      domain.disk_bus = 'virtio'
      domain.nic_model_type = 'virtio'
      domain.volume_cache = 'writeback'
    end
  end

  # csagentv1 node.
  # Add one networking, modify hostname, define memory, CPU cores.
  config.vm.define :csagentv1 do |csagentv1|
    csagentv1.vm.box = "Xen62Patched"
    csagentv1.vm.hostname = CLOUDSTACK_AGENT_HOSTNAME
    csagentv1.vm.network :private_network, :ip => CLOUDSTACK_AGENT_IP, :mac => "08:00:27:B2:9D:5C"
    # Disable mounting of vagrant folder as it's not supported on xenserver
    csagentv1.vm.synced_folder ".", "/vagrant", disabled: true
    csagentv1.vm.provider :libvirt do |domain|
      domain.memory = 8192
      domain.cpus = 4
      domain.nested = true
      domain.cpu_mode = 'host-passthrough'
      domain.nic_model_type = 'e1000'
      domain.management_network_mac = "08:00:27:F7:38:0B"
    end
  end

  # Run Ansible against csmgmtv1, csmgmtv1 will be ready for deploying CloudStack.
  config.vm.define "csmgmtv1" do |csmgmtv1|
    csmgmtv1.vm.provision :ansible do |ansible|
      ansible.playbook = "./Ansible/provision.yml"
    end
  end

  # cloudstackmgmt.yml should only provisioned on CLOUDSTACK_MGMT_HOSTNAME
  config.vm.define "csmgmtv1" do |csmgmtv1|
    csmgmtv1.vm.provision :ansible do |ansible|
      ansible.playbook = "./Ansible/cloudstackmgmt.yml"
      ansible.limit = CLOUDSTACK_MGMT_HOSTNAME
      ansible.inventory_path = "./Ansible/vagrant"
    end
  end

  ### # Before provision cloudstackagent packages, change cloudbr0 bridged eth1
  ### config.vm.define "csagentv1" do |csagentv1|
  ###   csagentv1.vm.provision :ansible do |ansible|
  ###     ansible.playbook = "./Ansible/agentcloudbr0.yml"
  ###     ansible.limit = CLOUDSTACK_AGENT_HOSTNAME
  ###     ansible.inventory_path = "./Ansible/vagrant"
  ###   end
  ### end

  ### # cloudstackagent.yml should only provisioned on CLOUDSTACK_AGENT_HOSTNAME
  ### config.vm.define "csagentv1" do |csagentv1|
  ###   csagentv1.vm.provision :ansible do |ansible|
  ###     ansible.playbook = "./Ansible/cloudstackagent.yml"
  ###     ansible.limit = CLOUDSTACK_AGENT_HOSTNAME
  ###     ansible.inventory_path = "./Ansible/vagrant"
  ###   end
  ### end

end
