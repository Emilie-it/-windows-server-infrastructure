# Windows Server Core — Installation et administration

## Présentation

J’installe Windows Server 2025 avec l’option Server Core,
puis je configure le serveur avec SConfig et PowerShell.

Je mets également en place son administration à distance
depuis un poste Windows 11 avec Windows Admin Center.

Cette page présente la préparation du serveur avant
sa promotion en contrôleur de domaine supplémentaire.

## Environnement technique

| Élément | Configuration |
|---|---|
| Hyperviseur | VMware Workstation |
| Système | Windows Server 2025 Standard Evaluation — Server Core |
| Nom du serveur | `SRV-WIN-EBS-02` |
| Processeurs virtuels | 2 |
| Mémoire vive | 2 Go |
| Disque virtuel | 64 Go |
| Démarrage | UEFI |
| Réseau de préparation | NAT pour les mises à jour |
| Réseau du laboratoire | LAN Segment `LAB-TSSR-EBS` |
| Adresse IPv4 finale | `192.168.100.11/24` |
| Serveur DNS configuré | `192.168.100.10` |
| Poste d’administration | Windows 11 Pro avec Windows Admin Center |

Le passage du NAT au LAN Segment correspond
à deux étapes différentes de la préparation.

## Objectifs

- Installer Windows Server sans environnement de bureau complet.
- Utiliser SConfig et PowerShell pour configurer le serveur.
- Installer VMware Tools et les mises à jour.
- Configurer le nom, l’adresse IPv4 et le DNS.
- Créer un compte administrateur local.
- Administrer le serveur à distance avec Windows Admin Center.

## 1. Installation de Server Core

Je crée une machine virtuelle dans VMware Workstation,
puis je démarre sur l’image ISO Windows Server.

Je sélectionne l’édition sans la mention
« Expérience utilisateur ».

Server Core est une option d’installation de Windows Server.
Ce n’est pas un mode que j’active après avoir installé
la version avec interface graphique.

Après l’installation, je définis le mot de passe
du compte Administrateur.

J’utilise ensuite SConfig et PowerShell
pour préparer le serveur.

## 2. Installation de VMware Tools

Depuis VMware Workstation, je connecte l’image
d’installation de VMware Tools à la machine virtuelle.

Dans PowerShell, je me place sur le lecteur correspondant,
je consulte son contenu et je lance le programme
d’installation.

Je redémarre ensuite le serveur.

Je vérifie notamment le fonctionnement du copier-coller
entre l’hôte et la machine virtuelle.

## 3. Installation des mises à jour

Pendant la préparation, j’utilise une connexion NAT
pour accéder à Internet.

Je vérifie la connectivité, puis je recherche
et installe les mises à jour depuis SConfig.

Après le redémarrage, je consulte les correctifs
avec les outils PowerShell.

Les mises à jour permettent d’appliquer les correctifs
de sécurité et de fonctionnement avant de poursuivre
la configuration du serveur.

## 4. Configuration réseau avec PowerShell

Je commence par identifier les interfaces réseau
et leur configuration.

| Commande | Utilisation |
|---|---|
| `Get-NetAdapter` | Identifier les cartes réseau et leur index |
| `Get-NetIPAddress -AddressFamily IPv4` | Afficher les adresses IPv4 |
| `Get-DnsClientServerAddress` | Consulter les serveurs DNS configurés |

Je configure ensuite l’adresse fixe et le DNS
sur l’interface concernée.

| Paramètre | Valeur |
|---|---|
| Adresse IPv4 | `192.168.100.11` |
| Préfixe | `/24` |
| Masque correspondant | `255.255.255.0` |
| DNS | `192.168.100.10` |
| Passerelle sur le LAN Segment | Aucune |

J’utilise `New-NetIPAddress` pour attribuer l’adresse
et `Set-DnsClientServerAddress` pour configurer le DNS.

L’index d’interface utilisé dépend de la machine :
je l’identifie avant d’appliquer les commandes.

