# Journal de bord - TP 1 : Socle, installation et premier durcissement

- **Membres** : Kosta DJORDJEVIC, Arsel DIFFO, Mathis GERALDO
- **Machines** : `rhel9-lab`, `ubuntu24-lab`
- **Procédure reproductible** : `tp01/INSTALL.md`. **Sorties brutes** : `tp01/preuves/`.

---

## 1. Planning et temps passé

| Date | Créneau | Durée (Approximative) | Activité |
|---|---|---|---|
| Mardi 29/09 | 15h45 - 19h00 | 3 h 15 | Installation des VMs |
| Vendredi 02/10 | 08h00 - 13h00 | 5 h | Installation des VMs |
| Lundi 12/10 | 10h00 - 18h00 | 8 h | Récolte des informations, refonte propre des VMs, documentation sur GitHub |
| Mardi 13/10 | 10h00 - 18h00 | 8 h | Récolte des informations, refonte propre des VMs, documentation sur GitHub |
| Lundi 26/10 | 08h00 - 20h00 | 12 h | Durcissement, fin de `INSTALL.md`, journal et preuves |
| **Total** | | **36 h 15** | |

---

## 2. Datation des actions

Les actions faites sur les machines n'ont pas toutes été horodatées au moment où elles ont été réalisées. Les heures de ce journal sont donc de trois natures :

- **Relevées** : produites par une commande qui affiche la date (`date -Is`, `Get-Date`, `timedatectl`) ou par l'historique d'un gestionnaire de paquets (`apt`, `dnf`).
- **Notées** : issues de ma prise de note sur Discord au moment des manipulations.
- **Déduites** : retrouvées après coup dans une sortie (par exemple `chage -l` ou la ligne `Last login`).

Chaque ligne de la chronologie indique sa source. Quand l'heure n'est pas connue, c'est écrit. Aucune heure n'est estimée de tête.

---

## 3. Chronologie

### Mardi 29/09 et vendredi 02/10 - premières installations

Préparation et installation incomplète des machines Ubuntu et RHEL (virtualisation désactivée, partitionnement LVM ignoré). Les machines ont été refaites à neuf le 12/10.

---

### Lundi 12/10

Réinstallation des VMs à neuf, avec la virtualisation matérielle activée.

| Heure | événement | Source |
|---|---|---|
| avant 12:31 | Ubuntu : installation et partitionnement (heure exacte non relevée) | - |
| 12:31 | Ubuntu : `apt install vim` | `/var/log/apt/history.log` |
| 12:41 | Ubuntu : `apt install iputils-ping -y` | `/var/log/apt/history.log` |
| 13:19 | RHEL : installation initiale (394 paquets) | `dnf history`, ID 1 |
| 14:07 | RHEL : `dnf install vim -y` | `dnf history`, ID 2 |
| 14:34 | Réseau interne : 10.42.0.1 (RHEL) et 10.42.0.2 (Ubuntu) | note Discord |
| 14:42 | Test de communication entre les machines (ping 4/4) | note Discord |
| 15:35 | Ubuntu : `apt install language-pack-fr` | `/var/log/apt/history.log` |
| 17:48 | Fuseau `Europe/Paris` et horloge synchronisée (RHEL 17:48:07, Ubuntu 17:48:49) | `timedatectl status` |
| 17:50 | Ubuntu : `apt -y upgrade` | `/var/log/apt/history.log` |
| 17:52 | RHEL : `dnf -y update`, « Rien à faire » | sortie de la commande |

#### Réseau interne

Création du réseau interne `lab-interne` (préfixe `10.42.0.0/24`) et rattachement des deux machines à ce second réseau : RHEL en `10.42.0.1`, Ubuntu en `10.42.0.2`. Communication établie entre les deux machines.

**12/10/2026 14:34** (source : ma prise de note sur Discord)

