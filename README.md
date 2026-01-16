# Examen Administration système 16 janvier 2026 - Noé Brossard

Attendu : une url menant à un fichier README.md (format _markdown_) sur une plateforme GIT publique _(codeberg, gitlab, github, ...)_.
reprenant ce document en le complétant.

## Les deux familles

Quelles sont les principales distributions de la famille Debian, de la famille Red Hat, et les autres ?

- Famille Debian= Debian, Ubuntu, Linux Mint, Kali Linux, Elementary OS
- Famille Red Hat= RHEL, Fedora, CentOS Stream, Rocky Linu et AlmaLinux
- Autres= Arch Linux, Manjaro, SUSE, openSUSE, Gentoo, Slackware

Comment pourriez-vous décrire, succintement, leurs positionnements respectifs ? Les différences au niveau des outils d'admnistration ?

- Debian= Stabilité maximale, cycles de releases longs, parfait pour serveurs de production. Utilise `apt`/`dpkg` pour la gestion de paquets .deb
- Red Hat= Support commercial LTS, certifications, orienté entreprise. Utilise `yum`/`dnf` pour les paquets .rpm
- arch= Rolling release, dernières versions, configuration manuelle. Utilise `pacman`
- SUSE= Distribution européenne, outils graphiques YaST, orienté entreprise. Utilise `zypper` pour les .rpm

## Back to basics 

Quelle est la différence fondamentale entre du code qui tourne en mode kernel/superviseur et en mode utilisateur ?

- Mode kernel= DANS RING 0 | Accès complet au matériel (CPU, mémoire, périphériques), exécution d'instructions privilégiées, gestion des interruptions. Un crash kernel provoque un kernel panic
- Mode utilisateur= DANS RING 3 | Accès restreint via lkes syscalls, espace mémoire isolé, pas d'accès direct au matériel. Un crash n'affecte que le process concerner et la séparation assure la sécu et la stab du sys

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

- La liste des pilotes chargées en mémoire ?
```bash
lsmod
```
- La liste des fichiers binaires _(modules)_ qui les fournissent ?
```bash
modinfo <module>
```
- Forcer leur chargement et leur déchargement ?
```bash
modprobe <module>
```
et
```bash
rmmod <module>
```
- Les messages qu'ils ont émis ?
```bash
dmesg | grep <module>
```

## MS Windows

Comment fonctionne en pratique WSL _(Windows Subsystem for Linux)_ ? Que permet-il de faire ? Quels protocoles utilise-t-il ?

WSL 1= Couche de compatibilité traduisant les syscalls Linux sur Windowss
WSL 2= Une machine virtuelle légère avec vrai noyau Linux qui est sur Hyper V

Les fonctionnalités= Uneexécution native des outils de Linux bash, grep, sed | Dev cross-platform et un accès au système de fichiers Windows `/mnt/c`

Les proto= 9P pour le partage de fichiers | networking virtuel localhost partagé entre Windows et WSL et intégration via VSock

## On commence

Vous venez d'installer un système GNU/Linux que faites-vous pour en sécuriser les accès via SSH ?

1. Mettre à jour le système= `apt update && apt upgrade`
2. Éditer `/etc/ssh/sshd_config`=
   - `PermitRootLogin no`= désactiver connexion root
   - `PasswordAuthentication no`= forcer authentification par clés
   - `Port 2222`= changer le port (optionnel, security by obscurity)
   - `AllowUsers user1 user2`= limiter les utilisateurs autorisés
3. Configurer le firewall= `ufw allow 22/tcp` puis `ufw enable`
4. Redémarrer SSH= `systemctl restart sshd`

## Alerte !

Vous vous connectez à un système GNU/Linux, le client ssh vous averti qu'il a un problème de reconnaissance de sa clef publique
d'hôte. Que faites-vous ? Pourquoi ?

**Attention** : Possible attaque MITM (Man-In-The-Middle) !

