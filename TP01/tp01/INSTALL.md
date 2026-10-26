# Installation de rhel9-lab et ubuntu24-lab

Note de procédure du TP 1 (ESGI). Elle doit permettre à un binôme qui ne connaît ni nos machines ni notre travail de reconstruire une machine identique.

- **Groupe** : Kosta DJORDJEVIC, Arsel DIFFO, Mathis
- **Dernière mise à jour** : 2026-10-26
- Les sorties brutes horodatées sont dans `tp01/preuves/`. Les extraits ci-dessous en sont tirés.
- Le déroulé chronologique, les temps passés et les erreurs de méthode sont dans `docs/journal.md`.

---

## 1. Périmètre

| | rhel9-lab | ubuntu24-lab |
|---|---|---|
| Rôle | Serveur applicatif | Serveur de référence |
| Distribution | Red Hat Enterprise Linux 9.8 (Plow) | Ubuntu 24.04.5 LTS (Noble Numbat) |
| Noyau | `5.14.0-687.56.1.el9_8.x86_64` | `6.8.0-146-generic` |
| Nom d'hôte | `rhel9-lab` | `ubuntu24-lab` |
| Adresse NAT | 192.168.224.132 | 192.168.224.131 |

Les adresses NAT sont attribuées par DHCP et peuvent changer : les relever avec `ip -br a`.

Relevé avec :

```bash
cat /etc/os-release
uname -r
hostnamectl
```

**Poste d'administration** : PC Windows, client OpenSSH en PowerShell, qui joint les machines par leur interface NAT. Le réseau interne `lab-interne` n'est pas joignable depuis ce poste.

---

## 2. Ressources

### 2.1 Poste et images

- Hyperviseur : VMware Workstation 17.5.2 (build 23775571).
- Virtualisation matérielle activée dans le micrologiciel (`systeminfo` : « Virtualisation activée dans le microprogramme : Oui »).
- Images vérifiées avec `Get-FileHash <image> -Algorithm SHA256`, comparées à la somme publiée par l'éditeur :

| Image | SHA-256 | Résultat |
|---|---|---|
| `rhel-9.8-x86_64-boot.iso` | `18fecefe070e9c40502287281ec86e3bd4d64e2f48796a1cb166d9371308e5e6` | identique à developers.redhat.com |
| `ubuntu-24.04.5-live-server-amd64.iso` | `97f3d7ffb032c3eb3b23d2c8be9cc76e60c2c1f2c0146ba5ba9fe01cafae0fd8` | identique à ubuntu.com |

### 2.2 Machines virtuelles

| Paramètre | rhel9-lab | ubuntu24-lab |
|---|---|---|
| Processeurs | 2 vCPU | 2 vCPU |
| Mémoire | 2048 Mio | 2048 Mio |
| Disque système | 30 Gio, alloué dynamiquement | 25 Gio, alloué dynamiquement |
| Second disque | aucun (ajouté au TP 2) | aucun (ajouté au TP 2) |
| Instantané initial | `avant-installation` | `avant-installation` |

### 2.3 Réseau

| Carte | Mode VMware | Usage | rhel9-lab | ubuntu24-lab |
|---|---|---|---|---|
| 1 | NAT (VMnet8) | Mises à jour et accès depuis le poste d'administration | DHCP : 192.168.224.132 | DHCP : 192.168.224.131 |
| 2 | LAN Segment `lab-interne` | Réseau interne 10.42.0.0/24 | 10.42.0.1 | 10.42.0.2 |

Le mode pont (*Bridged*) n'est pas utilisé : il est interdit par les règles du lab.

---

## 3. Partitionnement

### 3.1 Schéma appliqué

Le schéma du sujet totalise 43 Gio (1 + 20 + 10 + 5 + 5 + 2), ce qui dépasse nos disques de 30 et 25 Gio. Les volumes ont été réduits en conservant les proportions et un espace libre dans le groupe de volumes.

