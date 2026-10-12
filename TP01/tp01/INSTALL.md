# Procédure d'installation rhel9-lab et ubuntu24-lab

## 0. Liste de contrôle matérielle

### 0.1 Virtualisation matérielle activée dans le micrologiciel du poste

**Windows** : `Win+R` > `cmd` > `systeminfo`

Sortie :

```text
Sécurité basée sur la virtualisation:   État : Non activé
                                        App Control for Business policy: Appliqué
                                        App Control for Business user mode policy: Audit
                                        Fonctionnalités de sécurité activées :

Configuration requise pour Hyper-V:     Extensions de mode du moniteur d’ordinateur virtuel : Oui
                                        Virtualisation activée dans le microprogramme : Oui
                                        Traduction d’adresse de second niveau : Oui
                                        Prévention de l’exécution des données disponible : Oui
```

**Linux** : dans un terminal

```bash
lscpu | grep -i virtual
```

### 0.2 Hyperviseur installé et à jour

```cmd
reg query "HKLM\SOFTWARE\WOW6432Node\VMware, Inc.\VMware Workstation" /v ProductVersion
```

Sortie :

```text
HKEY_LOCAL_MACHINE\SOFTWARE\WOW6432Node\VMware, Inc.\VMware Workstation
    ProductVersion    REG_SZ    17.5.2.23775571
```

Version relevée : **VMware Workstation 17.5.2 (build 23775571)**.

### 0.3 Espace disque

```cmd
C:\Windows\System32>dir C:\
```

Sortie :

```text
 Le volume dans le lecteur C n’a pas de nom.
 Le numéro de série du volume est 8EB4-4B2F

 Répertoire de C:\

07/03/2026  01:54    <DIR>          inetpub
29/09/2026  17:53    <DIR>          Intel
29/09/2026  18:44    <DIR>          ISO
01/04/2024  09:26    <DIR>          PerfLogs
29/09/2026  15:30    <DIR>          Program Files
29/09/2026  15:30    <DIR>          Program Files (x86)
02/10/2026  12:27                10 TP2 Exos C.txt
29/09/2026  14:35    <DIR>          Users
12/10/2026  11:21    <DIR>          VM
29/09/2026  16:07    <DIR>          Windows
               1 fichier(s)               10 octets
               9 Rép(s)  939 680 542 720 octets libres
```

Espace libre sur `C:` : **939 680 542 720 octets (≈ 875 Gio)**.

### 0.4 Mémoire disponible pour deux VM en parallèle

```cmd
systeminfo
```

Sortie :

```text
Mémoire physique totale:                    16 163 Mo
Mémoire physique disponible:                8 917 Mo
Mémoire virtuelle : taille maximale:        19 107 Mo
Mémoire virtuelle : disponible:             11 408 Mo
Mémoire virtuelle : en cours d’utilisation: 7 699 Mo
```

### 0.5 Images d'installation téléchargées et vérifiées

#### Ubuntu Server 24.04.5

```powershell
PS C:\ISO> Get-FileHash .\ubuntu-24.04.5-live-server-amd64.iso -Algorithm SHA256
```

```text
Algorithm       Hash                                                                   Path
---------       ----                                                                   ----
SHA256          97F3D7FFB032C3EB3B23D2C8BE9CC76E60C2C1F2C0146BA5BA9FE01CAFAE0FD8       C:\ISO\ubuntu-24.04.5-live-se...
```

- Source : <https://ubuntu.com/download/server/thank-you?version=24.04.5&architecture=amd64&lts=true>
- Somme SHA-256 officielle : `97f3d7ffb032c3eb3b23d2c8be9cc76e60c2c1f2c0146ba5ba9fe01cafae0fd8`
- Résultat : identique (la comparaison est insensible à la casse).

#### RHEL 9.8

```powershell
PS C:\ISO> Get-FileHash .\rhel-9.8-x86_64-boot.iso -Algorithm SHA256
```

```text
Algorithm       Hash                                                                   Path
---------       ----                                                                   ----
SHA256          18FECEFE070E9C40502287281EC86E3BD4D64E2F48796A1CB166D9371308E5E6       C:\ISO\rhel-9.8-x86_64-boot.iso
```

- Source : <https://developers.redhat.com/products/rhel/download#downloadsbyrelease> (architecture `x86_64`, image *Boot ISO*)
- Somme SHA-256 officielle : `18fecefe070e9c40502287281ec86e3bd4d64e2f48796a1cb166d9371308e5e6`
- Résultat : identique.

---

