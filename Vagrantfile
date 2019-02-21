home = ENV['HOME']

MACHINES = {
  :tempvm => {
    :box_name => "centos/7",
    :disks => {
      :sata1 => {
        :dfile => home + '/VirtualBox\ VMs/tmp_vagrant/sata1.vdi',
        :size => 1024,
        :port => 2
      },
      :sata2 => {
        :dfile => home + '/VirtualBox\ VMs/tmp_vagrant/sata2.vdi',
        :size => 1024,
        :port => 3
      }
    }
  },
}

Vagrant.configure("2") do |config|
  MACHINES.each do |boxname, boxconfig|

    config.vm.define boxname do |box|

      box.vm.box = boxconfig[:box_name]
      box.vm.host_name = boxname.to_s

      box.vm.network "private_network", type: "dhcp"

      box.vm.provider :virtualbox do |vb|
        vb.customize ["modifyvm", :id, "--memory", "512"]
        needsController = false
        boxconfig[:disks].each do |dname, dconf|
          unless File.exist?(dconf[:dfile])
            vb.customize ['createhd', '--filename', dconf[:dfile], '--variant', 'Fixed', '--size', dconf[:size]]
            needsController = true
          end
        end
        if needsController == true
          vb.customize ["storagectl", :id, "--name", "SATA", "--add", "sata" ]
          boxconfig[:disks].each do |dname, dconf|
            vb.customize ['storageattach', :id, '--storagectl', 'SATA', '--port', dconf[:port], '--device', 0, '--type', 'hdd', '--medium', dconf[:dfile]]
          end
        end
      end
      config.vm.provision "shell", inline: <<-SHELL
        yum install -y vim smartmontools tree mdadm hdparm gdisk lvm2 &&
        mdadm --create --force --metadata=0.90 /dev/md0 -l 1 -n 2 /dev/sd{b,c} &&
        pvcreate /dev/md0 &&
        vgcreate vg0 /dev/md0 && 
        lvcreate -l100%FREE -n lv01 vg0 &&
        mkfs.ext4 /dev/vg0/lv0 &&
        mkdir /new &&
        mount /dev/vg0/lv01 /new
      SHELL
    end
  end
end
