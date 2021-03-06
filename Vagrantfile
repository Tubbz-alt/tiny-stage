# -*- mode: ruby -*-
# vi: set ft=ruby :
ENV['VAGRANT_NO_PARALLEL'] = 'yes'

domain = "tinystage.test"

machines = {
  "freeipa": {
    "vm.hostname": "ipa.#{domain}",
    "hostmanager.aliases": ["kerberos"],
    "autostart": true,
  },
  "fasjson": {"autostart": true},
  "ipsilon": {"autostart": true},
  "oidctest": {},
  "openidtest": {},
  "fas2ipa": {},
  "noggin": {},
  "elections": {},
  "mirrormanager2": {},
}

Vagrant.configure(2) do |config|
  config.hostmanager.enabled = true
  config.hostmanager.manage_host = true
  config.hostmanager.manage_guest = true

  machines.each do |mname, mdef|
    autostart = mdef.fetch(:autostart, false)
    mdef.delete(:autostart)
    config.vm.define mname, autostart: autostart do |machine|
      machine.vm.box_url = "https://download.fedoraproject.org/pub/fedora/linux/releases/32/Cloud/x86_64/images/Fedora-Cloud-Base-Vagrant-32-1.6.x86_64.vagrant-libvirt.box"
      machine.vm.box = "f32-cloud-libvirt"
      machine.vm.hostname = "#{mname}.#{domain}"

      mdef.each do |prop, value|
        prop = prop.to_s

        if prop == "hostmanager.aliases"
          value = value.map {|n| "#{n}.#{domain}"}.compact
        end

        prop_elems = prop.split(".")
        obj = machine
        prop_elems[..-2].each do |elem_prop|
          obj = obj.send("#{elem_prop}")
        end

        obj.send("#{prop_elems[-1]}=", value)
      end

      machine.vm.synced_folder ".", "/vagrant", type: "sshfs"

      machine.vm.provider :libvirt do |libvirt|
        libvirt.cpus = 2
        libvirt.memory = 2048
      end

      machine.vm.provision "ansible" do |ansible|
        ansible.playbook = "ansible/#{mname}.yml"
        ansible.config_file = "ansible/ansible.cfg"
      end
    end
  end
end
