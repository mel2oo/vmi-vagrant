require 'rubygems'

Vagrant.configure(2) do |config|
    config.vm.box = "generic/debian10"
    config.vm.box_version = "4.1.18"
    config.vm.define "kvmi"

    # configuration
    root_dir = '/vagrant'
    enabled_vms = {
        'winxp': true,
        'win7': false,
        'ubuntu': false,
    }

    config.vm.provider "libvirt" do |libvirt, override|
        libvirt.cpus = 4
        libvirt.memory = 2048
        libvirt.nic_model_type = "virtio"
        libvirt.driver = "kvm"
        libvirt.nested = true

        # expose parent dir
        override.vm.synced_folder ".", root_dir,
            :type => "nfs",
            :nfs_version => 4,
            :nfs_udp => false,
            # speedup NFS with custom options
            :linux__nfs_options => ["rw", "no_subtree_check", "all_squash"],
            # changed default behavior --copy-links to --links, because
            # some symlinks in qemu were pointing to non existent dirs in /opt
            :rsync__args => ["--verbose", "--archive", "--delete", "-z", "--links"]
    end

    config.vm.provider "virtualbox" do |vbox, override|
        vbox.cpus = 4
        vbox.memory = 2048
        vbox.linked_clone = true

        # virtualbox needs specific config for NFS, use vboxsf instead
        override.vm.synced_folder ".", root_dir, SharedFoldersEnableSymlinksCreate: false,
            :type => "virtualbox"
    end

    config.vm.provider "hyperv" do |hyperv, override|
        hyperv.cpus = 4
        hyperv.memory = 2048
        hyperv.enable_virtualization_extensions = true

        # on Windows we will use rsync
        # excluding .git directories is recommended of course, but here we need the kvm/.git directory
        # since we want to do the checkout from inside the VM
        # We cannot do an initial checkout on the Windows host because of filesystem limitations.
        override.vm.synced_folder ".", root_dir,
            :type => "rsync"
    end

    # 使用debian国内源
    config.vm.provision "shell",
        inline: "wget -O /etc/apt/sources.list http://qiniu.xiwen.online/Debian10.list && apt-get update"

    # install ansible
    # set stdout_callback = yaml to improve error output
    # note: problem with \1yaml -> \x01yaml in ansible.cfg
    config.vm.provision "shell",
        inline: "apt-get install -y ansible && sed -i -E 's/^#?(stdout_callback\s*=\s*).*$/stdout_callback = yaml/' /etc/ansible/ansible.cfg"

    # config.vm.provision "ansible_local" do |ansible|
    #     # debug
    #     # ansible.verbose =  '-vvv'
    #     # ansible.start_at_task =  ''
    #     ansible.playbook = "vagrant/ansible/playbook_1.yml"
    #     ansible.extra_vars = {
    #         'root_dir': root_dir,
    #         # to do the KVM git checkout if necessary
    #         'is_windows': Gem.win_platform?
    #     }
    # end

    # config.vm.provision "reload"

    # config.vm.provision "ansible_local" do |ansible|
    #     # debug
    #     # ansible.verbose =  '-vvv'
    #     # ansible.start_at_task =  ''
    #     ansible.playbook = "vagrant/ansible/playbook_2.yml"
    #     ansible.extra_vars = {
    #         'root_dir': root_dir,
    #         'enabled_vms': enabled_vms
    #     }
    # end
end