# Windows Server — Configuration des GPO

## Présentation

Je configure des stratégies de groupe pour appliquer
des paramètres aux utilisateurs et aux ordinateurs
du domaine `TSSR-EBS.LAB`.

Je travaille sur les restrictions d’accès, le Bureau
à distance, les imprimantes et la personnalisation
de l’environnement utilisateur.

## Environnement technique

| Élément | Configuration |
|---|---|
| Virtualisation | Proxmox pour les manipulations décrites sur cette page |
| Serveur | Windows Server 2025 |
| Domaine | `TSSR-EBS.LAB` |
| Poste client | Windows 11 Pro |
| Administration | Console Gestion des stratégies de groupe |
| Vérification | Tests sur le client et `gpresult` |

## Objectifs

- Créer et configurer des GPO.
- Les lier au bon emplacement dans Active Directory.
- Distinguer les paramètres utilisateur et ordinateur.
- Définir les utilisateurs ou les postes concernés.
- Vérifier l’application des paramètres sur le client.

## 1. Organisation des stratégies

Je crée les GPO dans la console Gestion des stratégies
de groupe, puis je les lie aux emplacements concernés.

Une GPO contient des paramètres. Son lien dans
Active Directory participe à définir son périmètre
d’application.

| Partie de la GPO | Objets concernés |
|---|---|
| Configuration utilisateur | Comptes utilisateurs |
| Configuration ordinateur | Comptes ordinateurs |

Je distingue donc l’emplacement du compte utilisateur
de celui du compte ordinateur avant de choisir
où lier la GPO.

## 2. Restriction du Panneau de configuration

Je configure une restriction pour empêcher les utilisateurs
concernés d’accéder au Panneau de configuration
et à l’application Paramètres.

### Paramètre utilisé

Configuration utilisateur → Stratégies →
Modèles d’administration → Panneau de configuration →
Interdire l’accès au Panneau de configuration
et à l’application Paramètres du PC.

J’active ce paramètre.

### Périmètre

Je souhaite appliquer cette restriction aux utilisateurs
des services concernés, en conservant l’accès
pour le service informatique.

J’ai testé un filtrage par refus d’application pour
le groupe informatique, puis travaillé sur un ciblage
par les OU des utilisateurs concernés.

Le ciblage par OU permet ici de rendre le périmètre
plus facile à lire et d’éviter une exception par refus.

### Vérification

Sur le client, j’ouvre une session avec un utilisateur
concerné et je teste l’accès aux Paramètres.

Je consulte également les stratégies appliquées
avec `gpresult /r`.

Le compte rendu contient un test de restriction réussi.

## 3. Autorisation du Bureau à distance

Je configure une GPO pour autoriser les connexions
Bureau à distance sur les ordinateurs concernés.

### Paramètre utilisé

Configuration ordinateur → Stratégies →
Modèles d’administration → Composants Windows →
Services Bureau à distance →
Hôte de la session Bureau à distance → Connexions →
Autoriser les utilisateurs à se connecter à distance
à l’aide des Services Bureau à distance.

Je configure également les règles entrantes
du pare-feu associées au Bureau à distance.

### Vérification

Je teste la connexion depuis une autre machine
et je consulte les stratégies appliquées.

L’activation du service et des règles réseau ne donne
pas automatiquement le droit de connexion
à tous les utilisateurs.

Le compte utilisé doit également être autorisé
à ouvrir une session Bureau à distance.

### Sécurité

Le périmètre de la GPO doit correspondre aux postes
qui ont réellement besoin de cet accès.

## 4. Déploiement d’une imprimante partagée

Je configure une préférence utilisateur pour ajouter
l’imprimante partagée `IMP-EBS-01`.

### Emplacement utilisé

Configuration utilisateur → Préférences →
Paramètres du Panneau de configuration → Imprimantes.

Je crée une imprimante partagée avec l’action
« Mettre à jour », puis je renseigne son chemin réseau :

`\\SRV-WIN-EBS-01\IMP-EBS-01`

### Vérification

Depuis une session utilisateur, je vérifie
que l’imprimante apparaît parmi les imprimantes disponibles.

La présence de l’imprimante et l’impression effective
d’un document sont deux contrôles différents.

## 5. Configuration d’un fond d’écran

Je configure un paramètre utilisateur pour définir
un fond d’écran commun.

### Paramètre utilisé

Configuration utilisateur → Stratégies →
Modèles d’administration → Bureau → Bureau →
Papier peint du Bureau.

Le fichier utilisé doit être accessible en lecture
par les utilisateurs concernés.

### Périmètre

Comme ce paramètre appartient à la configuration
utilisateur, je dois tenir compte de l’emplacement
des comptes utilisateurs pour son application normale.

Un lien uniquement sur l’OU des ordinateurs ne suffit
pas à appliquer ce paramètre aux utilisateurs,
sauf configuration particulière du traitement
en boucle des stratégies utilisateur.

## 6. Ajout des modèles d’administration Chrome

Pour disposer des paramètres Google Chrome
dans l’éditeur de GPO, j’ajoute ses modèles
d’administration.

| Type de fichier | Rôle |
|---|---|
| `.admx` | Définition des paramètres |
| `.adml` | Textes et traductions affichés dans l’éditeur |

Je prépare le magasin central à cet emplacement :

`\\TSSR-EBS.LAB\SYSVOL\TSSR-EBS.LAB\Policies\PolicyDefinitions`

Les fichiers ADMX sont placés à la racine
de `PolicyDefinitions`.

Les fichiers ADML sont placés dans les dossiers
de langue correspondants, notamment `fr-FR`
et `en-US`.

Ces fichiers permettent de configurer les stratégies
de Chrome. Ils n’installent pas le navigateur
sur les postes clients.

## 7. Vérifications

Après les modifications, je demande une actualisation
des stratégies avec `gpupdate /force`.

Je vérifie ensuite leur application avec `gpresult`
et des tests sur le poste client.

| Contrôle | Ce que je recherche |
|---|---|
| Lien de la GPO | Bon emplacement dans Active Directory |
| Partie configurée | Utilisateur ou ordinateur |
| Filtrage | Compte autorisé à appliquer la stratégie |
| Rapport `gpresult` | Présence de la GPO dans les stratégies appliquées |
| Test sur le client | Comportement attendu du paramètre |

Certaines modifications nécessitent une fermeture
de session ou un redémarrage.

Une GPO présente dans le rapport ne remplace pas
le test du comportement attendu.

## Résultat

J’ai travaillé sur plusieurs usages des GPO :
restriction des Paramètres, accès distant,
déploiement d’une imprimante et personnalisation
de l’environnement utilisateur.

Ces manipulations m’ont permis de mieux comprendre
le lien entre le contenu d’une GPO, son emplacement
et les comptes auxquels elle s’applique.

## Compétences mobilisées

- Création et modification de GPO.
- Ciblage des utilisateurs et des ordinateurs.
- Configuration de préférences utilisateur.
- Gestion de paramètres de sécurité et de pare-feu.
- Ajout de modèles d’administration.
- Vérification avec `gpresult` et tests sur le client.

## Références

- [Documentation Microsoft sur les stratégies de groupe](https://learn.microsoft.com/fr-fr/windows-server/identity/ad-ds/manage/group-policy/group-policy-overview)
- [Documentation Google Chrome Enterprise](https://chromeenterprise.google/policies/)
