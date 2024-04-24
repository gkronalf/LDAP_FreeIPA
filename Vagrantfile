# -*- mode: ruby -*-
# vim: set ft=ruby :

# OS = "bento/centos-8.4"
OS = "centos/7"

Vagrant.configure("2") do |config|
   # Указываем ОС, версию, количество ядер и ОЗУ
   config.vm.box = OS
   #config.vm.box_version = "20210210.0"

   config.vm.provider :virtualbox do |v|
     v.memory = 2048
     v.cpus = 1
   end
 
   # Указываем имена хостов и их IP-адреса
   boxes = [
     { :name => "ipa.otus.lan",
       :ip => "192.168.57.10",
     },
     { :name => "client1.otus.lan",
       :ip => "192.168.57.11",
     },
     { :name => "client2.otus.lan",
       :ip => "192.168.57.12",
     }
   ]
   # Цикл запуска виртуальных машин
   boxes.each do |opts|
     config.vm.define opts[:name] do |config|
       config.vm.hostname = opts[:name]
       config.vm.network "private_network", ip: opts[:ip]
      #  if opts[:name] == "client2.otus.lan"
      #   config.vm.provision "ansible" do |ansible|
      #    ansible.playbook = "ansible/playbook.yml"
      #    ansible.inventory_path = "ansible/hosts.ini"
      #    ansible.host_key_checking = "false"
      #    ansible.limit = "all"
      #   end
      #  end
     end
   end
 end
