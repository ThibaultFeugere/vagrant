Vagrant.configure(2) do |config|
  config.vm.box = "debian/buster64"
  config.vm.define "wiki" do |dokuwiki|
    dokuwiki.vm.network "private_network", ip: "192.168.56.10"
    dokuwiki.vm.hostname = "wiki"
    dokuwiki.vm.provision "shell", inline: <<-SHELL
      sudo su
      cd /etc
      echo "*/5 * * * * vagrant sudo rsync --partial -e ssh /var/www/html/data root@192.168.56.11:/var/www/html/data" >> crontab
    SHELL
  end
  config.vm.define "backup" do |backup|
    backup.vm.box = "debian/buster64"
    backup.vm.network "private_network", ip: "192.168.56.11"
    backup.vm.hostname = "backup"
  end
  config.vm.provision "shell", inline: <<-SHELL
      ssh-keygen -t rsa -N '' -f /home/vagrant/.ssh/id_rsa
      sudo chmod 700 /home/vagrant/.ssh/id_rsa
      cd .ssh/
      sudo cat id_rsa >> authorized_keys
      cd /etc/ssh/
      sudo echo "PermitRootLogin yes" >> sshd_config
      sudo echo "PubkeyAuthentication yes" >> sshd_config
      sudo echo "PermitEmptyPasswords yes" >> sshd_config
      sudo service ssh restart
      echo "### Installation de php-geshi ###"
      sudo apt install php-geshi -y
      cd /var/www/html
      echo "### Téléchargement de la dernière version de DokuWiki dans le dossier /var/www/html ###"
      sudo wget https://download.dokuwiki.org/out/dokuwiki-63487079d8919ad20087d39beea025a9.tgz
      echo "### Extraction de l'élément téléchargé ###"
      sudo tar -xvf dokuwiki-63487079d8919ad20087d39beea025a9.tgz
      echo "### Suppression de l'archive telecharge precedement ###"
      sudo rm -Rf dokuwiki-63487079d8919ad20087d39beea025a9.tgz
      echo "### Suppression de index.html ###"
      sudo rm index.html
      cd dokuwiki/
      sudo mv * ../
      sudo chown -R www-data /var/www/html/
    SHELL
end