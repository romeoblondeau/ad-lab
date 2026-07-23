# GPO — Fond d'écran imposé

## Objectif

Créer et lier une première stratégie de groupe pour tester le mécanisme complet des GPO : liaison à une OU, partage réseau, configuration d'un paramètre concret.

---

## Liaison de la GPO

Une GPO ne peut être liée qu'à un domaine, un site, ou une OU — jamais directement à un groupe de sécurité. Création et liaison sur l'OU `Employes` :

```
Clic droit sur l'OU Employes → Créer un GPO dans ce domaine, et le lier ici...
Nom : Employe - Fond d'écran imposé
```

Tout objet placé dans `Employes` hérite automatiquement de cette GPO, y compris de futurs groupes qui viendraient s'y ajouter.

---

## Partage réseau pour l'image

Le paramètre attend un chemin **UNC** (`\\serveur\partage\fichier`), pas un chemin local — un chemin local n'aurait de sens que si le fichier existait identiquement sur chaque poste client.

Choix retenu : un partage dédié plutôt que SYSVOL (partage système utilisé pour les fichiers de GPO eux-mêmes, à ne pas mélanger avec du contenu applicatif).

| Élément | Valeur |
|---|---|
| Dossier local sur le DC | `C:\Partage\Wallpaper` |
| Nom du partage | `Wallpaper` |
| Chemin UNC | `\\WIN-LL6GJET7NUT\Wallpaper\SW_wallpaper.jpg` |
| Permissions | Lecture seule |

---

## Transfert du fichier — Guest Additions

Le dossier partagé VirtualBox (hôte ↔ VM) configuré pour transférer l'image n'apparaissait pas côté Windows. Cause : les **VirtualBox Guest Additions** n'étaient pas installées sur la VM.

Premier essai via `Devices → Insert Guest Additions CD image...` : échec silencieux du téléchargement (barre à 100 % quasi instantanément, message `During certificate downloading: Unknown reason`).

Vérification réseau côté VM avant de chercher ailleurs :

```
C:\Users\Administrateur>ping 10.0.2.2
Réponse de 10.0.2.2 : octets=32 temps<1ms TTL=255 (0% de perte)

C:\Users\Administrateur>ping google.com
Réponse de 172.217.20.46 : octets=32 temps=8ms TTL=255 (0% de perte)
```

Réseau VM fonctionnel — le téléchargement de l'ISO par VirtualBox se fait depuis l'hôte (Ubuntu), pas depuis la VM. Recherche d'une copie locale déjà présente sur Ubuntu :

```bash
find / -iname "VBoxGuestAdditions.iso" 2>/dev/null
# (aucun résultat)
```

Installation du paquet fournissant l'ISO en local :

```bash
sudo apt update
sudo apt install virtualbox-guest-additions-iso
```

Montage manuel dans la VM via `Devices → Optical Drives → Choose a disk file...`, exécution de `VBoxWindowsAdditions-amd64.exe`, redémarrage. Après redémarrage : dossier partagé visible, image copiée vers `C:\Partage\Wallpaper`.

---

## Configuration du paramètre GPO

```
Configuration utilisateur
  → Stratégies
    → Modèles d'administration
      → Bureau
        → Bureau
          → Papier peint du Bureau
```

- **État** : Activé
- **Nom du papier peint** : `\\WIN-LL6GJET7NUT\Wallpaper\SW_wallpaper.jpg`
- **Style** : Ajuster

Paramètre situé côté Configuration **utilisateur** (propre à la session), pas Configuration ordinateur.

---

## Test

Pas de test possible sur le DC lui-même (`gpupdate /force` n'aurait aucun effet visible, le DC n'étant pas membre de l'OU Employes). Vérification visuelle reportée à la jonction d'une VM Windows 10 cliente et à la connexion du compte `jdupont`.
