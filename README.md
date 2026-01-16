# Examen Administration système 16 janvier 2026 - Noé Brossard

Attendu : une url menant à un fichier README.md (format _markdown_) sur une plateforme GIT publique _(codeberg, gitlab, github, ...)_.
reprenant ce document en le complétant.

## Les deux familles

Quelles sont les principales distributions de la famille Debian, de la famille Red Hat, et les autres ?

On a la famille Débian= Debian, Ubuntu, Linux Mint Et pour la famille Red Hat= RHEL, Fedora, CentOS Et les autres Autres= Arch, SUSE openSUSE

Comment pourriez-vous décrire, succintement, leurs positionnements respectifs ? Les différences au niveau des outils d'admnistration ?

Debian= stabilité et serveurs | Red Hat= support commercial et ceux des entreprises | Autres= flexibilité et cas spécifiques. Outils= `apt` (Debian), `yum` / `dnf` (Red Hat)

## Back to basics 

Quelle est la différence fondamentale entre du code qui tourne en mode kernel/superviseur et en mode utilisateur ?

Mode kernel= accès sans restriction au matériel et ressources  du système. Mode utilisateur=f accès restreint pour protéger le système et les autres processus.

## Qui est où ?

Indiquez, en quelques mots, pour ces répertoires, ce qu'ils sont supposés contenir :

- /etc = fichiers de configuration système
- /bin et /usr/bin = ^programmes exécutables essentiels et non essentiels
- /sbin et /usr/sbin = programmes d'administration système
- /home = répertoires personnels des utilisateurs
- /var = données variables comme les logs et les temporaires
- /var/log = fichiers de logs système
- /var/lib = données persistantes des applications

## Où est le pilote ?

Comment retrouver, sous GNU/Linux :

- La liste des pilotes chargées en mémoire ? `lsmod`
- La liste des fichiers binaires _(modules)_ qui les fournissent ? `modinfo <module>`
- Forcer leur chargement et leur déchargement ? `modprobe <module>` et le `rmmod <module>`
- Les messages qu'ils ont émis ? `dmesg | grep <module>`

## MS Windows

Comment fonctionne en pratique WSL _(Windows Subsystem for Linux)_ ? Que permet-il de faire ? Quels protocoles utilise-t-il ?

LE WSL exécute le noyyau Linux léger sur Windows et il permet de réaliser l'exécution d'applications Linux qui sont natives. Sans oublier qu'il utilise Hyper V

## On commence

Vous venez d'installer un système GNU/Linux que faites-vous pour en sécuriser les accès via SSH ?

On doit changer le port SSH quii est le 22 et désactiver l'accès root, utiliser des key SSH, configurer les firewall. Sans oublier de fetch le system.

## Alerte !

Vous vous connectez à un système GNU/Linux, le client ssh vous averti qu'il a un problème de reconnaissance de sa clef publique
d'hôte. Que faites-vous ? Pourquoi ?

Vérifier si l'hôte du SSH est le bon. Tout en mettant à jour le `~/.ssh/known_hosts` et l'hôte dela clé est légitime.

## Oh les gourmands !

Comment identifier rapidement, avec une seule ligne de commande, les utilisateurs qui consomment le plus d'espace de stockage.

_N.B._ Vous pouvez supposer que les répertoires personnels sont tous des sous-répertoires de `/home` 

`du -sh /home/* | sort -hr | head -10`

## On va aider les devs...

Une équipe de développeur.se.s vous demande de préparer un système GNU/Linux pour du développent système (C, make, etc.)

Que faites-vous en pratique en tant qu'administrateur.trice pour commencer ?

- Sur une distribution de la famille Debian : `apt install build-essential gcc g++ make git`
- Sur une distribution de la famille Red Hat : `dnf groupinstall "Development Tools" && dnf install gcc gcc-c++ make git`

## Qui fait quoi ?

Comment pourriez-vous retrouver la liste des dernières connexions d'utilisateurs à un système GNU/Linux, autant à distance que localement ?

Déjà `last` pour les connexions local `lastb` pour les tentatives qui n'ont pas réussi et pour log SSH dans `/var/log/auth.log` ou `/var/log/secure`

Comment identifier les sessions où un.e utilisateur.trice à utilisé _sudo_ pour avoir les droit d'administration ?

`grep sudo /var/log/auth.log` ou voir soimplement `/var/log/secure` et chercher les entrées du sudo

## Post mortem

Le système que vous administrez est tombé cette nuit. Comment récupérer des informations sur ce qui s'est passé ?

On doit regardé alors le `/var/log/syslog`, `/var/log/messages`.

## On surveille...

Sur un serveur GNU/Linux qui démarre via _systemd_, un service nommé _bidule_, comment :

- Savoir si le service est lancé au démarrage automatiquement =`systemctl is-enabled bidule`
- Savoir s'il est en cours de fonctionnement =`systemctl is-active bidule`
- Faire en sorte qu'il le soit =`systemctl enable bidule`
- Examiner les messages qu'il émet =`journalctl -u bidule`
- Le redémarrer =`systemctl restart bidule`
- Déterminer de quel paquet il provient = `dpkg -S /usr/lib/systemd/system/bidule.service`

## Zut

Vous devez reprendre la main, en tant qu'administrateur, d'un système GNU/Linux mais vous n'avez aucun mot de passe à votre disposition, ni _root_ ni utilisateur. Que pouvez-vous faire ?

Au redémarrage on doitalors appuyer sur E pour éditer GRUB et add le `init=/bin/bash` puis réadd le mot de passe root `passwd`

## C'est la faute à Rémy

Un service réseau http(s) ne fonctionne plus. Vous avez accès à ce système comme administrateur.trice. Quels outils utilisez-vous pour diagnostiquer le problème, indiquez précisément ce que chaque outil permet de vérifier.

-> `systemctl status <service>`= état du service | `journalctl -u <service>`= logs | `netstat -tuln | grep :80` | `curl localhost`= connectivité | `/var/log/apache2/error.log`

## Au charbon !

On vous demande d'installer un serveur de base de données _Elastic_ (autrefois nommé _Elastic Search_) sur un système Debian (ou autre si vous préférez)

Décrivez votre recherche documentaire, la méthode que vous sélectionnez, les commandes que vous exécutez et les fichiers de configuration impliqués.

1. consulter la doc officielle elastic.co
2. Ajouter le depot= `apt-key adv --keyserver ... && echo "deb ..." | tee /etc/apt/sources.list.d/elastic.list`
3. `apt update && apt install elasticsearch`
4. config rapide dans le `/etc/elasticsearch/elasticsearch.yml`
5. `systemctl enable --now elasticsearch`

## Le Web

Vous avez la charge de gérer un serveur Web sous Debian GNU/Linux, pour héberger plusieurs sites Web (hébergement mutualisé).

- Quels paquets installez-vous ? `apache2` | `php` | `mysql-server` | `libapache2-mod-php`
- Comment organisez sous le stockage des différents contenus ? `/var/www/site1` | `/var/www/site2`
- Certains sites ont besoin de PHP, comment l'installez-vous ? `apt install php libapache2-mod-php php-mysql`
- Comment pouvez-vous organiser l'enregistrement séparé des visites des différents sites ? VirtualHost= `CustomLog` et `ErrorLog` et séparer pour les diff sites
- Quel(s) outil(s) peuvent vous permettre de fournir une vison des visites de chaque site à chaque Webmaster ? `awstats` | `webalizer`

# Noé Brossard - B2 2026