```bash
[fousta@rhel9-lab ~]$ nmcli connection show
NAME   UUID                                  TYPE      DEVICE
ens33  a47dfbd2-782d-3cef-b263-ef0fedb8387e  ethernet  ens33
lo     aed37fd5-3816-456a-9f4d-14057d4aa11e  loopback  lo
ens34  6736e34d-c271-317a-b838-88eea7e79c63  ethernet  --
[fousta@rhel9-lab ~]$ sudo nmcli connection modify "ens34" ipv4.method manual ipv4.addresses "10.42.0.1/24"
[fousta@rhel9-lab ~]$ sudo nmcli connection up "ens34"
Connexion activée (chemin D-Bus actif : /org/freedesktop/NetworkManager/ActiveConnection/12)
[fousta@rhel9-lab ~]$ nmcli connection show
NAME   UUID                                  TYPE      DEVICE
ens33  a47dfbd2-782d-3cef-b263-ef0fedb8387e  ethernet  ens33
ens34  6736e34d-c271-317a-b838-88eea7e79c63  ethernet  ens34
lo     aed37fd5-3816-456a-9f4d-14057d4aa11e  loopback  lo
```
```bash
fousta@ubuntu24-lab:~$ sudo vim /etc/netplan/50-cloud-init.yaml
fousta@ubuntu24-lab:~$ sudo netplan apply
fousta@ubuntu24-lab:~$ ip a
[...]
2: ens33: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc fq_codel state UP group default qlen 1000
    link/ether 00:0c:29:8a:0e:df brd ff:ff:ff:ff:ff:ff
    altname enp2s1
    inet 192.168.224.131/24 metric 100 brd 192.168.224.255 scope global dynamic ens33
       valid_lft 1795sec preferred_lft 1795sec
    inet6 fe80::20c:29ff:fe8a:edf/64 scope link
       valid_lft forever preferred_lft forever
3: ens37: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc fq_codel state UP group default qlen 1000
    link/ether 00:0c:29:8a:0e:e9 brd ff:ff:ff:ff:ff:ff
    altname enp2s5
    inet 10.42.0.2/24 brd 10.42.0.255 scope global ens37
       valid_lft forever preferred_lft forever
    inet6 fe80::20c:29ff:fe8a:ee9/64 scope link
       valid_lft forever preferred_lft forever
```

**12/10/2026 14:42** (source : ma prise de note sur Discord)
```bash
[fousta@rhel9-lab ~]$ ping 10.42.0.2
PING 10.42.0.2 (10.42.0.2) 56(84) octets de données.
64 octets de 10.42.0.2 : icmp_seq=1 ttl=64 temps=0.474 ms
64 octets de 10.42.0.2 : icmp_seq=2 ttl=64 temps=0.807 ms
64 octets de 10.42.0.2 : icmp_seq=3 ttl=64 temps=0.859 ms
64 octets de 10.42.0.2 : icmp_seq=4 ttl=64 temps=0.843 ms

--- statistiques ping 10.42.0.2 ---
4 paquets transmis, 4 reçus, 0% packet loss, time 3015ms
rtt min/avg/max/mdev = 0.474/0.745/0.859/0.158 ms
--------------------------------------------------------------
fousta@ubuntu24-lab:~$ ping 10.42.0.1
PING 10.42.0.1 (10.42.0.1) 56(84) bytes of data.
64 bytes from 10.42.0.1: icmp_seq=1 ttl=64 time=1.62 ms
64 bytes from 10.42.0.1: icmp_seq=2 ttl=64 time=0.929 ms
64 bytes from 10.42.0.1: icmp_seq=3 ttl=64 time=0.924 ms
64 bytes from 10.42.0.1: icmp_seq=4 ttl=64 time=0.948 ms

--- 10.42.0.1 ping statistics ---
4 packets transmitted, 4 received, 0% packet loss, time 3006ms
rtt min/avg/max/mdev = 0.924/1.106/1.624/0.299 ms
```

#### Poste d'administration et images

Virtualisation matérielle, version de VMware Workstation, espace disque et mémoire du poste :

```text
Sécurité basée sur la virtualisation:       état : Non activé
                                            App Control for Business policy: Appliqué
                                            App Control for Business user mode policy: Audit
                                            Fonctionnalités de sécurité activées :
Configuration requise pour Hyper-V:         Extensions de mode du moniteur d’ordinateur virtuel : Oui
                                            Virtualisation activée dans le microprogramme : Oui
                                            Traduction d’adresse de second niveau : Oui
                                            Prévention de l’exécution des données disponible : Oui

reg query "HKLM\SOFTWARE\WOW6432Node\VMware, Inc.\VMware Workstation" /v ProductVersion
HKEY_LOCAL_MACHINE\SOFTWARE\WOW6432Node\VMware, Inc.\VMware Workstation
    ProductVersion    REG_SZ    17.5.2.23775571
```

