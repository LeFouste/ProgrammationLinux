# Procédure d'installation rhel9-lab et ubuntu24-lab

## 1 Liste de contrôle matérielle

### 1.1 - La virtualisation matérielle est activée dans le micrologiciel du poste

Sur Windows : Win+R > cmd > systeminfo

Sortie :
[...]
Sécurité basée sur la virtualisation:   État : Non activé
					                              App Control for Business policy: Appliqué
					                              App Control for Business user mode policy: Audit
					                              Fonctionnalités de sécurité activées :

Configuration requise pour Hyper-V:     Extensions de mode du moniteur d’ordinateur virtuel : Oui
                                        Virtualisation activée dans le microprogramme : Oui
                                        Traduction d’adresse de second niveau : Oui
                                        Prévention de l’exécution des données disponible : Oui

Sur Linux : Terminal > lscpu | grep -i virtual


## 1. Périmetre
Distributions et versions exactes relevées sur les machines :

## 2. Ressources des machines virtuelles
| Parametre | rhel9-lab | ubuntu24-lab |
|---|---|---|

## 3. Partitionnement applique
| Volume | Taille obtenue | Point de montage | Systeme de fichiers |
Justification des choix :

## 4. Installation
Profil, nom d'hote, locale, fuseau horaire, horloge.

## 5. Mises a jour
Commandes executees et extrait d'historique horodate.

## 6. Comptes et acces distant
Comptes crees, mecanisme de delegation, configuration sshd retenue.

## 7. Pare-feu
Outil, regles, methode de verification.

## 8. Ecarts entre les deux distributions
| Operation | RHEL 9 | Ubuntu 24.04 | Raison |

## 9. Incidents et corrections

## 10. Reconstruction rapide
Sequence minimale de commandes pour reproduire une machine identique.
