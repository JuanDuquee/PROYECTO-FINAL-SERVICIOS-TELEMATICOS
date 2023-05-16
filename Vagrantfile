# -*- mode: ruby -*-
# vi: set ft=ruby :

Vagrant.configure("2") do |config|

  config.vm.define :mysqlrouter do |mysqlrouter|
    mysqlrouter.vm.box = "generic/centos8"
    mysqlrouter.vm.network :private_network, ip: "192.168.50.2"
#   mysqlrouter.vm.provision "shell", path: "mysqlrouter.sh.txt"
    mysqlrouter.vm.hostname = "mysqlrouter"
  end
  config.vm.define :mysql1 do |mysql1|
    mysql1.vm.box = "generic/centos8"
    mysql1.vm.network :private_network, ip: "192.168.50.3"
#   mysql1.vm.provision "shell", path: "mysql.sh.txt"
    mysql1.vm.hostname = "mysql1"
  end
   config.vm.define :mysql2 do |mysql2|
     mysql2.vm.box = "generic/centos8"
     mysql2.vm.network :private_network, ip: "192.168.50.4"
#    mysql2.vm.provision "shell", path: "mysql.sh.txt"
     mysql2.vm.hostname = "mysql2"
   end
   config.vm.define :mysql3 do |mysql3|
     mysql3.vm.box = "generic/centos8"
     mysql3.vm.network :private_network, ip: "192.168.50.5"
#    mysql3.vm.provision "shell", path: "mysql.sh.txt"
     mysql3.vm.hostname = "mysql3"
   end
   config.vm.define :cliente do |cliente|
     cliente.vm.box = "generic/centos8"
     cliente.vm.network :private_network, ip: "192.168.50.6"
#    cliente.vm.provision "shell", path: "docker.sh.txt"
     cliente.vm.hostname = "cliente"
   end
end