```text
C:\Windows\System32>dir C:\
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
               9 Rép(s)  939 680 542 720 octets libres
```

```text
Mémoire physique totale:                    16 163 Mo
Mémoire physique disponible:                8 917 Mo
Mémoire virtuelle : taille maximale:        19 107 Mo
Mémoire virtuelle : disponible:             11 408 Mo
Mémoire virtuelle : en cours d’utilisation: 7 699 Mo
```

Vérification des images d'installation (SHA-256, comparé à la somme publiée par l'éditeur) :

```text
PS C:\ISO> Get-FileHash .\ubuntu-24.04.5-live-server-amd64.iso -Algorithm SHA256

Algorithm       Hash                                                                   Path
---------       ----                                                                   ----
SHA256          97F3D7FFB032C3EB3B23D2C8BE9CC76E60C2C1F2C0146BA5BA9FE01CAFAE0FD8       C:\ISO\ubuntu-24.04.5-live-se...

PS C:\ISO> Get-FileHash .\rhel-9.8-x86_64-boot.iso -Algorithm SHA256

Algorithm       Hash                                                                   Path
---------       ----                                                                   ----
SHA256          18FECEFE070E9C40502287281EC86E3BD4D64E2F48796A1CB166D9371308E5E6       C:\ISO\rhel-9.8-x86_64-boot.iso
```

Sources officielles :

- https://ubuntu.com/download/alternative-downloads
- https://developers.redhat.com/products/rhel/download#downloadsbyrelease

#### Versions relevées (premier relevé)

**ubuntu24-lab**

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
fousta@ubuntu24-lab:~$
```

**rhel9-lab**

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

#### Partitionnement obtenu

**ubuntu24-lab**

```text
fousta@ubuntu24-lab:~$ lsblk -f
NAME                FSTYPE      FSVER    LABEL UUID                                   FSAVAIL FSUSE% MOUNTPOINTS
sda
├─sda1
├─sda2              ext4        1.0            e60deb8c-2d32-4cbc-928e-ad58ccbd0eab    803.2M    11% /boot
└─sda3              LVM2_member LVM2 001       1KZp3m-8sjF-pbEJ-J27Y-P7WL-tk0B-03lxa3
  ├─vg_sys-lv--root ext4        1.0            7e63e773-2a12-4ab2-ad92-ab28add0d8e0      5.5G    24% /
  ├─vg_sys-lv--var  ext4        1.0            45465913-5f71-474c-86e6-aaddf7df6032      3.3G     9% /var
  ├─vg_sys-lv--log  ext4        1.0            925c178e-a917-4a1a-8026-77a3cadb51f2      1.8G     2% /var/log
  ├─vg_sys-lv--home ext4        1.0            3b55f634-bdf5-497a-99bd-3bed6ca3cad8      1.8G     0% /home
  └─vg_sys-lv--swap swap        1              39db11be-8dac-485a-ae3f-f66e2105ddcc                  [SWAP]
sr0
fousta@ubuntu24-lab:~$ df -hT
Filesystem                  Type   Size  Used Avail Use% Mounted on
tmpfs                       tmpfs  192M 1004K  192M   1% /run
/dev/mapper/vg_sys-lv--root ext4   7.8G  1.9G  5.5G  26% /
tmpfs                       tmpfs  960M     0  960M   0% /dev/shm
tmpfs                       tmpfs  5.0M     0  5.0M   0% /run/lock
/dev/mapper/vg_sys-lv--home ext4   2.0G   56K  1.8G   1% /home
/dev/mapper/vg_sys-lv--var  ext4   3.9G  358M  3.3G  10% /var
/dev/mapper/vg_sys-lv--log  ext4   2.0G   33M  1.8G   2% /var/log
/dev/sda2                   ext4   974M  103M  804M  12% /boot
tmpfs                       tmpfs  192M   12K  192M   1% /run/user/1000
tmpfs                       tmpfs  512M     0  512M   0% /tmp

fousta@ubuntu24-lab:~$ swapon --show
NAME      TYPE      SIZE USED PRIO
/dev/dm-4 partition   2G   0B   -2

