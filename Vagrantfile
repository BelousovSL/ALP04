# -*- mode: ruby -*-
# vi: set ft=ruby :

ENV['VAGRANT_SERVER_URL'] = 'https://vagrant.elab.pro'
Vagrant.configure("2") do |config|
   config.vm.define "belousovRaid" do |belousovRaid|
      belousovRaid.vm.box = "bento/ubuntu-24.04"      
      belousovRaid.vm.host_name = "belousovRaid"
      belousovRaid.vm.provider "virtualbox" do |vb|
        vb.memory = "1024"
        vb.cpus = "2"
      end

      (1..5).each do |i|
        belousovRaid.vm.disk :disk, size: "1GB", name: "disk-#{i}"
      end

      
      belousovRaid.vm.provision "shell", inline: <<-SHELL
     
      # Создаем Создаем 10-RAID
      mdadm --create --verbose /dev/md10 -l 10 -n 4 /dev/sdb /dev/sdc /dev/sdd /dev/sde
      
      # В Файле /etc/mdadm/mdadm.conf фиксируем значение, так как после перезагрузки они меняются.
      mdadm --detail --scan >> /etc/mdadm/mdadm.conf
      update-initramfs -u
      
      # Эмитируем поломку диска /dev/sdd в Raid.
      mdadm /dev/md10 --fail /dev/sdd

      # Удаляем диск /dev/sdd из Raid.
      mdadm /dev/md10 --remove /dev/sdd

      # Добавляем новый диск /dev/sdf
      mdadm /dev/md10 --add /dev/sdf

      # После наш Raid хорошо себя чувствует. 

      # Для нашего Raid создаем таблицу gpt, и один раздел на 1G.
      (
        echo g
        echo n
        echo   
        echo   
        echo +1G
        echo w 
      ) | fdisk /dev/md10

      # Форматируем новый раздел /dev/md10p1 в EXT4
      mkfs.ext4 /dev/md10p1 
      
      # Добавляем каталог монтирования
      mkdir /mnt/raid10-1  

      # Прописываем в /etc/fstab для автоматического монтирования при запуске
      echo "/dev/md10p1 /mnt/raid10-1 ext4  defaults        1 2" >> /etc/fstab
      
      # Монтируем все что можем
      mount -a

      # Все. Счастье есть, оно не может не быть.

      SHELL
    

   end
end