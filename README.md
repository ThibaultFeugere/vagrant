# Dokuwiki

## Points positifs

Vagrant est super cool, comme tu peux le voir sur les commits j'ai très vite réussi à mettre en place deux VM's avec Dokuwiki accessible sur les deux VM's.

L'édition de la crontab ne fut pas très compliqué à part l'histoire de ```home/root``` et ```home/vagrant```.

## Point négatif

J'ai eu de grosses difficultés avec l'erreur ```permission denied (Publickey)```.

Je suis persuadé que ma commande rsync ajoutée dans le crontab est bonne mais les deux VM's n'ont pas la même clé RSA. Je n'ai pas réussi à fix ce problème.

Je pense que la génération de clé en locale (rsa et rsa.pub peut être une bonne piste) avec ces commandes peuvent peut-être résoudre ce problème :

```ruby
config.ssh.insert_key = false
config.ssh.private_key_path = ["rsa", "~/.vagrant.d/insecure_private_key"]
config.vm.provision "file", source: "rsa", destination: "/home/vagrant/.ssh/id_rsa"
config.vm.provision "file", source: "rsa.pub", destination: "/home/vagrant/.ssh/authorized_keys"
config.vm.provision "file", source: "rsa.pub", destination: "/home/vagrant/.ssh/id_rsa.pub"
```