fousta@ubuntu24-lab:~$ findmnt --target /var/log
TARGET   SOURCE                     FSTYPE OPTIONS
/var/log /dev/mapper/vg_sys-lv--log ext4   rw,relatime
```

**rhel9-lab**

```text
[fousta@rhel9-lab ~]$ lsblk -f
NAME                FSTYPE      FSVER    LABEL UUID                                   FSAVAIL FSUSE% MOUNTPOINTS
sda
├─sda1              xfs                        7c63443e-7a6b-4222-ae44-d94d1ab9525b    706,8M    26% /boot
└─sda2              LVM2_member LVM2 001       zZOoZP-jy4R-tWGw-iww2-dzn4-fCRd-tF6EpS
  ├─vg--sys-lv_root xfs                        7001df62-c1b0-4c8b-9557-fc6990a98c4d      8,3G    16% /
  ├─vg--sys-lv_swap swap        1              ee5c3775-8692-4aea-9d1f-82b82eb0b83a                  [SWAP]
  ├─vg--sys-lv_home xfs                        ef7a65e2-3634-4aeb-8164-61c483895dea      1,9G     2% /home
  ├─vg--sys-lv_log  xfs                        b7823846-7f56-44f1-bf03-c83c2f332642      2,9G     2% /var/log
  └─vg--sys-lv_var  xfs                        4a1888ec-cdfc-4cc7-9a6c-f976a734f3bc      4,4G    11% /var
sr0
[fousta@rhel9-lab ~]$ df -hT
Sys. de fichiers            Type     Taille Utilisé Dispo Uti% Monté sur
devtmpfs                    devtmpfs   815M       0  815M   0% /dev
tmpfs                       tmpfs      838M       0  838M   0% /dev/shm
tmpfs                       tmpfs      335M    5,0M  330M   2% /run
/dev/mapper/vg--sys-lv_root xfs         10G    1,7G  8,4G  17% /
/dev/sda1                   xfs        960M    254M  707M  27% /boot
/dev/mapper/vg--sys-lv_home xfs        2,0G     47M  1,9G   3% /home
/dev/mapper/vg--sys-lv_var  xfs        5,0G    575M  4,4G  12% /var
/dev/mapper/vg--sys-lv_log  xfs        3,0G     64M  2,9G   3% /var/log
tmpfs                       tmpfs      168M       0  168M   0% /run/user/1000

[fousta@rhel9-lab ~]$ swapon --show
NAME      TYPE      SIZE USED PRIO
/dev/dm-1 partition   2G   0B   -2
[fousta@rhel9-lab ~]$ findmnt --target /var/log
TARGET   SOURCE                     FSTYPE OPTIONS
/var/log /dev/mapper/vg--sys-lv_log xfs    rw,relatime,seclabel,attr2,inode64,logbufs=8,logbsize=32k,noquota
```

#### Relevé après redémarrage (Boot ID différent du premier relevé)

**ubuntu24-lab**

```text
fousta@ubuntu24-lab:~$ hostnamectl
 Static hostname: ubuntu24-lab
       Icon name: computer-vm
         Chassis: vm 🖴
      Machine ID: fbd006756f38402bbbde01d1b64dd65b
         Boot ID: adb3b19f47904eafa0f42677ab1de662
  Virtualization: vmware
Operating System: Ubuntu 24.04.5 LTS
          Kernel: Linux 6.8.0-146-generic
    Architecture: x86-64
 Hardware Vendor: VMware, Inc.
  Hardware Model: VMware Virtual Platform
Firmware Version: 6.00
   Firmware Date: Thu 2020-11-12
    Firmware Age: 5y 10month 4w 2d
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

fousta@ubuntu24-lab:~$ systemctl --failed
  UNIT LOAD ACTIVE SUB DESCRIPTION

0 loaded units listed.
fousta@ubuntu24-lab:~$ systemctl list-units --type=service --state=running | head -30
  UNIT                        LOAD   ACTIVE SUB     DESCRIPTION
  dbus.service                loaded active running D-Bus System Message Bus
  getty@tty1.service          loaded active running Getty on tty1
  multipathd.service          loaded active running Device-Mapper Multipath Device Controller
  polkit.service              loaded active running Authorization Manager
  ssh.service                 loaded active running OpenBSD Secure Shell server
  systemd-journald.service    loaded active running Journal Service
  systemd-logind.service      loaded active running User Login Management
  systemd-networkd.service    loaded active running Network Configuration
  systemd-resolved.service    loaded active running Network Name Resolution
  systemd-timesyncd.service   loaded active running Network Time Synchronization
  systemd-udevd.service       loaded active running Rule-based Manager for Device Events and Files
  unattended-upgrades.service loaded active running Unattended Upgrades Shutdown
  user@1000.service           loaded active running User Manager for UID 1000