## 1. Périmètre
```text
fousta@ubuntu24-lab:~$ cat /etc/os-release
PRETTY_NAME="Ubuntu 24.04.5 LTS"
NAME="Ubuntu"
VERSION_ID="24.04"
VERSION="24.04.5 LTS (Noble Numbat)"
VERSION_CODENAME=noble
ID=ubuntu
ID_LIKE=debian
HOME_URL="https://www.ubuntu.com/"
SUPPORT_URL="https://help.ubuntu.com/"
BUG_REPORT_URL="https://bugs.launchpad.net/ubuntu/"
PRIVACY_POLICY_URL="https://www.ubuntu.com/legal/terms-and-policies/privacy-policy"
UBUNTU_CODENAME=noble
LOGO=ubuntu-logo
fousta@ubuntu24-lab:~$ uname -r
6.8.0-146-generic
fousta@ubuntu24-lab:~$ hostnamectl
 Static hostname: ubuntu24-lab
       Icon name: computer-vm
         Chassis: vm 🖴
      Machine ID: fbd006756f38402bbbde01d1b64dd65b
         Boot ID: 4ede13aab76b46508aa7eb1fc9769534
  Virtualization: vmware
Operating System: Ubuntu 24.04.5 LTS
          Kernel: Linux 6.8.0-146-generic
    Architecture: x86-64
 Hardware Vendor: VMware, Inc.
  Hardware Model: VMware Virtual Platform
Firmware Version: 6.00
   Firmware Date: Thu 2020-11-12
    Firmware Age: 5y 10month 4w 1d
```
```text
[fousta@rhel9-lab ~]$ cat /etc/os-release
NAME="Red Hat Enterprise Linux"
VERSION="9.8 (Plow)"
ID="rhel"
ID_LIKE="fedora"
VERSION_ID="9.8"
PLATFORM_ID="platform:el9"
PRETTY_NAME="Red Hat Enterprise Linux 9.8 (Plow)"
ANSI_COLOR="0;31"
LOGO="fedora-logo-icon"
CPE_NAME="cpe:/o:redhat:enterprise_linux:9::baseos"
HOME_URL="https://www.redhat.com/"
DOCUMENTATION_URL="https://access.redhat.com/documentation/en-us/red_hat_enterprise_linux/9"
BUG_REPORT_URL="https://issues.redhat.com/"

REDHAT_BUGZILLA_PRODUCT="Red Hat Enterprise Linux 9"
REDHAT_BUGZILLA_PRODUCT_VERSION=9.8
REDHAT_SUPPORT_PRODUCT="Red Hat Enterprise Linux"
REDHAT_SUPPORT_PRODUCT_VERSION="9.8"
[fousta@rhel9-lab ~]$ uname -r
5.14.0-687.56.1.el9_8.x86_64
[fousta@rhel9-lab ~]$ hostnamectl
 Static hostname: rhel9-lab
       Icon name: computer-vm
         Chassis: vm 🖴
      Machine ID: 9d0eca9bf6b2431aadcce38fad31f4a0
         Boot ID: 6aa03bd14eec465298ac56abf6e5d76f
  Virtualization: vmware
Operating System: Red Hat Enterprise Linux 9.8 (Plow)
     CPE OS Name: cpe:/o:redhat:enterprise_linux:9::baseos
          Kernel: Linux 5.14.0-687.56.1.el9_8.x86_64
    Architecture: x86-64
 Hardware Vendor: VMware, Inc.
  Hardware Model: VMware Virtual Platform
Firmware Version: 6.00
```
## 2. Ressources des machines virtuelles

| Paramètre | rhel9-lab | ubuntu24-lab |
| --- | --- | --- |
|Hyperviseur| VMware Workstation 17.5.2 | VMware Workstation 17.5.2 |
|Image|rhel-9.8-x86_64-boot.iso|ubuntu-24.04.5-live-server-amd64.iso|
|Processeur|2 vCPU|2 vCPU|
|Mémoire|2048 Mio|2048 Mio|
|Disque système|30 Gio, alloué dynamiquement|25 Gio, alloué dynamiquement|
|Second disque|aucun (ajouté au TP 2)|aucun (ajouté au TP 2)|
|Snapshot|avant-installation|avant-installation|

## 3. Partitionnement appliqué

| Volume | Point de montage | Taille rhel9-lab | Taille ubuntu24-lab | FS rhel9-lab | FS ubuntu24-lab | Utilité |
|---|---|---|---|---|---|---|
|/dev/sda1|/boot|1 Gio|1 Gio|xfs|ext4|Fichiers de démarrage|
|lv_root|/|10 Gio|8 Gio|xfs|ext4|Contient le système d'exploitation et les programmes|
|lv_var|/var|5 Gio|4 Gio|xfs|ext4|Stocke tout ce qui change au fil du temps : journaux, caches, files d'attente, bases de données de services|
|lv_log|/var/log|3 Gio|2 Gio|xfs|ext4|Journaux des services|
|lv_home|/home|2 Gio|2 Gio|xfs|ext4|Fichiers utilisateur x|
|lv_swap|swap|2 Gio|2 Gio|swap|swap|Espace que Linux utilise comme extension de la mémoire vive|
|tmpfs|/tmp|--|512 Mio|--|tmpfs|Fichiers temporaires|
|vg_sys|--|~7 Gio|~6 Gio|--|--|Espace libre/restant|

