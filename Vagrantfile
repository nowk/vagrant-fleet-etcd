# -*-  mode: ruby -*-
# vim: set filetype=ruby :
Vagrant.require_version ">= 1.6.0"
VAGRANTFILE_API_VERSION = 2

COREOS_VAGRANT_JSON = "http://%s.release.core-os.net/amd64-usr/current/coreos_production_vagrant.json"

require 'fileutils'

CLOUD_CONFIG_ETCD = File.join(File.dirname(__FILE__), "user-data.etcd")
CLOUD_CONFIG_FLEET = File.join(File.dirname(__FILE__), "user-data.fleet")

# server struct
Server = Struct.new(:name, :cpus, :memory, :ip, :is_etcd) do
  def is_etcd?
    is_etcd
  end

  def is_fleet?
    !is_etcd?
  end

  def box
    "coreos-%s" % update_channel
  end

  def update_channel
    "stable"
  end

  def cloud_config
    if is_fleet?
      CLOUD_CONFIG_FLEET
    else
      CLOUD_CONFIG_ETCD
    end
  end
end

servers = [
  Server.new("etcd-00", 1, 512, "192.168.10.100", true),
  Server.new("etcd-01", 1, 512, "192.168.10.101", true),
  Server.new("etcd-02", 1, 512, "192.168.10.102", true),

  Server.new("fleet-00", 1, 1024, "192.168.10.110", false),
  Server.new("fleet-01", 1, 1024, "192.168.10.111", false),
  Server.new("fleet-02", 1, 1024, "192.168.10.112", false),
  Server.new("fleet-03", 1, 1024, "192.168.10.113", false),
]


Vagrant.configure(VAGRANTFILE_API_VERSION) do |c|
  c.vm.boot_timeout = 60

  if Vagrant.has_plugin?("vagrant-vbguest") then
    c.vbguest.auto_update = false
  end

  servers.each do |s|
    c.vm.define s.name do |n|
      n.vm.box_check_update = false

      n.vm.box = s.box
      n.vm.box_url = COREOS_VAGRANT_JSON % s.update_channel

      n.vm.hostname = s.name
      n.vm.network "private_network", ip: s.ip

      n.vm.provider :virtualbox do |vb|
        vb.gui = false
        vb.cpus = s.cpus
        vb.memory = s.memory

        # 100% cpu fix
        vb.customize "pre-boot", ['modifyvm', :id, '--nestedpaging', 'off']

        # On VirtualBox, we don't have guest additions or a functional vboxsf
        # in CoreOS, so tell Vagrant that so it can be smarter.
        vb.check_guest_additions = false
        vb.functional_vboxsf = false
      end

      # provision the servers with their respective cloud-configs
      n.vm.provision :file,
        source: "#{s.cloud_config}", destination: "/tmp/vagrantfile-user-data"

      n.vm.provision :shell,
        inline: "mv /tmp/vagrantfile-user-data /var/lib/coreos-vagrant/",
        privileged: true
    end
  end
end