Legend: LOAD   → Reflects whether the unit definition was properly loaded.
        ACTIVE → The high-level unit activation state, i.e. generalization of SUB.
        SUB    → The low-level unit activation state, values depend on unit type.

13 loaded units listed.
```

**rhel9-lab**

```text
[fousta@rhel9-lab ~]$ hostnamectl
 Static hostname: rhel9-lab
       Icon name: computer-vm
         Chassis: vm 🖴
      Machine ID: 9d0eca9bf6b2431aadcce38fad31f4a0
         Boot ID: c3ae2aaa0d25493ca367595a410b5f28
  Virtualization: vmware
Operating System: Red Hat Enterprise Linux 9.8 (Plow)
     CPE OS Name: cpe:/o:redhat:enterprise_linux:9::baseos
          Kernel: Linux 5.14.0-687.56.1.el9_8.x86_64
    Architecture: x86-64
 Hardware Vendor: VMware, Inc.
  Hardware Model: VMware Virtual Platform
Firmware Version: 6.00
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

[fousta@rhel9-lab ~]$ systemctl --failed
  UNIT LOAD ACTIVE SUB DESCRIPTION
0 loaded units listed.
[fousta@rhel9-lab ~]$ systemctl list-units --type=service --state=running | head -30
  UNIT                     LOAD   ACTIVE SUB     DESCRIPTION
  auditd.service           loaded active running Security Auditing Service
  chronyd.service          loaded active running NTP client/server
  crond.service            loaded active running Command Scheduler
  dbus-broker.service      loaded active running D-Bus System Message Bus
  firewalld.service        loaded active running firewalld - dynamic firewall daemon
  getty@tty1.service       loaded active running Getty on tty1
  irqbalance.service       loaded active running irqbalance daemon
  NetworkManager.service   loaded active running Network Manager
  rhsmcertd.service        loaded active running Enable periodic update of entitlement certificates.
  rsyslog.service          loaded active running System Logging Service
  sshd.service             loaded active running OpenSSH server daemon
  systemd-journald.service loaded active running Journal Service
  systemd-logind.service   loaded active running User Login Management
  systemd-udevd.service    loaded active running Rule-based Manager for Device Events and Files
  user@1000.service        loaded active running User Manager for UID 1000
  vgauthd.service          loaded active running VGAuth Service for open-vm-tools
  vmtoolsd.service         loaded active running Service for virtual machines hosted on VMware

LOAD   = Reflects whether the unit definition was properly loaded.
ACTIVE = The high-level unit activation state, i.e. generalization of SUB.
SUB    = The low-level unit activation state, values depend on unit type.
17 loaded units listed.
```

`systemctl --failed` : « 0 loaded units listed » sur les deux machines.

#### Locale, fuseau horaire et synchronisation de l'heure

**rhel9-lab**

```text
[fousta@rhel9-lab ~]$ localectl status
System Locale: LANG=fr_FR.UTF-8
    VC Keymap: fr-oss
   X11 Layout: fr
  X11 Variant: oss
[fousta@rhel9-lab ~]$ locale -a | grep -i fr_FR
fr_FR
fr_FR@euro
fr_FR.utf8
[fousta@rhel9-lab ~]$ timedatectl status
               Local time: lun. 2026-10-12 17:48:07 CEST
           Universal time: lun. 2026-10-12 15:48:07 UTC
                 RTC time: lun. 2026-10-12 15:48:08
                Time zone: Europe/Paris (CEST, +0200)
System clock synchronized: yes
              NTP service: active
          RTC in local TZ: no
```

**ubuntu24-lab**

```text
fousta@ubuntu24-lab:~$ localectl status
System Locale: LANG=C.UTF-8
    VC Keymap: (unset)
   X11 Layout: fr
    X11 Model: pc105
