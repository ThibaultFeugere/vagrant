# TPs

## Dokuwiki

### Points positifs

Vagrant est super cool, comme tu peux le voir sur les commits j'ai très vite réussi à mettre en place deux VM's avec Dokuwiki accessible sur les deux VM's.

L'édition de la crontab ne fut pas très compliqué à part l'histoire de ```home/root``` et ```home/vagrant```.

### Point négatif

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

Update : beaucoup plus clair grâce à ta correction, j'ai compris que tout le dossier est synchronisé avec Vagrant et ça fonctionne :D

## Tp-https

On exporte la configuration de apache2 qui nous intéresse. On continue l'installation de Dokuwiki et on restart les services.

On pense bien à créer un index.html avec pour contenu `site inconnu`.

## Td-dns

### Connectez vous au host "proxy" avec vagrant et vérifier comment est configuré le resolver dns du système ?

Pour se connecter au host "proxy" : `vagrant ssh proxy`

Pour vérifier comment est configuré le resolver dns du système, j'effectue la commande `cat /etc/resolver.conf`.

Résultat :

```shell script
vagrant@proxy:~$ cat /etc/resolv.conf 
domain auvence.co
search auvence.co
nameserver 192.168.33.21
nameserver 192.168.33.22
nameserver 10.0.2.3
```

Ce fichier sert à indiquer quels sont les serveurs DNS ainsi que le domaine par défaut qui est auvence.co, et il cherche dans auvence.co.

Le premier nameserver sera interrogé (`192.168.33.21`), puis s'il échoue, le deuxième nameserver sera interrogé (`192.168.33.22`) et sinon le troisième (`10.0.2.3`).

Remarque : on peut mettre jusqu'à trois nameserver, ce qui est le cas ici.

Les deux premiers nameservers inscrits semblent être les deux serveurs recursifs.

Preuve, `ip a` sur recursor-1 :

```shell script
4: eth2: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc pfifo_fast state UP group default qlen 1000
    link/ether 08:00:27:aa:79:23 brd ff:ff:ff:ff:ff:ff
    inet 192.168.33.21/24 brd 192.168.33.255 scope global noprefixroute eth2
       valid_lft forever preferred_lft forever
    inet6 fe80::a00:27ff:feaa:7923/64 scope link 
       valid_lft forever preferred_lft forever
```

Preuve, `ip a` sur recursor-2 :

```shell script
4: eth2: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc pfifo_fast state UP group default qlen 1000
    link/ether 08:00:27:eb:a9:ef brd ff:ff:ff:ff:ff:ff
    inet 192.168.33.22/24 brd 192.168.33.255 scope global noprefixroute eth2
       valid_lft forever preferred_lft forever
    inet6 fe80::a00:27ff:feeb:a9ef/64 scope link 
       valid_lft forever preferred_lft forever
```

### Retrouvez l'adresse ip du host wiki.lab.local avec la commande dig.

Pour retrouver l'adresse ip du host `wiki.lab.local` avec la commande dig, il y a plusieurs variantes de cette commande aboutissant au même résultat :

```shell script
vagrant@proxy:~$ dig wiki.lab.local

; <<>> DiG 9.10.3-P4-Debian <<>> wiki.lab.local
;; global options: +cmd
;; Got answer:
;; ->>HEADER<<- opcode: QUERY, status: NOERROR, id: 11250
;; flags: qr rd ra; QUERY: 1, ANSWER: 1, AUTHORITY: 0, ADDITIONAL: 1

;; OPT PSEUDOSECTION:
; EDNS: version: 0, flags:; udp: 4096
;; QUESTION SECTION:
;wiki.lab.local.			IN	A

;; ANSWER SECTION:
wiki.lab.local.		3600	IN	A	192.168.56.11

;; Query time: 6 msec
;; SERVER: 192.168.33.21#53(192.168.33.21)
;; WHEN: Fri Oct 18 14:21:30 GMT 2019
;; MSG SIZE  rcvd: 59
```

Ou avec le @ avec comme paramétre suivant le dns que l'on souhaite interroger manuellement.

```shell script
vagrant@proxy:~$ dig wiki.lab.local@192.168.33.21

; <<>> DiG 9.10.3-P4-Debian <<>> wiki.lab.local@192.168.33.21
;; global options: +cmd
;; Got answer:
;; ->>HEADER<<- opcode: QUERY, status: NXDOMAIN, id: 33657
;; flags: qr rd ra; QUERY: 1, ANSWER: 0, AUTHORITY: 1, ADDITIONAL: 1

;; OPT PSEUDOSECTION:
; EDNS: version: 0, flags:; udp: 4096
;; QUESTION SECTION:
;wiki.lab.local\@192.168.33.21.	IN	A

;; AUTHORITY SECTION:
.			3600	IN	SOA	a.root-servers.net. nstld.verisign-grs.com. 2019101800 1800 900 604800 86400

;; Query time: 21 msec
;; SERVER: 192.168.33.21#53(192.168.33.21)
;; WHEN: Fri Oct 18 14:22:03 GMT 2019
;; MSG SIZE  rcvd: 132
```

On peut voir que le champ A, c'est à dire la liaison entre le nom de domaine est l'adresse ip est : 192.168.56.11 qui est associé à wiki.lab.local

