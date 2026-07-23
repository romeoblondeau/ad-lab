# AD-Lab — Active Directory Homelab

Lab Active Directory déployé sur Windows Server 2022, dans le but de comprendre le fonctionnement légitime d'un environnement AD — brique centrale de l'infrastructure de la majorité des entreprises — afin d'être capable, à terme, de détecter les comportements anormaux et les attaques qui le ciblent.

Ce projet s'inscrit dans une démarche de reconversion vers la cybersécurité (Admin Infrastructure Sécurisée → SOC Analyst), et vient compléter mon projet [Cyberdeck](https://github.com/romeoblondeau/cyberdeck) (SIEM/SOC dashboard), avec pour objectif futur de forwarder les logs de sécurité Windows vers ce pipeline de supervision.

---

## Matériel

| Machine | Rôle |
|---|---|
| Dell Latitude 5420 — Ubuntu, VirtualBox | Hyperviseur hôte |
| VM Windows Server 2022 Standard (Evaluation), Desktop Experience — 4 Go RAM, 2 vCPU | Contrôleur de domaine (DC) |

*(VM Windows 10 cliente à ajouter une fois jointe au domaine)*

---

## Stack technique

| Composant | Technologie |
|---|---|
| Hyperviseur | VirtualBox |
| OS invité | Windows Server 2022 Standard (Evaluation), Desktop Experience |
| Rôles serveur | AD DS (Active Directory Domain Services), DNS Server |
| Structure du domaine | Forêt `loupbleu.local` |
| Réseau | IP statique 10.0.2.15/24, passerelle 10.0.2.2 |

---

## Objets AD déployés

| Objet | Type | Description |
|---|---|---|
| `loupbleu.local` | Forêt / Domaine | Domaine racine du lab |
| Employes | Unité d'Organisation (OU) | Conteneur pour organiser les objets employés, futur point d'ancrage des GPO |
| Baristas | Groupe de sécurité, étendue Globale | Regroupe les utilisateurs pour la gestion des permissions |
| Jean Dupont (`jdupont@loupbleu.local`) | Compte utilisateur | Membre du groupe Baristas, situé dans l'OU Employes |

---

## Points de vigilance / Sécurité à renforcer

État des lieux honnête d'un DC fraîchement installé, sans durcissement particulier pour l'instant :

- **Mots de passe** — un seul mot de passe simple utilisé partout pour l'instant. À faire : politique de mot de passe via GPO (longueur, complexité, expiration)
- **Pare-feu Windows** — non configuré pour l'instant. À faire : définir les règles pour les ports AD nécessaires (voir section Réseau/Ports)
- **Comptes à privilèges** — pas de compte admin nominatif séparé du compte Administrateur natif. À faire : appliquer le principe du moindre privilège
- **Synchronisation NTP** — non vérifiée. Point important : Kerberos rejette les tickets si l'horloge dévie de plus de 5 min (protection contre le rejeu de tickets volés)

---

## Ports et protocoles clés (Active Directory)

| Port | Protocole | Service |
|---|---|---|
| 53 | TCP/UDP | DNS |
| 88 | TCP/UDP | Kerberos (authentification) |
| 389 | TCP/UDP | LDAP (requêtes annuaire) |
| 636 | TCP | LDAPS (LDAP chiffré) |
| 445 | TCP | SMB (partage de fichiers, GPO) |
| 135 | TCP | RPC Endpoint Mapper |
| 3268 / 3269 | TCP | Global Catalog (+ SSL) |
| 464 | TCP/UDP | Changement de mot de passe Kerberos |
| 123 | UDP | NTP (synchronisation horaire, critique pour Kerberos) |

---

## Prochaines étapes

- [ ] Joindre une VM Windows 10 cliente au domaine `loupbleu.local`
- [ ] Tester la connexion avec le compte `jdupont`
- [ ] Créer une première GPO (ex : fond d'écran imposé, politique de mot de passe)
- [ ] Configurer le pare-feu Windows Defender sur le DC
- [ ] Forwarding des logs Windows Event Log (authentification, création de comptes, GPO) vers le pipeline SOC [Cyberdeck](https://github.com/romeoblondeau/cyberdeck) (Promtail/Loki/Grafana) — outil à déterminer (Winlogbeat, NXLog, agent natif)

---

## Objectifs pédagogiques

- Comprendre l'architecture AD (forêt, domaine, OU, groupes, utilisateurs)
- Maîtriser la différence entre GPO (politique, appliquée sur une OU) et permissions (groupes de sécurité)
- Identifier les protocoles et ports critiques d'AD (Kerberos, LDAP, DNS)
- Repérer les mauvaises pratiques de sécurité sur un DC non durci
- Préparer une intégration future avec un pipeline SOC (corrélation logs AD ↔ détection)
