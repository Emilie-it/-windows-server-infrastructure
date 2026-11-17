# Windows Server — DHCP et basculement

## Présentation

Je configure une relation de basculement DHCP
entre les deux serveurs Windows.

Le premier serveur est actif et le second,
installé sur Server Core, assure le secours.

Je teste ensuite une indisponibilité du service principal,
puis son retour au fonctionnement normal.

## Environnement technique

| Élément | Configuration |
|---|---|
| Hyperviseur | VMware Workstation |
| Domaine | `TSSR-EBS.LAB` |
| Serveur actif | `SRV-WIN-EBS-01` — `192.168.100.10` |
| Serveur de secours | `SRV-WIN-EBS-02` — `192.168.100.11` |
| Étendue | `LAN-TSSR` — réseau `192.168.100.0/24` |
| Mode de basculement | Hot Standby — actif/secours |
| Relation | `CLUSTER-DHCP-TSSR-EBS` |
| Réserve d’adresses | 10 % |
| Durée du bail | 8 jours |
| Poste client | Windows 11 Pro |

Le nom donné à la relation ne signifie pas
qu’un cluster Windows a été installé.
Il s’agit d’une relation de basculement DHCP.

## Objectifs

- Installer DHCP sur Server Core.
- Autoriser les serveurs DHCP dans Active Directory.
- Configurer la distribution des paramètres réseau.
- Créer une relation de basculement.
- Tester le serveur de secours et le retour à la normale.

## 1. Installation et autorisation de DHCP

J’installe le rôle DHCP et ses outils de gestion
sur `SRV-WIN-EBS-02`.

Je vérifie ensuite la présence du rôle
et le fonctionnement du service.

J’autorise le serveur dans Active Directory
et je consulte les journaux d’événements.

Les événements DHCP 1044 consignés dans mon compte rendu
confirment l’autorisation des deux serveurs à cette étape.

## 2. Vérification de l’étendue

Je vérifie l’étendue existante et les options distribuées
aux postes clients.

| Option | Valeur |
|---|---|
| 006 — Serveurs DNS | `192.168.100.10` et `192.168.100.11` |
| 015 — Nom de domaine DNS | `TSSR-EBS.LAB` |

Depuis Windows 11, je contrôle la configuration
avec `ipconfig /all`.

Le client reçoit l’adresse `192.168.100.150/24`
du serveur `192.168.100.10`, avec les deux DNS internes
et le suffixe du domaine.

## 3. Configuration du basculement

Je crée la relation avec
`Add-DhcpServerv4Failover`.

Je choisis le mode Hot Standby :

- `SRV-WIN-EBS-01` est le serveur actif ;
- `SRV-WIN-EBS-02` est le serveur de secours ;
- une réserve de 10 % est configurée.

Ce mode est différent du partage de charge,
dans lequel les deux serveurs traitent les demandes
selon une répartition définie.

## 4. Vérification des paramètres

Je contrôle la relation sur les deux serveurs.

| Commande | Utilisation |
|---|---|
| `Get-DhcpServerv4Failover` | Vérifier le partenaire, le mode et l’état de la relation |
| `Get-DhcpServerv4Scope` | Vérifier la présence de l’étendue |
| `Get-DhcpServerv4OptionValue` | Consulter les options distribuées |
| `Get-DhcpServerv4Lease` | Consulter les baux |

Je retrouve la même étendue et les mêmes paramètres
sur le serveur de secours.

## 5. Test depuis le client

Depuis Windows 11, je libère puis renouvelle
le bail DHCP.

Je vérifie l’adresse reçue, les serveurs DNS
et l’identité du serveur DHCP.

Avant la simulation de panne, le bail est obtenu
depuis le serveur actif `192.168.100.10`.

## 6. Simulation d’une indisponibilité

J’arrête le service DHCP sur `SRV-WIN-EBS-01`.

Sur le second serveur, la relation passe
à l’état `CommunicationInterrupted`.

Depuis Windows 11, je libère puis renouvelle le bail.
Le client obtient une configuration depuis
le serveur de secours `192.168.100.11`.

Ce test confirme la prise en charge du client
par le serveur de secours dans les conditions testées.

L’état `CommunicationInterrupted` indique une perte
de communication entre les partenaires.
Il est distinct de l’état `PartnerDown`.

## 7. Retour au fonctionnement normal

Je redémarre le service DHCP sur le premier serveur
avec `Start-Service DHCPServer`.

Je vérifie ensuite :

- que le service est démarré ;
- que la relation revient à l’état `Normal`
  sur les deux serveurs ;
- que les baux sont présents et actifs.

Le retour à l’état normal est confirmé
dans mon compte rendu.

## 8. Réplication des modifications de configuration

Les informations de bail sont échangées
dans le cadre de la relation de basculement.

Après une modification de la configuration de l’étendue,
je réplique cette configuration vers le partenaire.

J’utilise pour cela :

`Invoke-DhcpServerv4FailoverReplication`

Je distingue donc la synchronisation des baux
de la réplication des modifications de configuration.

## Résultat

Le client obtient une configuration réseau valide
depuis le serveur principal.

Lors de l’arrêt du service principal, le serveur
de secours prend en charge le renouvellement testé.

Après le rétablissement du service, la relation revient
à l’état normal sur les deux serveurs.

## Compétences mobilisées

- Installation et autorisation du rôle DHCP.
- Configuration des options d’étendue.
- Mise en place du mode Hot Standby.
- Consultation des baux et des journaux.
- Simulation d’une panne de service.
- Vérification du basculement et du retour à la normale.