1. Vérifier= Contacter l'admin ou verif par canal autre si le système a été redownload
2. Comparer= Vérifier l'empreinte de la clé avec `ssh-keygen -lf /etc/ssh/ssh_host_ecdsa_key.pub` du côté serveur
3. Si leg= Supprimer l'ancienne key avec `ssh-keygen -R hostname` ou edit `~/.ssh/known_hosts`
4. Si suspect= Ne pas se connecter et chercher le problème

Raisons possible= Réinstallation dud système | régénération des clés | ou tentative d'interception

## Oh les gourmands !

Comment identifier rapidement, avec une seule ligne de commande, les utilisateurs qui consomment le plus d'espace de stockage.

_N.B._ Vous pouvez supposer que les répertoires personnels sont tous des sous-répertoires de `/home` 

```bash
du -sh /home/* | sort -hr | head -10
```

## On va aider les devs...

Une équipe de développeur.se.s vous demande de préparer un système GNU/Linux pour du développent système (C, make, etc.)

Que faites-vous en pratique en tant qu'administrateur.trice pour commencer ?

- Sur une distribution de la famille Debian=
```bash
apt update
apt install build-essential gcc g++ make git gdb valgrind
apt install autoconf automake libtool pkg-config
apt install linux-headers-$(uname -r) #Pour les kernls
```

- Sur une distribution de la famille Red Hat=
```bash
dnf groupinstall "Development Tools"
dnf install gcc gcc-c++ make git gdb valgrind
dnf install autoconf automake libtool pkgconfig
dnf install kernel-devel kernel-headers
```

## Qui fait quoi ?

Comment pourriez-vous retrouver la liste des dernières connexions d'utilisateurs à un système GNU/Linux, autant à distance que localement ?

Déjà :
```bash
last
```
pour les connexions locales :
```bash
lastb
```
pour les tentatives qui n'ont pas réussi et pour log SSH dans `/var/log/auth.log` ou `/var/log/secure`

Comment identifier les sessions où un.e utilisateur.trice à utilisé _sudo_ pour avoir les droit d'administration ?

```bash
grep sudo /var/log/auth.log
```
ou voir simplement `/var/log/secure` et chercher les entrées du sudo

## Post mortem

Le système que vous administrez est tombé cette nuit. Comment récupérer des informations sur ce qui s'est passé ?

```bash
journalctl --since "yesterday" --until "today"  # logs systemd
journalctl -p err..emerg  # uniquement erreurs critiques
last -x  # dernières connexions/redémarrages
cat /var/log/syslog /var/log/messages  # logs système
dmesg -T  # messages kernel avec les timestamps
grep -i "error\|fail\|panic" /var/log/syslog
uptime  # verif l'heure du dernier boot
```

Check aussi= `/var/log/kern.log` | `/var/log/auth.log` | `systemctl --failed`

## On surveille...

Sur un serveur GNU/Linux qui démarre via _systemd_, un service nommé _bidule_, comment :

- Savoir si le service est lancé au démarrage automatiquement :
```bash
systemctl is-enabled bidule
```
- Savoir s'il est en cours de fonctionnement :
```bash
systemctl is-active bidule
```
- Faire en sorte qu'il le soit :
```bash
systemctl enable bidule
```
- Examiner les messages qu'il émet :
```bash
journalctl -u bidule
```
- Le redémarrer :
```bash
systemctl restart bidule
```
- Déterminer de quel paquet il provient :
```bash
dpkg -S /usr/lib/systemd/system/bidule.service
```

## Zut

Vous devez reprendre la main, en tant qu'administrateur, d'un système GNU/Linux mais vous n'avez aucun mot de passe à votre disposition, ni _root_ ni utilisateur. Que pouvez-vous faire ?