**Justification des choix :**

**Pourquoi séparer /var , et surtout /var/log , du volume racine ?**  
Une partition dédiée pour /var/log est utile pour isoler l'espace occupé par les logs et prévenir une utilisation excessive de l'espace disque qui pourrait affecter d'autres parties du système. Par exemple un service qui journalise en boucle remplit /var/log ; sans volume séparé, la racine sature et la machine ne peut plus rien écrire.

**Pourquoi /boot reste-t-il une partition classique et n'est-il pas placé dans le groupe de volumes ?**  
Dans ce cas précis, c'est utile de ne pas placer la partition /boot dans le lvm car la partition root peut avoir des caractéristiques qui ne la rendent pas bootable directement. GRUB doit lire le noyau avant que le lvm soit activé, donc une partition classique est plus simple et plus sûre.

**Pourquoi 2 Gio de mémoire d'échange sur une machine de 2 Gio de mémoire vive, plutôt que 8 Gio ou rien du tout ?**  
Dans ce cas, prévoir beaucoup plus de swap que de RAM physique (2Gio et 8Gio) est une mauvaise idée dans le sens où l'espace dans lequel la RAM va puiser se situe sur le disque dur, qui a fatalement un temps d'accès à la ressource beaucoup plus lent de la RAM physique. Le système va passer plus de temps à déplacer des données qu'à travailler, et ne rien mettre non plus est une mauvaise idée, la mémoire swap sert de filet de sécurité, éviter un plantage du système (qui n'a plus assez de RAM pour opérer), et globalement son rôle va être d'absorber des pics d'utilisation de la RAM.  
*https://docs.redhat.com/en/documentation/red_hat_enterprise_linux/9/html/managing_storage_devices/getting-started-with-swap_managing-storage-devices*

**Pourquoi ne pas allouer la totalité du disque, et à quoi servira l'espace restant ?**  
L'espace restant sert à garder de la marge en cas d'évolution du système. Dans les cas où l'on a besoin d'agrandir un volume à chaud et de prendre des snapshot. 

**Quel volume, d'après vous, devra être agrandi en premier sur un serveur qui tourne, et quelle opération technique faudra-t-il maîtriser pour le faire sans coupure ?**  
Le volume à agrandir en premier sur un serveur est le volume des logs (/var/log). Il s'agit d'un volume qui se remplit avec l'activité du serveur et si ce volume sature, le service qui journalise ne peut plus écrire, parfois s'arrête, et on perd la trace de ce qui s'est passé. D'où l'intérêt, d'ailleurs, de l'avoir isolé (on limite la panne à /var/log).  
L'opération à maitriser s'appelle le *Online Resizing*. Ici on l'opère en deux temps, on agrandit d'abord le volume logique et ensuite le système de fichiers qu'il contient, sans démonter quoi que ce soit.

## 4. Installation

| Élément | rhel9-lab | ubuntu24-lab |
|---|---|---|
|Profil|Séléction des logiciels > installation minimale|Ubuntu Server (minimized)|
|Nom d'hôte|rhel9-lab|ubuntu24-lab|
|Langue / locale|fr_FR.UTF-8|fr_FR.UTF-8 (ajoutée après installation)|
|Clavier|fr|fr|
|Fuseau horaire|Europe/Paris|Europe/Paris (après MàJ manuelle)|

## 5. Mises à jour