fousta@ubuntu24-lab:~$ locale -a | grep -i fr_FR
fr_FR.utf8
fousta@ubuntu24-lab:~$ timedatectl status
               Local time: Mon 2026-10-12 17:48:49 CEST
           Universal time: Mon 2026-10-12 15:48:49 UTC
                 RTC time: Mon 2026-10-12 15:48:49
                Time zone: Europe/Paris (CEST, +0200)
System clock synchronized: yes
              NTP service: active
          RTC in local TZ: no
```

Sur Ubuntu, la locale française est disponible (`fr_FR.utf8`) mais n'est pas définie comme locale système (`LANG=C.UTF-8`).

#### Mises à jour

**ubuntu24-lab** : historique `apt` (extrait)

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

**rhel9-lab** : `dnf -y update`, historique et dépôts

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

---

### Mardi 13/10

| Heure | événement | Source |
|---|---|---|
| 12:57 | RHEL : création des comptes `kosta`, `mathis`, `arsel` | `chage -l` : « Last password change : oct. 13, 2026 » |
| 12:57 | Verrouillage du compte root sur les deux machines | sorties de `passwd -S root` |
| 13:47 environ | RHEL : observation du groupe après reconnexion (session A du compte `arsel`, voir section 6) | ligne `Last login` de la session suivante |

#### Création de `/tmp` en tmpfs (Ubuntu)

```text
sudo echo 'tmpfs  /tmp  tmpfs  defaults,noatime,nosuid,nodev,size=512M  0  0' | sudo tee -a /etc/fstab
   19  sudo systemctl daemon-reload
   20  sudo mount /tmp