### Connectez vous au host auth-1, quels sont les services réseaux qui sont en fonctionnement actuellement quels sont leur socket d'écoute ?

Connexion au host auth-1 avec la commande `vagrant ssh auth-1`.

Pour voir quels sont les services réseaux qui sont en fonctionnement actuellement et qui écoute on utilise la commande : `sudo ss -lnp`
avec comme paramétre L pour listen, N pour number et P for process name.
 
```shell script                                                                                  :::*                   users:(("rpcbind",pid=1376,fd=11))
tcp   LISTEN     0      50                                                                  127.0.0.1:3306                                                                                    *:*                   users:(("mysqld",pid=5958,fd=14))
tcp   LISTEN     0      128                                                                         *:111                                                                                     *:*                   users:(("rpcbind",pid=1376,fd=4),("systemd",pid=1,fd=64))
tcp   LISTEN     0      128                                                             192.168.33.31:80                                                                                      *:*                   users:(("httpd",pid=6015,fd=3),("httpd",pid=6014,fd=3),("httpd",pid=6013,fd=3),("httpd",pid=6012,fd=3),("httpd",pid=6011,fd=3),("httpd",pid=6009,fd=3))
tcp   LISTEN     0      128                                                                         *:53                                                                                      *:*                   users:(("pdns_server",pid=6030,fd=7))
tcp   LISTEN     0      128                                                                         *:22                                                                                      *:*                   users:(("sshd",pid=2380,fd=3))
tcp   LISTEN     0      100                                                                 127.0.0.1:25                                                                                      *:*                   users:(("master",pid=2607,fd=13))
tcp   LISTEN     0      128                                                                        :::111                                                                                    :::*                   users:(("rpcbind",pid=1376,fd=6),("systemd",pid=1,fd=66))
tcp   LISTEN     0      128                                                                        :::53                                                                                     :::*                   users:(("pdns_server",pid=6030,fd=8))
tcp   LISTEN     0      128                                                                        :::22                                                                                     :::*                   users:(("sshd",pid=2380,fd=4))
tcp   LISTEN     0      100                                                                       ::1:25                                                                                     :::*                   users:(("master",pid=2607,fd=14))
```

- On peut voir qu'il y a le port 53 qui est le port DNS
- Le port 25 qui est le SMTP (Simple Mail Transfer Protocol)
- Le port 3306 est le port de mysql
- Le port 80 est le port du protocole HTTP
- Le port 111 est le port SUN Remote Procedure Call
- Le port 22 est le port ssh (auquel nous sommes connecté)

### Connectez vous au host recursor-1,  quels sont les services réseaux qui sont en fonctionnement actuellement quels sont leur socket d'écoute ?

```shell script
tcp   LISTEN     0      128                                                                         *:111                                                                                     *:*                   users:(("rpcbind",pid=1270,fd=4),("systemd",pid=1,fd=66))
tcp   LISTEN     0      128                                                             192.168.33.21:53                                                                                      *:*                   users:(("pdns_recursor",pid=5797,fd=7))
tcp   LISTEN     0      128                                                                 127.0.0.1:53                                                                                      *:*                   users:(("pdns_recursor",pid=5797,fd=6))
tcp   LISTEN     0      128                                                                         *:22                                                                                      *:*                   users:(("sshd",pid=2377,fd=3))
tcp   LISTEN     0      100                                                                 127.0.0.1:25                                                                                      *:*                   users:(("master",pid=2589,fd=13))
tcp   LISTEN     0      128                                                                        :::111                                                                                    :::*                   users:(("rpcbind",pid=1270,fd=6),("systemd",pid=1,fd=68))
tcp   LISTEN     0      128                                                                        :::22                                                                                     :::*                   users:(("sshd",pid=2377,fd=4))
tcp   LISTEN     0      100                                                                       ::1:25                                                                                     :::*  
```

- On peut voir qu'il y a le port 53 qui est le port DNS
- Le port 25 qui est le SMTP (Simple Mail Transfer Protocol)
- Le port 80 est le port du protocole HTTP
- Le port 111 est le port SUN Remote Procedure Call
- Le port 22 est le port ssh (auquel nous sommes connecté)

Le port mysql n'est pas présent ce qui est normal.

### Où sont configuré chacuns de ces composants ?

- Le composant Apache est configuré dans `cd /etc/apache2/`
- Le composant MySQL est configuré dans `cd /etc/mysql/`
- Le composant SMTP est configuré dans ``
- Le composant SSH est configuré dans `cd /etc/ssh`
- Le composant DNS est configuré dans ``
- Le composant HTTP est configuré dans 

### Qu'est ce qui est configuré sur les serveurs recursifs pour le domaine lab.local ? (Important pour les Actions qui suivent)

Si nous faisons un `cat /etc/pdns-recursor/recursor.conf` et que nous recherchons la ligne qui concerne lab.local on peut trouver :

`forward-zones=lab.local=192.168.56.31;192.168.56.32`

Toutes les demandes et les requêtes effectuées sur la machine (lab local) seront envoyées a 192.168.56.31 et 192.168.56.32

### Comment mettre en évidence le fait que le récurseurs ne répondent que sur l’interface du réseaux back (192.168.33.0/24) ?

Pas trouvé

### Comment est sécurisé l’accès à mysql ?

Pas trouvé