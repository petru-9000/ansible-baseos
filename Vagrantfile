require 'yaml'

ip = '192.168.56.201'
vmname = 'demo'

# initialize ansible inventory
inventory = {
  'all' => {
    'vars' => {
      'ansible_host_key_checking' => false,
      'ansible_ssh_timeout' => 60
    },
    'hosts' => {
      vmname + '.test.local' => {
        'ansible_host' => ip,
        'ansible_user' => 'vagrant',
        'ansible_ssh_private_key_file' => '.vagrant/machines/' + vmname + '/virtualbox/private_key'
      }
    }
  }
}

# create ansible inventory file
File.open('./hosts.yml', 'w') {|f| f.write inventory.to_yaml}

# -----------------------------------------------------------------------------

Vagrant.configure("2") do |config|
  config.vm.synced_folder "~/Downloads", "/files"
  config.vm.define "#{vmname}" do |node|
    node.vm.network 'private_network', ip: "#{ip}"
    config.vm.box = "bento/rockylinux-8"
    node.vm.provider 'virtualbox' do |v|
      v.name = "#{vmname}"
      v.memory = 8192
      v.cpus = 2
      v.linked_clone = true
    end
  end

  config.vm.network "forwarded_port", guest: 9990, host: 9990
  config.vm.network "forwarded_port", guest: 8080, host: 8080

  config.vm.disk :disk, name: "newswap", size: "10GB"

  config.vm.provision :ansible do |ansible|
    ansible.playbook = "test.yml"
    ansible.inventory_path = "./hosts.yml"
    ansible.raw_arguments = ["--vault-password-file", "~/vault-pass.txt"]
    ansible.limit = "all" # defaults to machine name
    ansible.compatibility_mode = "2.0"
  end

  # idempotency check - should end with changed=0
  config.trigger.after [:up, :provision] do |trigger|
    trigger.info = "Run ansible"
    trigger.run = {inline: "ansible-playbook -i hosts.yml --vault-password-file ~/vault-pass.txt test.yml"}
  end
end