| Volume | Point de montage | Taille rhel9-lab | Taille ubuntu24-lab | FS rhel9-lab | FS ubuntu24-lab | Rôle |
|---|---|---|---|---|---|---|
| `/boot` | `/boot` | 1 Gio | 1 Gio | xfs | ext4 | Fichiers de démarrage, hors LVM |
| root | `/` | 10 Gio | 8 Gio | xfs | ext4 | Système et programmes |
| var | `/var` | 5 Gio | 4 Gio | xfs | ext4 | Données qui changent : caches, files d'attente, bases de services |
| log | `/var/log` | 3 Gio | 2 Gio | xfs | ext4 | Journaux des services |
| home | `/home` | 2 Gio | 2 Gio | xfs | ext4 | Fichiers des utilisateurs |
| swap | swap | 2 Gio | 2 Gio | swap | swap | Extension de la mémoire vive |
| tmpfs | `/tmp` | — | 512 Mio | — | tmpfs | Fichiers temporaires |
| *Libre dans le groupe* | — | ≈ 7 Gio | ≈ 6 Gio | — | — | Réservé à l'agrandissement (TP 2) |

### 3.2 Écarts avec le sujet et entre les deux machines

- **Tailles** : réduites par rapport au sujet (voir 3.1).
- **`/tmp`** : en tmpfs de 512 Mio sur Ubuntu uniquement, ajouté à la main (voir 4). Sur RHEL, aucun montage dédié n'est demandé : [À COMPLÉTER : résultat de `findmnt /tmp`].

### 3.3 Marche à suivre dans les installateurs

**RHEL 9 (Anaconda)**

1. *Installation Source* : configurer la source réseau et l'abonnement.
2. *Installation Destination* : sélectionner le disque, choisir **Custom**.
3. *Manual Partitioning* : schéma **LVM**.
4. Créer `/boot` (1 GiB, partition standard), puis `/` (10 GiB), ce qui crée le groupe de volumes.
5. Renommer le groupe via **Modify…**, taille de groupe fixe pour garder l'espace libre.
6. Ajouter `/var` (5 GiB), `/var/log` (3 GiB), `/home` (2 GiB) et `swap` (2 GiB), et nommer chaque volume.
7. Contrôler l'espace libre du groupe (~ 7 Gio), puis valider.

**Ubuntu Server 24.04 (Subiquity)**

1. *Guided storage configuration* : **Custom storage layout**.
2. Sur le disque : **Use As Boot Device**.
3. **Add GPT Partition** : 1 Go, ext4, `/boot`.
4. **Add GPT Partition** avec le reste du disque, format *Leave unformatted*.
5. **Create volume group (LVM)** avec cette partition, nommé `vg_sys`, sans chiffrement.
6. **Create Logical Volume** : root (8 Go, `/`), var (4 Go, `/var`), log (2 Go, `/var/log`), home (2 Go, `/home`), swap (2 Go).
7. Laisser le reste du groupe non alloué, relire le résumé, valider.

### 3.4 Justification des choix

**Pourquoi séparer `/var`, et surtout `/var/log`, du volume racine ?**  
Une partition dédiée pour /var/log est utile pour isoler l'espace occupé par les logs et prévenir une utilisation excessive de l'espace disque qui pourrait affecter d'autres parties du système. Par exemple un service qui journalise en boucle remplit /var/log ; sans volume séparé, la racine sature et la machine ne peut plus rien écrire.

**Pourquoi `/boot` reste-t-il une partition classique, hors du groupe de volumes ?**  
Dans ce cas précis, c'est utile de ne pas placer la partition /boot dans le lvm car la partition root peut avoir des caractéristiques qui ne la rendent pas bootable directement. GRUB doit lire le noyau avant que le lvm soit activé, donc une partition classique est plus simple et plus sûre.

