Vagrant.configure(2) do |config|
  config.vm.box = "debian/buster64"
  config.vm.define "wiki" do |dokuwiki|
    dokuwiki.vm.network "private_network", ip: "192.168.56.10"
    dokuwiki.vm.hostname = "wiki"
  end
  config.vm.define "backup" do |backup|
    backup.vm.box = "debian/buster64"
    backup.vm.network "private_network", ip: "192.168.56.11"
    backup.vm.hostname = "backup"
  end
  config.vm.provision "shell", inline: <<-SHELL
      echo "### Installation de php-geshi ###"
      sudo apt install php-geshi -y
      cd /var/www/html
      echo "### Téléchargement de la dernière version de DokuWiki dans le dossier /var/www/html ###"
      sudo wget https://download.dokuwiki.org/out/dokuwiki-63487079d8919ad20087d39beea025a9.tgz
      echo "### Extraction de l'élément téléchargé ###"
      sudo tar -xvf dokuwiki-63487079d8919ad20087d39beea025a9.tgz
      sudo rm dokuwiki-63487079d8919ad20087d39beea025a9.tgz index.html
      cd dokuwiki/
      sudo mv * ../
      sudo chown -R www-data /var/www/html/
    SHELL
end