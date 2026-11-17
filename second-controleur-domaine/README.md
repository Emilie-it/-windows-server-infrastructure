# Active Directory — Second contrôleur de domaine et redondance DNS

## Présentation

J’ajoute un second contrôleur de domaine sur Windows Server Core
dans le domaine existant `TSSR-EBS.LAB`.

Je vérifie ensuite la réplication Active Directory
et la présence des données DNS sur les deux serveurs.

## Environnement technique

| Élément | Configuration |
|---|---|
| Hyperviseur | VMware Workstation |
| Domaine | `TSSR-EBS.LAB` |
| Premier contrôleur | `SRV-WIN-EBS-01` — Windows Server 2025 avec interface graphique |
| Adresse du premier contrôleur | `192.168.100.10` |
| Second contrôleur | `SRV-WIN-EBS-02` — Windows Server 2025 Core |
| Adresse du second contrôleur | `192.168.100.11` |
| Réseau | `192.168.100.0/24` |
| Poste client | Windows 11 Pro |

## Objectifs

- Installer AD DS sur Server Core.
- Ajouter un contrôleur au domaine existant.
- Vérifier la réplication de l’annuaire.
- Vérifier les zones et les enregistrements DNS.
- Tester la résolution depuis le poste client.

## 1. Préparation du serveur Core

Je vérifie le nom du serveur, son adresse IPv4
et sa configuration DNS.

Avant la promotion, le serveur utilise le DNS
du premier contrôleur, à l’adresse `192.168.100.10`,
pour retrouver le domaine.

Je vérifie également la communication avec le premier serveur.

## 2. Installation du rôle AD DS

Depuis PowerShell, j’installe le rôle
Active Directory Domain Services avec ses outils de gestion.

J’utilise `Get-WindowsFeature` pour vérifier
la présence du rôle.

Je consulte également les commandes disponibles
dans le module `ADDSDeployment`.

## 3. Promotion en contrôleur de domaine

J’utilise `Install-ADDSDomainController` pour ajouter
`SRV-WIN-EBS-02` au domaine existant `TSSR-EBS.LAB`.

Je demande également l’installation du service DNS.

Je fournis les identifiants nécessaires à l’opération
et je termine la promotion avec le redémarrage du serveur.

Je crée ainsi un contrôleur supplémentaire
dans le même domaine.

## 4. Vérification de la réplication Active Directory

Je vérifie que les deux contrôleurs sont reconnus,
puis je consulte l’état de la réplication.

| Outil | Utilisation |
|---|---|
| `Get-ADDomainController -Filter *` | Afficher les contrôleurs du domaine |
| `nltest /dclist:TSSR-EBS.LAB` | Lister les contrôleurs connus |
| `repadmin /replsummary` | Consulter le résumé de la réplication |
| `Get-ADOrganizationalUnit` | Consulter les OU sur un contrôleur donné |

Je compare les unités d’organisation présentes
sur les deux serveurs.

## 5. Vérification du service DNS

Sur Server Core, je vérifie l’installation du rôle DNS
et l’état du service.

Je retrouve la zone `TSSR-EBS.LAB` sur les deux serveurs.

La propriété `IsDsIntegrated` indique que la zone
est intégrée à Active Directory.

Ses données sont donc répliquées par les mécanismes
d’Active Directory, selon sa portée de réplication.

Je vérifie également les enregistrements nécessaires
à la recherche des services du domaine.

## 6. Tests de réplication DNS

Je crée un enregistrement de test depuis
le premier contrôleur, puis je recherche sa présence
sur le second.

Je réalise également un test dans l’autre sens,
avec l’enregistrement `test-retour` créé depuis Server Core.

Ces manipulations permettent de vérifier
la propagation des enregistrements entre les serveurs.

## 7. Vérifications depuis Windows 11

Je configure la distribution des deux DNS internes
dans les options DHCP :

- `192.168.100.10` ;
- `192.168.100.11`.

Depuis le client, je renouvelle la configuration réseau
et je vérifie les paramètres obtenus.

Je teste ensuite la résolution avec `Resolve-DnsName`,
en interrogeant explicitement chacun des deux DNS.

## Résultat

Les deux contrôleurs sont présents dans le domaine
et la réplication Active Directory fonctionne.

La zone DNS est présente sur les deux serveurs.
Les enregistrements de test sont retrouvés
et le client peut interroger chacun des DNS.

Cette configuration apporte une redondance
pour l’annuaire et la résolution DNS.
Elle ne remplace pas une sauvegarde d’Active Directory.

## Compétences mobilisées

- Installation d’AD DS en PowerShell.
- Promotion de Server Core en contrôleur de domaine.
- Vérification de la réplication Active Directory.
- Contrôle des zones DNS intégrées à l’annuaire.
- Tests de résolution depuis un poste client.