Je vérifie ensuite le résultat avec `ipconfig /all`.

## 5. Nom du serveur et compte local

Je renomme le serveur `SRV-WIN-EBS-02`
avec `Rename-Computer`, puis je le redémarre.

Je crée également un compte local et je l’ajoute
au groupe local Administrateurs.

| Commande | Utilisation |
|---|---|
| `New-LocalUser` | Créer le compte local |
| `Get-LocalUser` | Vérifier sa présence |
| `Get-LocalGroup` | Afficher les groupes locaux |
| `Add-LocalGroupMember` | Ajouter le compte à un groupe |
| `Get-LocalGroupMember` | Vérifier les membres du groupe |

Ce compte sert aux manipulations d’administration.
Les droits administrateur doivent rester réservés
aux comptes qui en ont besoin.

## 6. Vérification du pare-feu

Je consulte le profil réseau actif et l’état du pare-feu
avec `Get-NetConnectionProfile` et `Get-NetFirewallProfile`.

Je configure l’autorisation des requêtes d’écho ICMPv4
pour les tests ping, puis je teste depuis
les autres machines du laboratoire.

Une réponse au ping valide ce test de communication.
Elle ne suffit pas à confirmer le fonctionnement
de l’administration distante.

Je conserve le pare-feu actif et travaille
sur les règles nécessaires.

## 7. Administration avec Windows Admin Center

Depuis le poste Windows 11, j’ajoute le serveur
dans Windows Admin Center avec son adresse IP :

`192.168.100.11`

À cette étape, la connexion est réalisée avec
un compte administrateur local du serveur Core.

Windows Admin Center me permet de consulter
et d’administrer le serveur depuis une interface web,
sans installer un bureau complet sur Server Core.

## 8. Difficulté rencontrée : connexion distante

Lors de l’ajout du serveur, je rencontre une erreur
liée aux hôtes approuvés de WinRM, appelés TrustedHosts.

Je configure ce paramètre côté client WinRM
pour permettre la connexion dans le contexte utilisé,
puis je reprends l’authentification.

L’accès depuis Windows Admin Center fonctionne ensuite.

TrustedHosts ne donne pas les droits administrateur
sur le serveur. Le compte utilisé doit toujours
disposer des autorisations nécessaires.

Ce paramètre concerne la confiance accordée à la cible :
sa configuration doit rester limitée aux besoins.

## 9. Vérifications et résultats

| Contrôle | Résultat |
|---|---|
| Installation de Server Core | Serveur opérationnel |
| VMware Tools | Fonctionnement vérifié |
| Mises à jour | Mises à jour applicables installées |
| Nom du serveur | `SRV-WIN-EBS-02` |
| Adresse IPv4 et DNS | Configuration vérifiée |
| Compte administrateur local | Présent et fonctionnel |
| Tests ICMP | Réponses depuis les machines du laboratoire |
| Windows Admin Center | Connexion et administration fonctionnelles |

Depuis Windows Admin Center, je consulte également
les ressources CPU et mémoire, le réseau,
les services et les règles du pare-feu.

## Résultat

Le serveur Core est installé, configuré
et accessible depuis Windows Admin Center.

Je peux l’administrer localement en PowerShell
et à distance depuis le poste Windows 11.

Il est ensuite utilisé pour les travaux
sur le second contrôleur de domaine.

## Compétences mobilisées

- Installation de Windows Server Core.
- Utilisation de SConfig et PowerShell.
- Configuration IPv4 et DNS.
- Gestion des comptes et groupes locaux.
- Vérification du pare-feu.
- Administration distante avec Windows Admin Center.
- Résolution d’un problème de connexion WinRM.

## Références

- [Documentation Microsoft sur Server Core](https://learn.microsoft.com/fr-fr/windows-server/administration/server-core/what-is-server-core)
- [Documentation Windows Admin Center](https://learn.microsoft.com/fr-fr/windows-server/manage/windows-admin-center/overview)
