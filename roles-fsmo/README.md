# Active Directory — Gestion et transfert des rôles FSMO

## Présentation

Je travaille sur les cinq rôles FSMO d’Active Directory.

J’identifie leurs détenteurs, puis je réalise des transferts
entre les deux contrôleurs du domaine, avec les consoles
graphiques et PowerShell.

## Environnement technique

| Élément | Configuration |
|---|---|
| Hyperviseur | VMware Workstation |
| Domaine | `TSSR-EBS.LAB` |
| Premier contrôleur | `SRV-WIN-EBS-01` |
| Second contrôleur | `SRV-WIN-EBS-02` — Server Core |
| Administration | Consoles Active Directory et PowerShell |

## Objectifs

- Identifier les cinq rôles FSMO.
- Comprendre leur fonction.
- Vérifier leurs détenteurs.
- Transférer les rôles entre les deux contrôleurs.
- Contrôler le fonctionnement du domaine après les transferts.

## 1. Identification des rôles

Les rôles FSMO correspondent à certaines opérations
qui doivent être assurées par un détenteur unique
dans le domaine ou dans la forêt.

| Rôle | Portée | Fonction principale |
|---|---|---|
| Maître de schéma | Forêt | Gérer les modifications du schéma de l’annuaire |
| Maître d’attribution des noms de domaine | Forêt | Gérer notamment l’ajout et la suppression de domaines |
| Maître RID | Domaine | Distribuer les blocs de RID nécessaires aux identifiants des objets de sécurité |
| Émulateur PDC | Domaine | Participer notamment à la gestion des mots de passe et à la hiérarchie de synchronisation de l’heure |
| Maître d’infrastructure | Domaine | Mettre à jour certaines références à des objets d’autres domaines |

Un contrôleur peut détenir plusieurs rôles.

## 2. Vérifications avant le transfert

Avant de déplacer les rôles, je vérifie
que les deux contrôleurs sont disponibles
et que la réplication fonctionne.

J’identifie également le détenteur de chaque rôle.

| Outil | Utilisation |
|---|---|
| `netdom query fsmo` | Afficher les détenteurs des cinq rôles |
| `Get-ADDomain` | Consulter les détenteurs des rôles du domaine |
| `Get-ADForest` | Consulter les détenteurs des rôles de la forêt |
| `repadmin /replsummary` | Vérifier l’état de la réplication |

## 3. Transfert avec les consoles graphiques

Je transfère les rôles vers `SRV-WIN-EBS-02`
depuis les consoles adaptées.

| Console | Rôles concernés |
|---|---|
| Utilisateurs et ordinateurs Active Directory | RID, PDC et infrastructure |
| Domaines et approbations Active Directory | Attribution des noms de domaine |
| Schéma Active Directory | Schéma |

Dans chaque console, je sélectionne le contrôleur
de destination avant de réaliser le transfert.

Server Core peut recevoir ces rôles même s’il ne dispose
pas d’un environnement de bureau complet.
L’administration graphique se fait depuis une autre machine.

## 4. Transfert avec PowerShell

Je réalise ensuite le transfert des rôles
vers `SRV-WIN-EBS-01` avec PowerShell.

Le module `ActiveDirectory` fournit la commande :

`Move-ADDirectoryServerOperationMasterRole`

Elle permet de choisir le contrôleur de destination
et les rôles à transférer.

Après l’opération, je consulte de nouveau les détenteurs
pour vérifier le résultat.

## 5. Transfert et saisie forcée

Dans ce projet, les deux contrôleurs sont disponibles :
je travaille donc sur un transfert normal.

La saisie forcée concerne une autre situation,
lorsque le détenteur ne peut plus rendre son rôle
par un transfert normal.

Elle demande une analyse de la panne
et des précautions sur le retour éventuel de l’ancien détenteur.

## 6. Vérifications après les transferts

Je contrôle les détenteurs après chaque transfert.

Je vérifie également :

- la disponibilité des deux contrôleurs ;
- l’état de la réplication ;
- le fonctionnement de l’authentification ;
- la présence des objets de test sur les deux contrôleurs.

Le déplacement des rôles ne doit pas empêcher
le fonctionnement normal du domaine.

## Résultat

Les rôles ont été transférés vers Server Core,
puis retransférés vers le premier contrôleur avec PowerShell.

Ce travail m’a permis de repérer les consoles utilisées,
de comprendre la fonction des rôles et de vérifier
leur attribution après une modification.

## Compétences mobilisées

- Identification des rôles FSMO.
- Administration avec les consoles Active Directory.
- Transfert des rôles avec PowerShell.
- Vérification des détenteurs.
- Contrôle de la réplication après intervention.
