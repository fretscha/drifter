# -*- mode: ruby -*-
# vi: set ft=ruby :

# Please do not edit this file directly as this is most probably a symbolic
# link to the rawbot-virtualization submodule. You can customize your Vagrant
# box by editing the following files :
#
#   * virtualization/parameters.yml for project related parameters
#   * virtualization/playbook.yml for provisioning option
#   * virtualization/VagrantfileExtra.rb should you need anything else
#
# If those options are not enough for you, please get in touch so we can
# devise something reusable for everyone !

# Vagrantfile API/syntax version. Don't touch unless you know what you're doing!
VAGRANTFILE_API_VERSION = "2"

load 'virtualization/VagrantfileExtra.rb'

custom_config = CustomConfig.new

Vagrant.configure(VAGRANTFILE_API_VERSION) do |config|
    config.vm.box = "jessie64"
    config.vm.hostname = custom_config.hostname

    config.vm.box_url = "http://vagrantbox-public.liip.ch/liip-jessie64.box"

    config.vm.network "forwarded_port", guest: 80, host: 8080, auto_correct: true

    config.ssh.forward_agent = true

    if Vagrant.has_plugin?("vagrant-hostmanager")
        config.hostmanager.enabled = true
        config.hostmanager.manage_host = true
        config.hostmanager.ignore_private_ip = false
        config.hostmanager.include_offline = true
        config.hostmanager.aliases = custom_config.load_aliases
    end

    config.vm.provider "virtualbox" do |v, override|
        override.vm.network :private_network, ip: custom_config.box_ip
        override.vm.synced_folder ".", "/vagrant", type: "nfs", mount_options: ['noatime', 'noacl', 'proto=udp', 'vers=3', 'async']

        v.memory = 4096
        v.cpus = 2
    end

    config.vm.provider "lxc" do |lxc, override|
        override.vm.box = "jessie64-lxc"

        override.vm.box_url = "http://vagrantbox-public.liip.ch/liip-jessie64-lxc.box"

        if Vagrant.has_plugin?("vagrant-hostmanager")
            override.hostmanager.ignore_private_ip = true
        end
    end

    config.vm.provision "ansible" do |ansible|
        ansible.host_key_checking = false
        # Default value for backwards-compatibility
        ansible.playbook = custom_config.respond_to?('playbook') ? custom_config.playbook : "virtualization/playbook.yml"
        ansible.host_key_checking = false
        # Update verbosity as needed, multiples 'v' means more verbose
        # ansible.verbose = 'v'

        # We can't use controlmaster because it makes SSH use the same
        # connection over and over again. This clashes with the base role which
        # changes sshd configuration. If controlmaster is used, the new sshd
        # configuration is not taken into account
        ansible.raw_ssh_args = ['-o ControlMaster=no']
    end
end
