# -*- mode: ruby -*-
# vi: set ft=ruby :
VM_NAME = "vagrant-install-plugins"
VAGRANT_BOX_NAME = "ubuntu/xenial-server-cloudimg-amd64-vagrant"
VAGRANT_BOX_URL= "https://cloud-images.ubuntu.com/xenial/current/xenial-server-cloudimg-amd64-vagrant.box"
VIRTUALBOX_MEMORY = "4096"
VIRTUALBOX_CPU = "4"
VAGRANT_VERSION = "2.0.1"
VAGRANTFILE_API_VERSION = "2"

# set locale
ENV["LC_ALL"] = "en_US.UTF-8"

# ensure vagrant version
Vagrant.require_version ">= #{VAGRANT_VERSION}"


# All Vagrant configuration is done below. The "2" in Vagrant.configure
# configures the configuration version (we support older styles for
# backwards compatibility). Please don't change it unless you know what
# you're doing.
Vagrant.configure(VAGRANTFILE_API_VERSION) do |config|
  # The most common configuration options are documented and commented below.
  # For a complete reference, please see the online documentation at
  # https://docs.vagrantup.com.

  # Every Vagrant development environment requires a box. You can search for
  # boxes at https://vagrantcloud.com/search.
  config.vm.box = VAGRANT_BOX_NAME
  config.vm.box_url = VAGRANT_BOX_URL

  # Disable automatic box update checking. If you disable this, then
  # boxes will only be checked for updates when the user runs
  # `vagrant box outdated`. This is not recommended.
  config.vm.box_check_update = false

  # Create a forwarded port mapping which allows access to a specific port
  # within the machine from a port on the host machine. In the example below,
  # accessing "localhost:8080" will access port 80 on the guest machine.
  # NOTE: This will enable public access to the opened port
  # config.vm.network "forwarded_port", guest: 80, host: 8080

  # Create a forwarded port mapping which allows access to a specific port
  # within the machine from a port on the host machine and only allow access
  # via 127.0.0.1 to disable public access
  # config.vm.network "forwarded_port", guest: 80, host: 8080, host_ip: "127.0.0.1"

  # Create a private network, which allows host-only access to the machine
  # using a specific IP.
  # config.vm.network "private_network", ip: "192.168.33.10"

  # Create a public network, which generally matched to bridged network.
  # Bridged networks make the machine appear as another physical device on
  # your network.
  # config.vm.network "public_network"

  config.vm.network "public_network", :type =>"dhcp" ,:bridge => 'eno1'




  # Share an additional folder to the guest VM. The first argument is
  # the path on the host to the actual folder. The second argument is
  # the path on the guest to mount the folder. And the optional third
  # argument is a set of non-required options.
  # config.vm.synced_folder "../data", "/vagrant_data"

  # Provider-specific configuration so you can fine-tune various
  # backing providers for Vagrant. These expose provider-specific options.
  # Example for VirtualBox:
  #
   config.vm.provider "virtualbox" do |vb|
    vb.name= VM_NAME
    vb.gui = false
    vb.memory = VIRTUALBOX_MEMORY
    vb.cpus = VIRTUALBOX_CPU
   end
  #
  # View the documentation for the provider you are using for more
  # information on available options.


  config.ssh.forward_env = ["TERM"]
  # Enable provisioning with a shell script. Additional provisioners such as
  # Puppet, Chef, Ansible, Salt, and Docker are also available. Please see the
  # documentation for more information about their specific syntax and use.
  config.vm.provision "shell", inline: <<-SHELL
    apt-get update
    apt-get upgrade
    apt-get autoremove
    # apt-get install -y apache2
    # set german locale
    locale-gen de_DE.UTF-8
    # install dependencys
    apt-get install -y ansible python-pip python-dev libffi-dev gcc libssl-dev python-selinux python-setuptools
    # pip install -U pip <= get error https://github.com/pypa/pip/issues/5240
    python -m pip install --upgrade pip

    # oslo-config 6.2.1 has requirement PyYAML>=3.12, but you'll have pyyaml 3.11 which is incompatible
    apt-get instal libpython-dev
    pip install -U ansible
    # save org orginal ansible.cfg
    cat /etc/ansible/ansible.cfg |grep -v '^#.*' |grep -v -e '^[[:space:]]*$' > /etc/ansible/ansible.cfg_onInstall
    # settup kolla ansible.cfg
    # TODO must be improved
    echo "[defaults]" >/etc/ansible/ansible.cfg
    echo "host_key_checking=False" >>/etc/ansible/ansible.cfg
    echo "pipelining=True" >>/etc/ansible/ansible.cfg
    echo "forks=100" >>/etc/ansible/ansible.cfg
    # install kolla-ansible
    pip install kolla-ansible
    # Copy globals.yml and passwords.yml to /etc/kolla directory
    cp -r /usr/local/share/kolla-ansible/etc_examples/kolla /etc/
    cp /usr/local/share/kolla-ansible/ansible/inventory/* /home/vagrant
    # clone repos
    git clone https://github.com/openstack/kolla
    git clone https://github.com/openstack/kolla-ansible
    # Install requirements of kolla and kolla-ansible
    pip install -r kolla/requirements.txt
    pip install -r kolla-ansible/requirements.txt
    # Copy the configuration files to /etc/kolla directory. kolla-ansible holds the configuration files ( globals.yml and passwords.yml) in etc/kolla
    mkdir -p /etc/kolla
    cp -r kolla-ansible/etc/kolla/* /etc/kolla
    # Copy the inventory files to the current directory. kolla-ansible holds inventory files ( all-in-one and multinode) in the ansible/inventory directory.
    cp kolla-ansible/ansible/inventory/*  /home/vagrant

    SHELL
end
