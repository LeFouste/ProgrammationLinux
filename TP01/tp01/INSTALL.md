# Procédure d'installation rhel9-lab et ubuntu24-lab

## 0. Liste de contrôle matérielle

### 0.1 Virtualisation matérielle activée dans le micrologiciel du poste

**Windows** : `Win+R` > `cmd` > `systeminfo`

Sortie (extrait) :

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

Espace libre sur `C:` : **939 680 542 720 octets (~ 875 Gio)**.

### 0.4 Mémoire disponible pour deux VM en parallèle

```cmd
systeminfo
```

Sortie (extrait) :

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
- Résultat : identique.

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

Distributions et versions exactes relevées sur les machines :

<!-- À compléter -->

## 2. Ressources des machines virtuelles

| Paramètre | rhel9-lab | ubuntu24-lab |
|---|---|---|
| … | … | … |

## 3. Partitionnement appliqué

| Volume | Taille obtenue | Point de montage | Système de fichiers |
|---|---|---|---|
| … | … | … | … |

**Justification des choix :**

<!-- À compléter -->

## 4. Installation

Profil, nom d'hôte, locale, fuseau horaire, horloge.

<!-- À compléter -->

## 5. Mises à jour

Commandes exécutées et extrait d'historique horodaté.

<!-- À compléter -->

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