**Pourquoi 2 Gio de mémoire d'échange sur 2 Gio de mémoire vive, plutôt que 8 Gio ou rien ?**  
Dans ce cas, prévoir beaucoup plus de swap que de RAM physique (2Gio et 8Gio) est une mauvaise idée dans le sens où l'espace dans lequel la RAM va puiser se situe sur le disque dur, qui a fatalement un temps d'accès à la ressource beaucoup plus lent de la RAM physique. Le système va passer plus de temps à déplacer des données qu'à travailler, et ne rien mettre non plus est une mauvaise idée, la mémoire swap sert de filet de sécurité, éviter un plantage du système (qui n'a plus assez de RAM pour opérer), et globalement son rôle va être d'absorber des pics d'utilisation de la RAM. 
Source : [documentation Red Hat 9 sur le swap](https://docs.redhat.com/en/documentation/red_hat_enterprise_linux/9/html/managing_storage_devices/getting-started-with-swap_managing-storage-devices).

**Pourquoi ne pas allouer la totalité du disque, et à quoi servira l'espace restant ?**  
L'espace libre du groupe de volumes permet d'agrandir un volume à chaud et de prendre des instantanés LVM, sans repartitionner. Il sera utilisé au TP 2.

**Quel volume agrandir en premier sur un serveur qui tourne, et quelle opération maîtriser ?**  
Le volume à agrandir en premier sur un serveur est le volume des logs (/var/log). Il s'agit d'un volume qui se remplit avec l'activité du serveur et si ce volume sature, le service qui journalise ne peut plus écrire, parfois s'arrête, et on perd la trace de ce qui s'est passé. D'où l'intérêt, d'ailleurs, de l'avoir isolé (on limite la panne à /var/log).  
L'opération à maitriser s'appelle le *Online Resizing*. Ici on l'opère en deux temps, on agrandit d'abord le volume logique et ensuite le système de fichiers qu'il contient, sans démonter quoi que ce soit.

---

## 4. Installation

| Élément | rhel9-lab | ubuntu24-lab |
|---|---|---|
| Profil | Sélection des logiciels : installation minimale | Ubuntu Server (minimized) |
| Nom d'hôte | `rhel9-lab` | `ubuntu24-lab` |
| Langue / locale | `fr_FR.UTF-8` | `fr_FR.UTF-8`, ajoutée après l'installation |
| Clavier | `fr` | `fr` |
| Fuseau horaire | `Europe/Paris` (choisi à l'installation) | `Europe/Paris`, réglé après l'installation |
| Compte créé à l'installation | `fousta` (nominatif) | `fousta` (nominatif) |
| Synchronisation de l'heure | `timedatectl status System clock synchronized: yes` | [À COMPLÉTER : `timedatectl status`] |
| Paquets ajoutés | `vim` | `vim`, `iputils-ping`, `language-pack-fr` |

**Ubuntu : locale, fuseau et `/tmp` (réglages absents après l'installation)**

```bash
sudo apt install language-pack-fr
sudo timedatectl set-timezone Europe/Paris

echo 'tmpfs  /tmp  tmpfs  defaults,noatime,nosuid,nodev,size=512M  0  0' | sudo tee -a /etc/fstab
sudo systemctl daemon-reload
sudo mount /tmp
findmnt /tmp
```
**Sur RHEL 9, le répertoire /tmp est automatiquement monté en tant que tmpfs (un système de fichiers temporaire stocké directement en RAM). C'est pourquoi on le voit immédiatement avec la commande df -hT. Sur Ubuntu, /tmp n'est pas un point de montage séparé; il s'agit simplement d'un dossier classique situé sur la racine (/).*  
Vérification : `hostnamectl`, `localectl status`, `timedatectl status`, `systemctl --failed` (résultat vide attendu). Sorties dans `tp01/preuves/`.

---

## 5. Mises à jour

### ubuntu24-lab

```bash
sudo apt -y upgrade
tail -20 /var/log/apt/history.log
```

Extrait de l'historique (liste des paquets abrégée) :

```text
Start-Date: 2026-10-12  17:50:45
Commandline: apt -y full-upgrade
Requested-By: fousta (1000)
Upgrade: netplan-generator:amd64 (1.1.2-8ubuntu1~24.04.2, 1.1.2-8ubuntu1~24.04.3), [...], libnetplan1:amd64 (1.1.2-8ubuntu1~24.04.2, 1.1.2-8ubuntu1~24.04.3)
End-Date: 2026-10-12  17:51:57
```

### rhel9-lab

```bash
sudo dnf -y update
sudo dnf history | head -20
sudo dnf repolist
```

```text
Mise à jour des référentiels de gestion des abonnements.
Dernière vérification de l'expiration des métadonnées effectuée il y a 0:00:02 le lun. 12 oct. 2026 17:52:41.
Dépendances résolues.
Rien à faire.
Terminé!

ID     | Ligne de commande | Date et heure    | Action(s) | Modifié
     2 | install vim -y    | 2026-10-12 14:07 | Install   |    4  <
     1 |                   | 2026-10-12 13:19 | Install   |  394 >E

rhel-9-for-x86_64-appstream-rpms   Red Hat Enterprise Linux 9 for x86_64 - AppStream (RPMs)
rhel-9-for-x86_64-baseos-rpms      Red Hat Enterprise Linux 9 for x86_64 - BaseOS (RPMs)
```
---

## 6. Comptes et accès distant

### 6.1 Comptes et délégation

| Compte | Rôle | Groupe de délégation RHEL | Groupe de délégation Ubuntu |
|---|---|---|---|
| `fousta` | Compte créé à l'installation | `wheel` | `sudo` |
| `kosta`, `mathis`, `arsel` | Comptes nominatifs, un par membre | `wheel` | `sudo` |

Délégation par le groupe prévu par la distribution (`usermod -aG`), jamais par modification de l'identifiant numérique. Aucun compte partagé. Un ajout à un groupe n'est effectif qu'après reconnexion (observation en 6.3).

Verrouillage de root (`sudo passwd -l root`, puis `sudo passwd -S root`) : 
```bash
fousta@ubuntu24-lab:~$ sudo passwd -l root
[sudo] password for fousta:
passwd: password changed.
fousta@ubuntu24-lab:~$ sudo passws -S root
sudo: passws: command not found
fousta@ubuntu24-lab:~$ sudo passwd -S root
root L 2026-09-09 0 99999 7 -1

[fousta@rhel9-lab ~]$ sudo passwd -l root
[sudo] Mot de passe de fousta :
Désolé, essayez de nouveau.
[sudo] Mot de passe de fousta :
Verrouillage du mot de passe pour l'utilisateur root.
passwd: Succès
[fousta@rhel9-lab ~]$ sudo passwd -S root
root LK 1969-12-31 0 99999 7 -1 (Mot de passe verrouillé.)
```  

### 6.2 Connexion par clé depuis le poste d'administration

```powershell
ssh-keygen -t ed25519 -a 100 -C "$env:USERNAME@lab"
```

L'emplacement par défaut est accepté et la clé est protégée par une passphrase. La clé privée ne quitte jamais le poste.

Déploiement de la clé publique, une fois par compte et par machine. Cette étape se fait encore par mot de passe, avant tout durcissement :

```powershell
type $env:USERPROFILE\.ssh\id_ed25519.pub | ssh kosta@192.168.224.131 "mkdir -p ~/.ssh && chmod 700 ~/.ssh && tr -d '\r' >> ~/.ssh/authorized_keys && chmod 600 ~/.ssh/authorized_keys"
type $env:USERPROFILE\.ssh\id_ed25519.pub | ssh kosta@192.168.224.132 "mkdir -p ~/.ssh && chmod 700 ~/.ssh && tr -d '\r' >> ~/.ssh/authorized_keys && chmod 600 ~/.ssh/authorized_keys"
```

`tr -d '\r'` retire les retours chariot Windows, qui rendent parfois la ligne de clé invalide. Sur RHEL, SELinux doit reconnaître les fichiers : `restorecon -R ~/.ssh`.

Test : la ligne `Authenticated ... using "publickey"` prouve que la clé a servi.

```powershell
ssh -v kosta@192.168.224.132 exit 2>&1 | Select-String "Authenticated|publickey"
ssh -v kosta@192.168.224.131 exit 2>&1 | Select-String "Authenticated|publickey"
```

```text
Authenticated to 192.168.224.132 ([192.168.224.132]:22) using "publickey".
Authenticated to 192.168.224.131 ([192.168.224.131]:22) using "publickey".
```

### 6.3 Test de la délégation

La commande du sujet (`sudo -n true && echo delegation-ok`) échoue par construction (voir 9, incident 2). Version adaptée, sans `NOPASSWD` :

```powershell
ssh -t kosta@192.168.224.131 'id ; sudo -v && echo delegation-ok'
```

```text
uid=1001(kosta) gid=1001(kosta) groups=1001(kosta),27(sudo),100(users)
[sudo] password for kosta:
delegation-ok
```

Même test pour `kosta` sur RHEL, et pour `mathis` et `arsel` sur les deux machines.

**Observation : le groupe n'est effectif qu'après reconnexion.**

### 6.4 Configuration `sshd` retenue

Objectif : refuser la connexion directe de root. L'authentification par mot de passe reste active : sa suppression est traitée au TP 6.

**ubuntu24-lab** : réglage `PermitRootLogin no` dans `/etc/ssh/sshd_config`, puis :

```bash
date -Is
sudo sshd -T | grep -Ei 'permitrootlogin|passwordauthentication'
sudo vim /etc/ssh/sshd_config
sudo sshd -t
sudo systemctl reload ssh
sudo sshd -T | grep -Ei 'permitrootlogin|passwordauthentication'
```

```text
2026-10-26T10:10:03+01:00
permitrootlogin without-password       <- avant
passwordauthentication yes
permitrootlogin no                     <- après
passwordauthentication yes
```

Test depuis le poste d'administration :

```powershell
ssh -o BatchMode=yes root@192.168.224.131 true
echo $LASTEXITCODE
```

```text
root@192.168.224.131: Permission denied (publickey,password).
255
```

Le refus seul ne prouve pas le réglage, car le compte root est aussi verrouillé et sans clé autorisée : la preuve est la sortie de `sshd -T` après rechargement (« la configuration effective »).

**rhel9-lab** : [À COMPLÉTER : valeur initiale de `sshd -T`, réglage appliqué, sortie après rechargement, test de refus de root].

**Justification** : le compte root est connu de tous les attaquants et sert de cible aux attaques par force brute. Une connexion nominative laisse en plus une trace de qui a fait quoi.

---

## 7. Pare-feu

| | rhel9-lab | ubuntu24-lab |
|---|---|---|
| Outil | `firewalld` | `ufw` |
| État initial | [À COMPLÉTER] | inactif (`Status: inactive`) |
| Règle appliquée | service `ssh` uniquement | profil `OpenSSH` uniquement |

**ubuntu24-lab** : la règle SSH est créée avant l'activation, sinon l'accès distant serait coupé.

```bash
sudo ufw allow OpenSSH
sudo ufw enable
sudo ufw status verbose
```

```text
Status: active
Logging: on (low)
Default: deny (incoming), allow (outgoing), disabled (routed)

To                         Action      From
22/tcp (OpenSSH)           ALLOW IN    Anywhere
22/tcp (OpenSSH (v6))      ALLOW IN    Anywhere (v6)
```

**rhel9-lab** :

```bash
sudo firewall-cmd --permanent --add-service=ssh
sudo firewall-cmd --reload
sudo firewall-cmd --list-all
```

[À COMPLÉTER : sortie de `firewall-cmd --list-all`].

**Méthode de vérification** : depuis le poste d'administration, après l'activation et sans fermer la session d'origine.

```powershell
Get-Date -Format o
ssh -t kosta@192.168.224.131 'id ; sudo whoami'
Test-NetConnection 192.168.224.131 -Port 22
Test-NetConnection 192.168.224.131 -Port 80
```

```text
2026-10-26T10:28:19.5966416+01:00
uid=1001(kosta) gid=1001(kosta) groups=1001(kosta),27(sudo),100(users)
root
Port 22 : TcpTestSucceeded : True
Port 80 : TcpTestSucceeded : False
```

Le port 80 n'a aucun service qui écoute : ce refus ne prouve pas à lui seul le filtrage. La preuve de l'ouverture unique est l'état complet des règles (`ufw status verbose`, `firewall-cmd --list-all`).

Même vérification sur rhel9-lab : [À COMPLÉTER].

---

## 8. Écarts entre les deux distributions

| Opération | RHEL 9 | Ubuntu 24.04 | Raison |
|---|---|---|---|
| Système de fichiers par défaut | xfs | ext4 | Choix par défaut de chaque installateur |
| Gestionnaire de paquets | `dnf` | `apt` | Familles de distributions différentes (rpm contre deb) |
| Configuration réseau | `nmcli` (NetworkManager) | fichier dans `/etc/netplan/` puis `netplan apply` | Pile réseau par défaut différente |
| Locale française | déjà disponible | `apt install language-pack-fr` | Anaconda installe les paquets de langue choisis à l'installation, pas Subiquity |
| Fuseau horaire | `Europe/Paris` déjà réglé | `timedatectl set-timezone Europe/Paris` | Le fuseau est demandé dans Anaconda, pas dans Subiquity |
| `/tmp` en tmpfs | non demandé par le sujet | ajouté à la main dans `/etc/fstab` | Le sujet le prévoit uniquement sur Ubuntu, et l'installateur ne gère pas ce montage |

Tableau en cours de complétion avec les opérations des parties C et D.

---

## 9. Incidents et corrections

### Incident 1 — Test de délégation du sujet inadapté

- **Symptôme** : `ssh kosta@... 'id ; sudo -n true && echo delegation-ok'` affiche `sudo: a password is required`. Avec `sudo whoami` sans terminal : `a terminal is required to read the password`.  
- **Diagnostic** : L'option `-n` (--non-interactive) évite de solliciter une quelconque saisie de la part de l'utilisateur. Si un mot de passe est requis pour l'exécution de la commande, sudo affichera un message d'erreur et se terminera. Il réussit donc uniquement si aucun mot de passe n'est nécessaire. Et comme le compte kosta a une délégation normale (donc un mot de passe requis). Sans option `-t`, `ssh` n'ouvre pas de terminal pour lire le mot de passe. La délégation elle-même était correcte (`groups=...27(sudo)...`).  
- **Correction** : Pour notre cas nous allons adapter la commande pour qu'elle puisse fonctionner, sans passer par NOPASSWD (qui serait un affaiblissement du durcicement), avec les arguements -t (pour ouvrir un terminal quand on lui donne une commande) et sudo -v qui valide les droits de l'utilisateur sans lancer de commande. Il demande le mot de passe, réussit si kosta a le droit d'utiliser sudo, et delegation-ok s'affiche alors : `ssh -t kosta@... 'id ; sudo -v && echo delegation-ok'`.  
```powershell
PS C:\WINDOWS\System32> ssh -t kosta@192.168.224.131 'id ; sudo -v && echo delegation-ok'
Enter passphrase for key 'C:\Users\fousta/.ssh/id_ed25519':
uid=1001(kosta) gid=1001(kosta) groups=1001(kosta),27(sudo),100(users)
[sudo] password for kosta:
delegation-ok
Connection to 192.168.224.131 closed.
```

---

## 10. Reconstruction rapide

Prérequis : machines créées selon 2.2 et 2.3, installation minimale faite selon 3.3 (partitionnement du tableau 3.1), compte administrateur créé à l'installation, réseau NAT fonctionnel. Remplacer `<interface-interne>` (relevée avec `ip -br a`) et `<addr>` (adresse libre de 10.42.0.0/24).



### Vérification finale, sur les deux machines

```bash
date -Is
lsblk -f && sudo vgs && sudo lvs
df -hT && swapon --show
systemctl --failed
sudo passwd -S root
sudo sshd -T | grep -Ei 'permitrootlogin|passwordauthentication'
```

**Règle de survie!!!** : ne jamais fermer la session d'origine (console VMware) avant d'avoir ouvert une seconde session avec le nouveau réglage et vérifié qu'elle fonctionne.
