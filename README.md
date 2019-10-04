# Dokuwiki

## Points positifs

Vagrant est super cool, comme tu peux le voir sur les commits j'ai très vite réussi à mettre en place deux VM's avec Dokuwiki accessible sur les deux VM's.

L'édition de la crontab ne fut pas très compliqué à part l'histoire de ```home/root``` et ```home/vagrant```.

## Point négatif

J'ai eu de grosses difficultés avec l'erreur ```permission denied (Publickey)```.

Je suis persuadé que ma commande rsync ajoutée dans le crontab est bonne mais les deux VM's n'ont pas la même clé RSA. Je n'ai pas réussi à fix ce problème.