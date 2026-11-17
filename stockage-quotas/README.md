# Windows Server — Gestion du stockage et des quotas

## Présentation

Je configure des quotas pour limiter l’espace utilisé
dans les dossiers partagés des services.

J’utilise le Gestionnaire de ressources du serveur de fichiers
(FSRM), puis je teste le comportement lorsque les limites
sont atteintes.

Je travaille également avec `fsutil` pour découvrir
la gestion des quotas NTFS sur un volume.

## Environnement technique

| Élément | Configuration |
|---|---|
| Hyperviseur | Proxmox VE |
| Serveur | Windows Server 2025 |
| Domaine | `TSSR-EBS.LAB` |
| Dossiers concernés | Sous-dossiers de `C:\PARTAGE` |
| Gestion des quotas par dossier | FSRM |
| Outil en ligne de commande | `fsutil` |

## Objectifs

- Installer FSRM.
- Définir des limites de stockage par dossier.
- Observer l’utilisation de l’espace.
- Tester le blocage lorsque le quota est atteint.
- Distinguer les quotas FSRM des quotas NTFS.

## 1. Installation de FSRM

Depuis le Gestionnaire de serveur, j’ouvre
l’assistant d’ajout de rôles et de fonctionnalités.

Dans Services de fichiers et de stockage →
Services de fichiers et iSCSI, je sélectionne
le Gestionnaire de ressources du serveur de fichiers.

J’installe également les outils de gestion,
puis j’ouvre la console FSRM.

FSRM permet notamment de gérer des quotas sur les dossiers
et de définir des actions lorsque certains seuils
d’utilisation sont atteints.

## 2. Création des quotas

Dans la console FSRM, j’ouvre
Gestion de quotas → Quotas → Créer un quota.

Je sélectionne le dossier concerné et je définis
ses propriétés de quota.

J’applique notamment une limite de 150 Mo au dossier :

`C:\PARTAGE\ADMINISTRATIF`

Je configure également des quotas sur d’autres dossiers
de services.

Le quota porte sur l’ensemble du contenu du dossier,
y compris ses sous-dossiers. Il ne s’agit pas
d’une limite séparée pour chaque utilisateur.

## 3. Choix du type de quota

FSRM propose deux comportements.

| Type | Comportement |
|---|---|
| Quota inconditionnel — strict | Empêche les écritures qui dépasseraient la limite |
| Quota conditionnel — souple | Suit l’utilisation sans bloquer le dépassement |

Pour tester la limitation de stockage,
j’utilise des quotas inconditionnels.

Les seuils permettent également de prévoir des notifications
ou des actions avant que la limite soit atteinte.

## 4. Tests de remplissage

J’ajoute des fichiers dans les dossiers concernés
et j’observe l’utilisation dans la console FSRM.

Le pourcentage utilisé augmente avec le volume
des fichiers ajoutés.

Je poursuis les tests pour atteindre la limite
et observer le comportement du quota.

| Test | Résultat observé |
|---|---|
| Ajout de fichiers | Augmentation de l’utilisation affichée dans FSRM |
| Dépassement d’un quota strict | Écriture bloquée avec un message d’espace insuffisant |
| Consultation de la console | Suivi de l’espace utilisé et de la limite configurée |

Le message d’espace insuffisant peut donc venir
du quota du dossier, même si le volume dispose
encore d’espace libre.

## 5. Modèles et extension des quotas

Je travaille également sur les modèles de quotas.

Un modèle permet de réutiliser une limite
et des paramètres de notification sur plusieurs dossiers.

J’aborde aussi le principe d’une extension automatique :
une action exécutée à un seuil peut appliquer
un autre modèle avec une limite plus élevée.

Ce comportement est différent d’un quota souple :
la limite du quota est modifiée au lieu d’être
simplement dépassée sans blocage.

## 6. Découverte des quotas NTFS avec fsutil

J’utilise également la commande suivante :

`fsutil quota track C:`

Elle active le suivi des quotas NTFS sur le volume `C:`.
Elle ne crée pas de quota FSRM sur les dossiers
de `C:\PARTAGE`.

Les deux mécanismes sont différents :

| Élément | Quotas FSRM | Quotas NTFS |
|---|---|---|
| Périmètre | Dossier et sous-dossiers | Volume NTFS |
| Calcul de l’utilisation | Ensemble du contenu du dossier | Espace attribué à chaque utilisateur selon le propriétaire des fichiers |
| Outil utilisé | Console FSRM | `fsutil` ou propriétés du volume |
| Limitation | Limite du dossier | Limite par utilisateur sur le volume |

La commande `track` active le suivi.
Elle ne suffit pas à définir et faire appliquer
une limite de stockage par utilisateur.

## 7. Vérifications

Pour analyser un blocage d’écriture, je distingue :

- l’espace libre sur le volume ;
- le quota FSRM du dossier ;
- les éventuels quotas NTFS ;
- les permissions de l’utilisateur.

Ces contrôles permettent d’éviter de confondre
un manque d’espace, une limite de quota
et un refus d’accès.

## Résultat

Les quotas FSRM permettent de limiter l’espace utilisé
par les dossiers des services.

Les tests montrent l’augmentation de l’utilisation
et le blocage des écritures lorsque la limite
d’un quota strict est atteinte.

Le travail avec `fsutil` m’a également permis
de distinguer les quotas par dossier des quotas NTFS
par utilisateur sur un volume.

## Compétences mobilisées

- Installation de FSRM.
- Configuration de quotas par dossier.
- Utilisation de modèles de quotas.
- Test des limites de stockage.
- Lecture des résultats dans la console FSRM.
- Découverte du suivi des quotas NTFS avec `fsutil`.

## Références

- [Présentation de FSRM](https://learn.microsoft.com/fr-fr/windows-server/storage/fsrm/fsrm-overview)
- [Commande fsutil quota](https://learn.microsoft.com/fr-fr/windows-server/administration/windows-commands/fsutil-quota)