```
13/10 - 12:31

#### Verrouillage de l'accès au compte root

**ubuntu24-lab**

```text
fousta@ubuntu24-lab:~$ sudo passwd -l root
[sudo] password for fousta:
passwd: password changed.
fousta@ubuntu24-lab:~$ sudo passws -S root
sudo: passws: command not found
fousta@ubuntu24-lab:~$ sudo passwd -S root
root L 2026-09-09 0 99999 7 -1
```

**rhel9-lab**

```text
[fousta@rhel9-lab ~]$ sudo passwd -l root
[sudo] Mot de passe de fousta :
Désolé, essayez de nouveau.
[sudo] Mot de passe de fousta :
Verrouillage du mot de passe pour l'utilisateur root.
passwd: Succès
[fousta@rhel9-lab ~]$ sudo passwd -S root
root LK 1969-12-31 0 99999 7 -1 (Mot de passe verrouillé.)
```

#### Création des utilisateurs sur RHEL

```text
[fousta@rhel9-lab ~]$ sudo useradd -m -c "Kosta DJORDJEVIC" kosta
[fousta@rhel9-lab ~]$ sudo usermod -aG wheel kosta
[fousta@rhel9-lab ~]$ sudo chage -l kosta
Last password change                                    : oct. 13, 2026
Password expires                                        : never
Password inactive                                       : never
Account expires                                         : never
Minimum number of days between password change          : 0
Maximum number of days between password change          : 99999
Number of days of warning before password expires       : 7
[fousta@rhel9-lab ~]$ sudo useradd -m -c "Mathis GERALDO" mathis
sudo usermod -aG wheel mathis
sudo chage -l mathis
Last password change                                    : oct. 13, 2026
Password expires                                        : never
Password inactive                                       : never
Account expires                                         : never
Minimum number of days between password change          : 0
Maximum number of days between password change          : 99999
Number of days of warning before password expires       : 7
[fousta@rhel9-lab ~]$ sudo useradd -m -c "Arsel DIFFO" arsel
sudo usermod -aG wheel arsel
sudo chage -l arsel
Last password change                                    : oct. 13, 2026
Password expires                                        : never
Password inactive                                       : never
Account expires                                         : never
Minimum number of days between password change          : 0
Maximum number of days between password change          : 99999
Number of days of warning before password expires       : 7
```

---

### Lundi 26/10

| Heure | événement | Source |
|---|---|---|
| 10:10 | Ubuntu : relevé `sshd -T` avant durcissement | `date -Is` |
| entre 10:10 et 10:26 | Ubuntu : `PermitRootLogin no`, `sshd -t`, rechargement, test de refus de root (code 255) | ordre des commandes |
| 10:26 | Ubuntu : état initial `ufw`, règle `OpenSSH`, activation | `date -Is` |
| 10:28 | Test SSH depuis le poste après activation du pare-feu | `Get-Date -Format o` |

---

## 4. Justifications écrites

### A.4 - Instantané `avant-installation`

L'instantané fige l'état des machines avant toute écriture sur le disque : il permet de revenir à l'état voulu (ici à une machine vierge), sans refaire la création ni la configuration matérielle, si l'installation échoue ou si un réglage rend la machine inutilisable.

### B.1 - Schéma de partitionnement

**Pourquoi séparer `/var`, et surtout `/var/log`, du volume racine ?**  
Une partition dédiée pour /var/log est utile pour isoler l'espace occupé par les logs et prévenir une utilisation excessive de l'espace disque qui pourrait affecter d'autres parties du système. Par exemple un service qui journalise en boucle remplit /var/log ; sans volume séparé, la racine sature et la machine ne peut plus rien écrire.

**Pourquoi `/boot` reste-t-il une partition classique, hors du groupe de volumes ?**  
Dans ce cas précis, c'est utile de ne pas placer la partition /boot dans le lvm car la partition root peut avoir des caractéristiques qui ne la rendent pas bootable directement. GRUB doit lire le noyau avant que le lvm soit activé, donc une partition classique est plus simple et plus sûre.

**Pourquoi 2 Gio de mémoire d'échange sur 2 Gio de mémoire vive, plutôt que 8 Gio ou rien ?**  
Dans ce cas, prévoir beaucoup plus de swap que de RAM physique (2 Gio et 8 Gio) est une mauvaise idée dans le sens où l'espace dans lequel la RAM va puiser se situe sur le disque dur, qui a fatalement un temps d'accès à la ressource beaucoup plus lent de la RAM physique. Le système va passer plus de temps à déplacer des données qu'à travailler, et ne rien mettre non plus est une mauvaise idée, la mémoire swap sert de filet de sécurité, éviter un plantage du système (qui n'a plus assez de RAM pour opérer), et globalement son rôle va être d'absorber des pics d'utilisation de la RAM. 
Source : [documentation Red Hat 9 sur le swap](https://docs.redhat.com/en/documentation/red_hat_enterprise_linux/9/html/managing_storage_devices/getting-started-with-swap_managing-storage-devices).

**Pourquoi ne pas allouer la totalité du disque, et à quoi servira l'espace restant ?**  
L'espace libre du groupe de volumes permet d'agrandir un volume à chaud et de prendre des instantanés LVM, sans repartitionner. Il sera utilisé au TP 2.

**Quel volume agrandir en premier sur un serveur qui tourne, et quelle opération maîtriser ?**  
Le volume à agrandir en premier sur un serveur est le volume des logs (/var/log). Il s'agit d'un volume qui se remplit avec l'activité du serveur et si ce volume sature, le service qui journalise ne peut plus écrire, parfois s'arrête, et on perd la trace de ce qui s'est passé. D'où l'intérêt, d'ailleurs, de l'avoir isolé (on limite la panne à /var/log).  
L'opération à maîtriser s'appelle le *Online Resizing*. Ici on l'opère en deux temps, on agrandit d'abord le volume logique et ensuite le système de fichiers qu'il contient, sans démonter quoi que ce soit.

---

## 5. Erreurs de méthode et leçons

| # | Erreur | Conséquence | Leçon |
|---|---|---|---|
| 1 | Actions réalisées sans horodatage | Dates à reconstituer après coup, preuves moins solides | Lancer `date -Is` avant chaque série de commandes et coller la sortie dans `preuves/` |
| 2 | `vgrename` lancé sur RHEL dans une session ouverte, sans instantané récent | Déconnexion, blocage au login (détail : `INSTALL.md` §9, incident 1) | Instantané avant toute opération de stockage, et lister les références à l'ancien nom avant de renommer |
| 3 | Test de délégation du sujet pris tel quel | `sudo -n` échoue sans `NOPASSWD` (détail : `INSTALL.md` §9, incident 2) | Comprendre ce qu'une option teste avant de conclure à une panne |
| 4 | `usermod -aG sudo arsel` tapé sur RHEL (commande d'Ubuntu) | `group 'sudo' does not exist` : le groupe d'administration de RHEL s'appelle `wheel` | Vérifier le nom du groupe propre à la distribution avant de lancer la commande |

---

## 6. Observation : le groupe n'est effectif qu'après reconnexion

Observation réalisée sur `rhel9-lab` avec le compte `arsel`, le mardi 13/10. La session A a été ouverte à 13:47:36 (d'après la ligne `Last login` affichée à la reconnexion). Les blocs ci-dessous suivent l'ordre de la procédure.

**Procédure**

1. Retirer le compte du groupe : `sudo gpasswd -d arsel wheel` (RHEL) ou `sudo gpasswd -d arsel sudo` (Ubuntu).
2. Ouvrir une session A avec ce compte : `id` ne montre pas le groupe, `sudo whoami` est refusé.
3. Depuis un compte administrateur, remettre le compte dans le groupe (`usermod -aG`), puis `id arsel` : le groupe apparaît.
4. Dans la session A, sans la fermer : `id` ne montre toujours pas le groupe, `sudo whoami` est toujours refusé.
5. Se déconnecter puis se reconnecter : `id` montre le groupe, `sudo whoami` répond `root`.

**Résultat observé**

étape 1 : retrait du groupe (compte administrateur `fousta`)

```text
[fousta@rhel9-lab ~]$ sudo gpasswd -d arsel wheel
[sudo] Mot de passe de fousta :
Removing user arsel from group wheel
```

étape 2 : ouverture de la session A (compte `arsel`)

```text
ssh arsel@192.168.224.132
arsel@192.168.224.132's password:
[arsel@rhel9-lab ~]$
[arsel@rhel9-lab ~]$ id
uid=1003(arsel) gid=1003(arsel) groupes=1003(arsel) contexte=unconfined_u:unconfined_r:unconfined_t:s0-s0:c0.c1023
[arsel@rhel9-lab ~]$ sudo whoami

