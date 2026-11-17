# Windows Server — Partages SMB et permissions NTFS

## Présentation

Je crée des dossiers partagés pour plusieurs services
et je configure leurs accès avec la méthode AGDLP.

Je distingue les autorisations de partage réseau
des permissions NTFS, puis je teste les accès
depuis un poste Windows 11.

## Environnement technique

| Élément | Configuration |
|---|---|
| Hyperviseur | Proxmox VE |
| Serveur | Windows Server 2025 |
| Nom du serveur | `SRV-WIN-EBS-01` |
| Poste client | Windows 11 Pro |
| Domaine | `TSSR-EBS.LAB` |
| Adresse du serveur | `192.168.100.10` |
| Réseau | `192.168.100.0/24` |
| Dossier partagé | `C:\PARTAGE` |
| Accès réseau | `\\SRV-WIN-EBS-01\PARTAGE` |

## Objectifs

- Créer les dossiers des différents services.
- Les rendre accessibles sur le réseau avec SMB.
- Organiser les groupes selon la méthode AGDLP.
- Attribuer les permissions NTFS aux groupes.
- Tester les accès avec les comptes utilisateurs concernés.

## 1. Préparation des dossiers

Je crée le dossier `C:\PARTAGE` sur le serveur,
puis un sous-dossier pour chaque service :

- ADMINISTRATIF ;
- DIRECTION ;
- COMPTABILITE ;
- INFORMATIQUE ;
- RH ;
- PRODUCTION.

J’ajoute des fichiers de test dans les dossiers
pour vérifier la lecture et la modification
depuis le poste client.

## 2. Configuration du partage réseau

Dans les propriétés de `C:\PARTAGE`,
j’ouvre le partage avancé et je partage le dossier.

Le chemin réseau utilisé est :

`\\SRV-WIN-EBS-01\PARTAGE`

SMB est le protocole utilisé pour accéder
aux fichiers partagés sur le réseau.

Pour cette configuration, j’attribue « Contrôle total »
au groupe « Tout le monde » dans les autorisations
du partage. Je définis ensuite les restrictions
dans les permissions NTFS.

Cette autorisation de partage ne donne pas, à elle seule,
un contrôle total sur les fichiers : les permissions NTFS
restent également prises en compte.

## 3. Organisation des groupes avec AGDLP

J’utilise la méthode AGDLP pour attribuer les droits
par groupes plutôt que directement aux utilisateurs.

| Élément | Signification | Utilisation |
|---|---|---|
| A | Accounts — comptes | Comptes des utilisateurs |
| G | Global groups — groupes globaux | Regroupement des utilisateurs par service |
| DL | Domain Local groups — groupes de domaine local | Regroupement selon l’accès à une ressource |
| P | Permissions | Droits attribués sur le dossier |

Les comptes utilisateurs sont membres des groupes globaux.
Les groupes globaux sont ajoutés aux groupes de domaine local.
Les permissions sont attribuées à ces derniers.

Par exemple, pour le service informatique :

- les utilisateurs appartiennent à `GG-INFORMATIQUE` ;
- ce groupe est membre de `GDL-INFORMATIQUE-RW` ;
- `GDL-INFORMATIQUE-RW` reçoit le droit de modification
  sur le dossier INFORMATIQUE.

Les OU permettent de ranger les objets dans l’annuaire.
Elles n’attribuent pas les permissions sur les dossiers.

## 4. Attribution des permissions NTFS

Dans les propriétés des dossiers, j’ouvre
Sécurité → Avancé.

J’ajoute les groupes de domaine local concernés
et je leur attribue les permissions nécessaires.

| Suffixe du groupe | Utilisation | Permission NTFS retenue |
|---|---|---|
| RO — Read Only | Consulter le contenu | Lecture et exécution |
| RW — Read/Write | Travailler sur les fichiers | Modification |

La permission « Modification » permet notamment
de créer, modifier et supprimer des fichiers.
Elle ne correspond pas au « Contrôle total ».

J’applique les autorisations au dossier,
à ses sous-dossiers et à ses fichiers.

Je vérifie également les permissions héritées :
ajouter un groupe ne retire pas les accès
déjà accordés par d’autres entrées.

## 5. Matrice des accès

Chaque service dispose du droit de modification
sur son propre dossier.

Des accès en lecture sont ajoutés sur certains
autres dossiers selon les besoins définis.

| Service utilisateur | Modification | Lecture seule sur les autres dossiers |
|---|---|---|
| Administratif | ADMINISTRATIF | COMPTABILITE, RH |
| Direction | DIRECTION | ADMINISTRATIF, COMPTABILITE, RH, PRODUCTION |
| Comptabilité | COMPTABILITE | Aucun accès supplémentaire prévu |
| Informatique | INFORMATIQUE | PRODUCTION |
| Ressources humaines | RH | Aucun accès supplémentaire prévu |
| Production | PRODUCTION | Aucun accès supplémentaire prévu |

Cette matrice concerne les comptes utilisateurs des services.
Les accès nécessaires à l’administration du serveur
sont traités séparément.

## 6. Vérifications depuis le client

Sur Windows 11, j’ouvre une session avec le compte
du service à tester.

Depuis l’Explorateur de fichiers, j’accède au partage :

`\\SRV-WIN-EBS-01\PARTAGE`

Je vérifie les accès aux dossiers et le comportement
sur les fichiers.

| Accès prévu | Test |
|---|---|
| Lecture | Ouvrir et consulter un fichier |
| Modification | Modifier puis enregistrer un fichier |
| Lecture seule | Vérifier que la modification n’est pas autorisée |
| Aucun accès | Vérifier que l’accès au contenu est refusé |

Après une modification des groupes, une nouvelle ouverture
de session peut être nécessaire pour prendre en compte
les nouvelles appartenances.

## 7. Résultats

Les tests consignés dans mon compte rendu confirment
les accès en modification sur les dossiers des services
et les accès en lecture définis dans la matrice.

J’ai notamment vérifié :

- la modification de fichiers par le service Comptabilité ;
- les accès en lecture de la Direction aux dossiers prévus ;
- les accès du service Informatique à son dossier
  et au dossier Production.

## Résultat

Les dossiers sont accessibles sur le réseau
et les droits sont organisés avec des groupes AD.

Pour modifier l’accès d’un utilisateur, je peux agir
sur son appartenance aux groupes sans lui attribuer
directement des permissions sur chaque dossier.

## Compétences mobilisées

- Création de partages SMB.
- Gestion des groupes de sécurité Active Directory.
- Application de la méthode AGDLP.
- Configuration des permissions NTFS.
- Vérification des accès depuis un poste client.
- Organisation des droits selon les besoins des services.

## Références

- [Documentation Microsoft sur les groupes de sécurité Active Directory](https://learn.microsoft.com/fr-fr/windows-server/identity/ad-ds/manage/understand-security-groups)
- [Documentation Microsoft sur le contrôle d’accès](https://learn.microsoft.com/fr-fr/windows-server/security/windows-authentication/access-control)
