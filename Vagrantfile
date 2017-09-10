VAGRANTFILE_API_VERSION = "2"
BOX = "ubuntu/trusty64"

base_dir = File.expand_path(File.dirname(__FILE__))

NUM_BOXES = 1

# The hypervisors bridge interface
BRIDGE_IFACE = "en0: Wi-Fi (AirPort)"

BOX_NAME = "vpn"
BOX_DOMAIN_NAME = "vagrant"

BOX_MEMORY = 2048
BOX_CPU = 1
NUM_BOX_DISKS = 0

def create_vm(config, name, index, domain, memory, cpu, numdisks, disksize, bridge, provision)

    ansible_provision = proc do |ansible|
      ansible.playbook = 'site.yml'
      ansible.groups = {
        'vpn_hosts'   => (1..NUM_BOXES).map { |j| "#{BOX_NAME}#{j}.#{BOX_DOMAIN_NAME}" },
        'all:children'   => [
          "vpn_hosts"
        ],
      }
      ansible.limit = 'all'
    end

    config.vm.define "#{name}#{index}.#{domain}" do |de|

        de.vm.hostname = "#{name}#{index}.#{domain}"
        de.vm.network :public_network, bridge: "#{bridge}"

        de.vm.provider :virtualbox do |vb|

            vb.customize ['modifyvm', :id, '--memory', "#{memory}"]
            vb.customize ['modifyvm', :id, '--cpus', "#{cpu}"]

            if numdisks > 0
                (1..numdisks).each do |d|
                    vb.customize ['createhd',
                      '--filename', "#{name}#{index}-#{d}",
                      '--size', "#{disksize}"] unless File.exist?("#{name}#{index}-#{d}.vdi")

                    vb.customize ['storageattach', :id,
                      '--storagectl', "SATAController",
                      '--port', 3 + d,
                      '--device', 0,
                      '--type', 'hdd',
                      '--medium', "#{name}#{index}-#{d}.vdi"]
                end
            end

        end

        de.vm.provision 'ansible', &ansible_provision if provision
    end
end

Vagrant.configure(VAGRANTFILE_API_VERSION) do |config|

  config.ssh.insert_key = false
  config.vm.box = BOX

  (1..NUM_BOXES).each do |i|
    last = (i == NUM_BOXES) ? true : false
    create_vm(config, BOX_NAME, i, BOX_DOMAIN_NAME, BOX_MEMORY, BOX_CPU, NUM_BOX_DISKS, 10000, BRIDGE_IFACE, last)
  end

end