```text
fousta@ubuntu24-lab:~$ tail -20 /var/log/apt/history.log
Install: vim:amd64 (2:9.1.0016-1ubuntu7.20), libgpm2:amd64 (1.20.7-11, automatic), vim-common:amd64 (2:9.1.0016-1ubuntu7.20, automatic), libsodium23:amd64 (1.0.18-1ubuntu0.24.04.1, automatic), vim-runtime:amd64 (2:9.1.0016-1ubuntu7.20, automatic)
End-Date: 2026-10-12  12:31:50

Start-Date: 2026-10-12  12:41:30
Commandline: apt install iputils-ping -y
Requested-By: fousta (1000)
Install: iputils-ping:amd64 (3:20240117-1ubuntu0.1)
End-Date: 2026-10-12  12:41:30

Start-Date: 2026-10-12  15:35:22
Commandline: apt install language-pack-fr
Requested-By: fousta (1000)
Install: language-pack-fr:amd64 (1:24.04+20260905), locales:amd64 (2.39-0ubuntu8.9, automatic), language-pack-fr-base:amd64 (1:24.04+20260905, automatic)
End-Date: 2026-10-12  15:35:30

Start-Date: 2026-10-12  17:50:45
Commandline: apt -y upgrade
Requested-By: fousta (1000)
Upgrade: netplan-generator:amd64 (1.1.2-8ubuntu1~24.04.2, 1.1.2-8ubuntu1~24.04.3), krb5-locales:amd64 (1.20.1-6ubuntu2.8, 1.20.1-6ubuntu2.10), libgssapi-krb5-2:amd64 (1.20.1-6ubuntu2.8, 1.20.1-6ubuntu2.10), open-iscsi:amd64 (2.1.9-3ubuntu5.4, 2.1.9-3ubuntu5.5), libaudit-common:amd64 (1:3.1.2-2.1build1.1, 1:3.1.2-2.1ubuntu0.1), python3-netplan:amd64 (1.1.2-8ubuntu1~24.04.2, 1.1.2-8ubuntu1~24.04.3), apparmor:amd64 (4.0.1really4.0.1-0ubuntu0.24.04.7, 4.0.1really4.0.1-0ubuntu0.24.04.8), libapparmor1:amd64 (4.0.1really4.0.1-0ubuntu0.24.04.7, 4.0.1really4.0.1-0ubuntu0.24.04.8), libkrb5support0:amd64 (1.20.1-6ubuntu2.8, 1.20.1-6ubuntu2.10), libx11-data:amd64 (2:1.8.7-1build1, 2:1.8.7-1ubuntu0.1), libkrb5-3:amd64 (1.20.1-6ubuntu2.8, 1.20.1-6ubuntu2.10), libk5crypto3:amd64 (1.20.1-6ubuntu2.8, 1.20.1-6ubuntu2.10), libopeniscsiusr:amd64 (2.1.9-3ubuntu5.4, 2.1.9-3ubuntu5.5), netplan.io:amd64 (1.1.2-8ubuntu1~24.04.2, 1.1.2-8ubuntu1~24.04.3), libaudit1:amd64 (1:3.1.2-2.1build1.1, 1:3.1.2-2.1ubuntu0.1), libx11-6:amd64 (2:1.8.7-1build1, 2:1.8.7-1ubuntu0.1), libnetplan1:amd64 (1.1.2-8ubuntu1~24.04.2, 1.1.2-8ubuntu1~24.04.3)
End-Date: 2026-10-12  17:50:57
```
```text
[fousta@rhel9-lab ~]$ sudo dnf -y update
[sudo] Mot de passe de fousta:
Mise à jour des référentiels de gestion des abonnements.
Red Hat Enterprise Linux 9 for x86_64 - AppStream (RPMs)                                                                       49 MB/s | 114 MB     00:02
Dernière vérification de l’expiration des métadonnées effectuée il y a 0:00:02 le lun. 12 oct. 2026 17:52:41.
Dépendances résolues.
Rien à faire.
Terminé!
[fousta@rhel9-lab ~]$
[fousta@rhel9-lab ~]$ sudo dnf history | head -20
Mise à jour des référentiels de gestion des abonnements.
ID     | Ligne de commande                                                                                       | Date et heure    | Action(s)      | Modifié
--------------------------------------------------------------------------------------------------------------------------------------------------------------
     2 | install vim -y                                                                                          | 2026-10-12 14:07 | Install        |    4  <
     1 |                                                                                                         | 2026-10-12 13:19 | Install        |  394 >E
[fousta@rhel9-lab ~]$ sudo dnf repolist
Mise à jour des référentiels de gestion des abonnements.
id du dépôt                                                        nom du dépôt
rhel-9-for-x86_64-appstream-rpms                                   Red Hat Enterprise Linux 9 for x86_64 - AppStream (RPMs)
rhel-9-for-x86_64-baseos-rpms                                      Red Hat Enterprise Linux 9 for x86_64 - BaseOS (RPMs)
```

## 6. Comptes et accès distant

Comptes créés, mécanisme de délégation, configuration `sshd` retenue.

<!-- À compléter -->

## 7. Pare-feu

Outil, règles, méthode de vérification.

<!-- À compléter -->

## 8. Écarts entre les deux distributions

| Opération | RHEL 9 | Ubuntu 24.04 | Raison |
|---|---|---|---|
| … | … | … | … |

## 9. Incidents et corrections

<!-- À compléter -->

## 10. Reconstruction rapide

Séquence minimale de commandes pour reproduire une machine identique.

```bash
# À compléter
```
