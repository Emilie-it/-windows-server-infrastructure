# Infrastructure Windows Server

## Présentation

Ce dépôt rassemble mes travaux de déploiement
et de configuration d’un environnement Windows Server.

J’y présente les services installés, les configurations réalisées,
les difficultés rencontrées et les vérifications effectuées.

## Environnement technique

Les manipulations sont réalisées sur des machines virtuelles,
avec Windows Server 2025 et un poste client Windows 11.

La configuration des machines et du réseau est précisée
dans chaque compte rendu pour distinguer les différentes étapes.

## Objectifs

- Déployer un domaine Active Directory.
- Administrer les utilisateurs, les groupes et les ordinateurs.
- Configurer les services DNS et DHCP.
- Appliquer des paramètres avec les GPO.
- Organiser les accès aux dossiers partagés.
- Administrer un serveur sans interface graphique.
- Mettre en place la redondance de certains services.

## Travaux présentés

| Projet | Contenu |
|---|---|
| [Active Directory et DNS](active-directory/README.md) | Déploiement du domaine et jonction d’un poste Windows 11 |
| [GPO](gpo/README.md) | Configuration des stratégies, ciblage et vérification sur les postes clients |
| Partages et permissions | Partages SMB, droits NTFS et méthode AGDLP |
| Stockage et quotas | Manipulations avec `fsutil` et gestion des quotas |
| Server Core | Configuration et administration en PowerShell |
| Second contrôleur de domaine | Ajout d’un contrôleur et vérification de la réplication |
| Rôles FSMO | Identification et transfert des rôles |
| DHCP | Configuration et basculement entre deux serveurs |

Les comptes rendus sont ajoutés progressivement.

## Démarche

Pour chaque projet, je présente :

- l’objectif et l’environnement utilisé ;
- les principales configurations ;
- les vérifications et les résultats ;
- les difficultés rencontrées et leur résolution.

Je distingue les configurations prévues, les actions réalisées
et les résultats observés.

## Compétences travaillées

- Administration Windows Server.
- Gestion des identités et des accès.
- Configuration des services réseau.
- Administration en interface graphique et en PowerShell.
- Diagnostic et documentation technique.
