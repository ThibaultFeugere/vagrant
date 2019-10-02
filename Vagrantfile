Vagrant.configure(2) do |config|
  config.vm.box = "debian/buster64"
  config.vm.define "dokuwiki" do |dokuwiki|
    dokuwiki.vm.network "private_network", ip: "192.168.56.10"
    dokuwiki.vm.hostname = "dokuwiki"
  end
  config.vm.define "backup" do |backup|
    backup.vm.box = "debian/buster64"
    backup.vm.network "private_network", ip: "192.168.56.11"
    backup.vm.hostname = "backup"
  end
end
