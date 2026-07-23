# Structure Active Directory — Forêt, OU, groupe, utilisateur

## Objectif

Mettre en place un contrôleur de domaine Windows Server 2022 et poser une première structure d'annuaire (forêt, OU, groupe, utilisateur) servant de base au lab.

---

## Environnement

| Élément | Détail |
|---|---|
| Hyperviseur | VirtualBox sur Dell Latitude 5420 (Ubuntu) |
| OS invité | Windows Server 2022 Standard (Evaluation), Desktop Experience |
| Ressources allouées | 4 Go RAM, 2 vCPU |
| Réseau | IP statique `10.0.2.15/24`, passerelle `10.0.2.2` |

---

## Rôles installés

- **AD DS** (Active Directory Domain Services)
- **DNS Server**

Promotion du serveur en contrôleur de domaine, création de la forêt `loupbleu.local`.

---

## Structure créée

| Objet | Type | Description |
|---|---|---|
| `loupbleu.local` | Forêt / Domaine | Domaine racine |
| Employes | Unité d'Organisation | Point d'ancrage des futures GPO |
| Baristas | Groupe de sécurité, étendue Globale | Regroupement pour la gestion des permissions |
| Jean Dupont (`jdupont@loupbleu.local`) | Compte utilisateur | Membre de Baristas, dans l'OU Employes |

Équivalent PowerShell de ces créations (faites ici via l'interface graphique) :

```powershell
New-ADOrganizationalUnit -Name "Employes" -Path "DC=loupbleu,DC=local"

New-ADGroup -Name "Baristas" -GroupScope Global -GroupCategory Security `
  -Path "OU=Employes,DC=loupbleu,DC=local"

New-ADUser -Name "Jean Dupont" -SamAccountName "jdupont" `
  -UserPrincipalName "jdupont@loupbleu.local" `
  -Path "OU=Employes,DC=loupbleu,DC=local" -Enabled $true

Add-ADGroupMember -Identity "Baristas" -Members "jdupont"
```

Le groupe `Baristas` a été créé avec les valeurs par défaut de l'assistant AD : étendue **Globale**, type **Sécurité**.

---

## OU vs Groupe

Deux objets qui se ressemblent en apparence (« regrouper des objets ») mais qui n'ont pas la même fonction :

- **OU** → point d'ancrage pour les **GPO** (stratégies de groupe) et pour la délégation d'administration. Une GPO liée à une OU s'applique à tous les objets qu'elle contient, quel que soit leur groupe (héritage).
- **Groupe de sécurité** → attribution de **permissions** sur des ressources (dossiers, imprimantes...), via son SID inscrit dans les ACL. Un groupe de distribution n'a pas de SID exploitable pour ça — usage messagerie uniquement.

Une GPO ne peut jamais être liée directement à un groupe, seulement à un domaine, un site, ou une OU.

---

## Ports et protocoles clés d'Active Directory

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
| 123 | UDP | NTP (synchronisation horaire) |

## NTP et Kerberos

Chaque ticket Kerberos embarque un horodatage. Le DC rejette un ticket si l'écart avec sa propre horloge dépasse **5 minutes** (le *clock skew*), même si le ticket est authentique. Cette tolérance limitée protège contre une **attaque par rejeu** : un ticket intercepté sur le réseau et réutilisé plus tard serait automatiquement invalidé par son timestamp trop ancien.

---

## Points de vigilance

- Mot de passe unique et simple utilisé partout pour l'instant
- Pare-feu Windows non configuré
- Pas de compte admin nominatif séparé du compte Administrateur natif
- Synchronisation NTP non vérifiée formellement
