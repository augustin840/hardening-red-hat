# Hardening pour Red Hat
## Déchiffrement du disque:

Installation des packages nécessaires pour le hardening:
```bash
sudo dnf update
sudo dnf install pamu2fcfg 
sudo dnf install dnf-automatic
```

L'étape consiste à configurer un deuxième mot de passe sur un autre slot disponible sur le disque chiffré afin de pouvoir déchiffrer le disque en cas de soucis avec l'utilisateur:

```bash
sudo cryptsetup luksAddKey /dev/nvme0n1p3*
```
La commande précédente assignera le nouveau mot de passe automatiquement au prochain slot disponible.


# Authentification deux facteurs pour s'identifier et utiliser la commande *sudo*


Génération du token pour l'utilisateur courant:
Attention: Veuillez entrer la clé avant de lancer la commande autrement le token ne pourra être généré.
```bash
pamu2fcfg -u `whoami` -opam://`hostname` -ipam://`hostname`
```

Note: il est également possible de générer un token pour un autre utilisateur en remplaçant l'option: ``-u `whoami` `` par ``-u `nom_utilisateur` ``

Copier le résultat puis le coller dans le fichier suivant: */etc/u2f_mappings* 

Remarque: il est possible d'ajouter un utilisateur sur chaque ligne du fichier. 

Ensuite, éditer le fichier **/etc/pam.d/gdm-passwor** pour l'authentification en mode graphique ainsi qu'en mode console, et **/etc/pam.d/sudo** pour l'authentification deux facteurs avec sudo

Ajouter la ligne suivante:

**auth sufficient\* pam_u2f.so origin=pam://$HOSTNAME appid=pam://$HOSTNAME authfile=/etc/u2f_mappings cue**

**Note: remplacer $HOSTNAME par le nom actuel de votre machine**
\* Sufficient signifie qu'il vous sera demandé d'utiliser la clé seulement si elle est connectée, autrement il faut remplacer *sufficient* par *required*

Mettre à jour PAM:
```bash
sudo pam-auth-update
```

**Tips: Il est fortement recommandé de garder un shell ouvert à côté avec l'identité root avec **sudo -i**, cela permettra de rétablir la configuration par défaut en cas d'erreur.**

**Répéter la manipulation avec la clé admin afin de pouvoir s'authentifier en tant qu'administrateur si jamais l'utilisateur à un problème.**

## Désactivtion du compte root

Il faut également désactiver le compte root. L'utilisateur peut uniquement utiliser sudo pour une élévation de droits.

Pour faire cela, il suffit d'éditer le fichier sudoers après avoir fait une sauvegarde de la configuration initiale.

```bash
sudo cp /etc/sudoers /etc/sudoers.bak
```

Ensuite il faut éditer le fichier sudoers à l'aide de la commande `sudo visudo`:

Dans la section commande aliases du fichier rajouter la ligne suivante:

```bash
Cmnd_Alias DISABLE_SU = /bin/su
```

Ensuite remplacer la ligne: *%sudo ALL=(ALL) ALL* par *%sudo ALL=(ALL) ALL, !DISABLE_SU*. 

Sauvegarder le fichier puis le fermer. 

## Désactivation de la fontion _visudo:_

Pour cela, il faut avoir un utilisateur admin, qui lui gardera accès au fichier en cas de problèmes. Il faut également s'asurer que l'utilisateur administrateur ait toutes les commandes autorisées. La ligne correspondante dans le fichier sudoers devrait ressembler à la suivante: `admin ALL=(ALL:ALL) ALL`

Une fois fait, on édite à nouveau le fichier */etc/sudoers* toujours à l'aide de `sudo visudo` et on rajoute un nouvel alias en précisant le chemin de la fonction *visudo*:

```bash
Cmnd_Alias DIABLE_VISUDO= /sbin/visudo
```

Et on édite à nouveau la ligne du groupe sudo en remplaçant *%sudo ALL=(ALL) ALL* par *%sudo ALL=(ALL) ALL, !DISABLE_SU, !DISABLE_VISUDO*. 

On peut maintenant tester la configuration en se connectant à un utilisateur membre du groupe sudo et entrer la commande `sudo visudo`. Si tout est fonctionnel l'utilisateur recevra le message suivant: "Sorry, user *user* is not allowed to execute /usr/sbin/visudo as root on *name_of_your_machine*"

## Mises à jour de sécurité automatiques:

Dans cette étape le but est d'automatiser les maj de sécurité.

il faut éditer le fichier: **/etc/dnf/automatic.conf** et remplacer la ligne `upgrade_type = default` par `upgrade_type = security` on peut également automatiser la manipulation en utilisant sed de la manière suivante:

```bash
sudo sed -i 's/^upgrade_type = default/upgrade_type = security/' /etc/dnf/automatic.conf
```

Une fois fait, il faut maintenant activer notre nouveau service à l'aide de la commande:
```bash
sudo systemctl enable --now dnf-automatic-install.timer
```

## Automatisation hardening

Il y a également un fichier nommé *auto-setup-hardening-rocky.sh* disponible sur la clé USB contenant tous les fichiers nécéssaires pour le Hardening, cependant il faut vérifier prudemment la configuration de l'ordinateur actuelle avant de le lancer. Le script est entièrement fonctionnel sur Red Hat avec les configurations de bases cependant en fonction des distributions ainsi que de l'usage de l'utilisateur les fichiers peuvent varier, le script sera donc à adapter. 