**Méthode GRUB** :
1. Au boot appuyer sur E dans le menu GRUB
2. Trouver la ligne `linux` et ajouter en fin : `init=/bin/bash` | `single` | `rd.break`
3. Appuie sur `Ctrl+X` `| `F10` pour start
4. Remonter le système en write-read=
```bash
mount -o remount,rw /
```
5. Changer le pwd=
```bash
passwd root
```
6. Si SELinux est actif=
```bash
touch /.autorelabel
```
7. Restart avec =`exec /sbin/init`

## C'est la faute à Rémy

Un service réseau http(s) ne fonctionne plus. Vous avez accès à ce système comme administrateur.trice. Quels outils utilisez-vous pour diagnostiquer le problème, indiquez précisément ce que chaque outil permet de vérifier.

```bash
systemctl status apache2/nginx
```
Vérifie si le service est ON | crashé | affiche les dernières erreurs

```bash
journalctl -u apache2 -n 20 --no-pager
```
Affiche les 20 dernières lignes de logs du service

```bash
curl -I http://localhost
curl -Ik https://localhost
```
Test de connectivité HTTP/S local

```bash
apachectl configtest
```
Check la syntaxe de la configuration

```bash
tail -f /var/log/apache2/error.log
tail -f /var/log/nginx/error.log
```
Logs d'error en temps réel

```bash
ps aux | grep apache2
```
Check des processus en cours

```bash
ufw status
```
Check les rules du firewall

## Au charbon !

On vous demande d'installer un serveur de base de données _Elastic_ (autrefois nommé _Elastic Search_) sur un système Debian (ou autre si vous préférez)

Décrivez votre recherche documentaire, la méthode que vous sélectionnez, les commandes que vous exécutez et les fichiers de configuration impliqués.

Recherche documentaire=
- Documentation officielle= [Elasstic.co](https://www.elastic.co/guide/en/elasticsearch/reference/current/deb.html)
- Vérifier les predownload= Java 11 requis

Install=
```bash
# install Java
apt install openjdk-11-jdk

# import la key GPG
wget -qO - https://artifacts.elastic.co/GPG-KEY-elasticsearch | gpg --dearmor -o /usr/share/keyrings/elasticsearch-keyring.gpg

# add le depot APT
echo "deb [signed-by=/usr/share/keyrings/elasticsearch-keyring.gpg] https://artifacts.elastic.co/packages/8.x/apt stable main" | tee /etc/apt/sources.list.d/elastic-8.x.list

# install
apt update && apt install elasticsearch
```

Config= (`/etc/elasticsearch/elasticsearch.yml`) :
```yaml
cluster.name: my-cluster
node.name: node-1
network.host: 0.0.0.0
http.port: 9200
path.data: /var/lib/elasticsearch
path.logs: /var/log/elasticsearch
```

Au start=
```bash
systemctl daemon-reload
systemctl enable elasticsearch
systemctl start elasticsearch

# check
curl -X GET "localhost:9200/"
```

## Le Web

Vous avez la charge de gérer un serveur Web sous Debian GNU/Linux, pour héberger plusieurs sites Web (hébergement mutualisé).

- Quels paquets installez-vous ?
```bash
apt install apache2 php libapache2-mod-php mysql-server
apt install php-mysql php-curl php-gd php-mbstring php-xml php-zip
apt install certbot python3-certbot-apache  # SSL Let's Encrypt
```

- Comment organisez vous le stockage des différents contenus ?
```bash
/var/www/site1.com/public_html
/var/www/site1.com/logs
/var/www/site2.com/public_html
/var/www/site2.com/logs
```

- Certains sites ont besoin de PHP, comment l'installez-vous ?
```bash
apt install php libapache2-mod-php php-mysql php-cli
a2enmod php8.2
systemctl restart apache2
```

- Comment pouvez-vous organiser l'enregistrement séparé des visites des différents sites ?
Dans chaque virtualhost(`/etc/apache2/sites-available/site1.conf`)=
```apache
<VirtualHost *:80>
    ServerName site1.com
    DocumentRoot /var/www/site1.com/public_html
    CustomLog /var/www/site1.com/logs/access.log combined
    ErrorLog /var/www/site1.com/logs/error.log
</VirtualHost>
```

- Quel(s) outil(s) peuvent vous permettre de fournir une vision des visites de chaque site à chaque Webmaster ?
```bash
awstats
webalizer
goaccess
matomo
```

# Noé Brossard - B2 2026