Nous espérons que vous avez reçu de votre administrateur système local
les consignes traditionnelles. Généralement, elles se concentrent sur ces trois éléments :

    #1) Respectez la vie privée des autres.
    #2) Réfléchissez avant d'utiliser le clavier.
    #3) De grands pouvoirs confèrent de grandes responsabilités.

Pour des raisons de sécurité, le mot de passe que vous tapez ne sera pas visible.

[sudo] Mot de passe de arsel :
arsel n'est pas dans le fichier sudoers.
```

étape 3 : remise dans le groupe (compte administrateur `fousta`). La première commande échoue car j'ai tapé la commande d'Ubuntu (`sudo`) sur RHEL, où le groupe s'appelle `wheel`.

```text
[fousta@rhel9-lab ~]$ sudo usermod -aG sudo arsel
usermod: group 'sudo' does not exist
[fousta@rhel9-lab ~]$ sudo usermod -aG wheel arsel
[fousta@rhel9-lab ~]$ id arsel
uid=1003(arsel) gid=1003(arsel) groupes=1003(arsel),10(wheel)
```

étape 4 : retour dans la session A, sans l'avoir fermée

```text
[arsel@rhel9-lab ~]$ id
uid=1003(arsel) gid=1003(arsel) groupes=1003(arsel) contexte=unconfined_u:unconfined_r:unconfined_t:s0-s0:c0.c1023
[arsel@rhel9-lab ~]$ sudo whoami
[sudo] Mot de passe de arsel :
arsel n'est pas dans le fichier sudoers.
```

étape 5 : déconnexion puis reconnexion

```text
[arsel@rhel9-lab ~]$ exit
déconnexion
Connection to 192.168.224.132 closed.
ssh arsel@192.168.224.132
arsel@192.168.224.132's password:
X11 forwarding request failed on channel 0
Last login: Tue Oct 13 13:47:36 2026 from 192.168.224.1
[arsel@rhel9-lab ~]$ id
uid=1003(arsel) gid=1003(arsel) groupes=1003(arsel),10(wheel) contexte=unconfined_u:unconfined_r:unconfined_t:s0-s0:c0.c1023
[arsel@rhel9-lab ~]$ sudo whoami
[sudo] Mot de passe de arsel :
root
```

**Conclusion** : `id` n'affiche le groupe `wheel` qu'après reconnexion, et `sudo` reste refusé (« n'est pas dans le fichier sudoers ») tant que la session n'a pas été renouvelée. Observation réalisée sur RHEL uniquement.

