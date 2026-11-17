# Windows Server — Active Directory et DNS

## Présentation

Je déploie un environnement Windows Server avec un domaine
Active Directory et un poste client Windows 11.

Je prépare les machines virtuelles, configure leur réseau
et mets en place les services nécessaires au domaine.

Cette installation sert ensuite de base aux configurations
des utilisateurs, des groupes, des GPO et des partages.

## Environnement technique

| Élément | Configuration |
|---|---|
| Hyperviseur | VMware Workstation |
| Serveur | Windows Server 2025 avec interface graphique |
| Nom du serveur | `SRV-WIN-EBS-01` |
| Poste client | Windows 11 Pro |
| Domaine Active Directory | `TSSR-EBS.LAB` |
| Réseau virtuel | LAN Segment `LAB-TSSR-EBS` |
| Adresse du serveur | `192.168.100.10/24` |
| Adresse du client | `192.168.100.100/24` |
| DNS du client | `192.168.100.10` |
| Démarrage du serveur | UEFI avec Secure Boot |
| Ressources du serveur | 2 processeurs virtuels, 4 Go de RAM, disque de 64 Go |

Les adresses présentées correspondent à cette étape
de l’installation, avant les travaux sur DHCP.

## Objectifs

- Installer et préparer Windows Server.
- Configurer un réseau virtuel isolé.
- Déployer un domaine Active Directory avec DNS.
- Joindre un poste Windows 11 au domaine.
- Préparer l’environnement pour la gestion des utilisateurs
  et des stratégies de groupe.

## 1. Préparation des machines virtuelles

Je crée la machine Windows Server avec une installation
personnalisée dans VMware.

Je choisis d’installer le système ultérieurement pour garder
la main sur les paramètres de l’installation.

Je configure les ressources, le démarrage UEFI et le disque,
puis je démarre sur l’image ISO.

Après l’installation, je renomme le serveur
`SRV-WIN-EBS-01` et j’installe VMware Tools.

Je prépare également un poste client Windows 11 Pro.

## 2. Configuration du réseau

Je connecte les deux machines au même LAN Segment :
`LAB-TSSR-EBS`.

Ce réseau permet aux machines virtuelles de communiquer
entre elles. Il ne fournit pas automatiquement de serveur DHCP
ni d’accès à Internet.

Je configure donc les adresses IPv4 manuellement.

| Paramètre | Serveur | Client |
|---|---|---|
| Adresse IPv4 | `192.168.100.10` | `192.168.100.100` |
| Masque | `255.255.255.0` | `255.255.255.0` |
| DNS | Service DNS du domaine | `192.168.100.10` |

Pour communiquer entre ces deux machines dans le même
sous-réseau, une passerelle n’est pas nécessaire.

## 3. Déploiement d’Active Directory et DNS

J’installe le rôle AD DS, puis je promeus le serveur
en contrôleur de domaine pour créer la forêt `TSSR-EBS.LAB`.

AD DS permet de centraliser les comptes utilisateurs,
les ordinateurs et les groupes dans un annuaire.

Le service DNS permet notamment aux machines de trouver
les services du domaine.

L’installation du rôle et la promotion sont deux étapes
différentes : installer AD DS ne suffit pas à transformer
le serveur en contrôleur de domaine.

## 4. Jonction du poste client

Je configure le client pour qu’il utilise le serveur DNS
du domaine à l’adresse `192.168.100.10`.

Je joins ensuite Windows 11 Pro au domaine `TSSR-EBS.LAB`,
puis je redémarre le poste.

Le client devient membre du domaine. Il peut alors être
administré dans ce cadre et recevoir les GPO qui le concernent.

## 5. Organisation de l’annuaire

L’environnement est ensuite utilisé pour créer
les utilisateurs, les groupes et les unités d’organisation.

Les unités d’organisation, ou OU, permettent de ranger
les objets de l’annuaire et de définir le périmètre
d’application des GPO.

Les groupes servent notamment à organiser les accès
aux ressources.

Les configurations détaillées sont présentées
dans les pages consacrées aux GPO et aux partages.

## 6. Difficulté rencontrée : communication entre les VM

Lors des premiers essais, la communication entre
le serveur et le client ne fonctionne pas.

La vérification réalisée avec le formateur montre
que les deux machines ne sont pas démarrées simultanément.

Je reprends les essais en m’assurant que les deux VM
sont allumées et connectées au même LAN Segment.

Cette situation m’a rappelé de commencer par les vérifications
simples : état des machines, connexion des cartes réseau
et réseau virtuel utilisé.

Le pare-feu doit également être pris en compte pour les tests
ping : l’absence de réponse ne prouve pas, à elle seule,
que toute communication réseau est impossible.

## 7. Vérifications

Les contrôles portent sur plusieurs niveaux :

| Contrôle | Ce que je vérifie |
|---|---|
| Configuration IP | Adresse, masque et DNS de chaque machine |
| Réseau VMware | Connexion des deux VM au même LAN Segment |
| Communication | Accessibilité entre les machines, selon les règles du pare-feu |
| DNS | Résolution des noms et recherche des services du domaine |
| Domaine | Présence du poste client dans Active Directory |
| Session | Ouverture d’une session avec un compte du domaine |

Ces contrôles permettent de distinguer un problème réseau,
un problème DNS et un problème de jonction au domaine.

## Résultat

Le domaine `TSSR-EBS.LAB` est déployé et le poste Windows 11
est intégré à l’environnement.

Cette base est ensuite utilisée pour les travaux sur les GPO,
les partages, les permissions et l’ajout d’un second
contrôleur de domaine.

## Compétences mobilisées

- Création et configuration de machines virtuelles.
- Installation de Windows Server.
- Configuration IPv4 et DNS.
- Déploiement d’Active Directory.
- Jonction d’un poste Windows au domaine.
- Vérification de la communication dans un réseau virtuel.

## Références

- [Documentation Windows Server](https://learn.microsoft.com/fr-fr/windows-server/)
- [Présentation d’Active Directory Domain Services](https://learn.microsoft.com/fr-fr/windows-server/identity/ad-ds/get-started/virtual-dc/active-directory-domain-services-overview)
