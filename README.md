# Lab_2_MobileSecurity

## Partie 1 — Vérification ADB

### But
Vérifier que l’**AVD** est bien détecté avant de commencer le lab.

### Étapes
1. Lancer l’**AVD** (Pixel 5).
2. Ouvrir PowerShell dans `platform-tools`.
3. Exécuter :

## Partie 2 — Étape 1 : Rooter l’AVD

### Objectif
Obtenir des privilèges **root** sur un **AVD** (environnement jetable) et observer l’état des contrôles d’intégrité (**AVB / vbmeta / verity**).

### Pré-requis
- AVD démarré
- `adb` détecte l’émulateur




### Où trouver le fichier logcat
Le fichier a été généré dans le dossier de travail utilisé pendant l’export, ici :

`C:\Users\Ismai\lab_logs\logcat_root_check.txt`

Pour l’ouvrir rapidement :
```powershell
explorer .
notepad .\logcat_root_check.txt



