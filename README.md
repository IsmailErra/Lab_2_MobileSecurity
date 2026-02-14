# Lab_2_MobileSecurity

> Objectif général : préparer un environnement Android de test (**AVD**), vérifier **ADB**, obtenir le **root** sur l’émulateur, puis travailler sur un **AVD propre** pour éviter des résultats faussés.

---

## Partie 1 — Vérification ADB

### But
Vérifier que l’**AVD** est bien détecté avant de commencer le lab.

### Étapes
1. Lancer l’**AVD** (Pixel — émulateur).
2. Ouvrir PowerShell.
3. Vérifier la connexion :

```powershell
adb devices
```

### Résultat attendu
- L’émulateur apparaît en **device** (ex: `emulator-5554   device`).
![](lb2/pre.png)


---

## Partie 2 — Étape 1 : Rooter l’AVD

### Objectif
Obtenir des privilèges **root** sur un **AVD** (environnement jetable) et observer l’état des contrôles d’intégrité (**AVB / vbmeta / verity**).

### Pré-requis
- AVD démarré
- `adb` détecte l’émulateur

### Vérifications principales (preuves)
- `adb root` :ôle : vérifier que **adbd** tourne en mode root.
- `adb shell id` : vérifier `uid=0(root)`.
- `getprop` : observer **verity / vbmeta / avb** (selon l’image Android).

### Journalisation (preuve)
Exporter les 200 dernières lignes de logcat :

### Résultat attendu
![](lb2/part1.png)

```powershell
adb logcat -d -t 200 > logcat_root_check.txt
```

### Où trouver le fichier logcat
Le fichier a été généré ici :

`C:\Users\Ismai\lab_logs\logcat_root_check.txt`

Pour l’ouvrir rapidement :

```powershell
explorer .
notepad .\logcat_root_check.txt
```
### Résultats attendu
![](lb2/prt1Last.png)
![](lb2/prt1Last2.png)


## Partie 3 — Étape 3 : Démarrer un AVD propre

### But
Démarrer un **AVD propre** (sans applications résiduelles et sans compte personnel) pour éviter des résultats faussés.

### Actions
- Android Studio → Device Manager → **Start** (ou créer un AVD récent, API 29+).
- Vérifier visuellement : **écran d’accueil Android**, **aucun compte personnel**.

### Vérification
```powershell
adb devices
```

### Résultat attendu
![](lb2/step3CleanAvd.png)

---
## Étape 4 — Installer et lancer l’app de test (DIVA)

### Source de l’APK
L’application de test (**DIVA**) a été récupérée depuis GitHub (dépôt contenant le fichier APK), puis installée sur l’AVD via **ADB**.
![](lb2/divaApp.png)

### Installation + Lancement
- Installation de l’APK sur l’émulateur avec `adb install`.
- Ouverture de l’application sur l’AVD et vérification qu’un scénario simple est réalisable.

### Version (rapport)
La version de l’application a été relevée (car le comportement de sécurité peut varier selon la version).

### Résultat attendu
![](lb2/afterDivaInstall.png)

---
## Étape 5 — Définir 3 scénarios simples (DIVA)

### Méthodologie
Ces scénarios sont des **parcours utilisateur** de référence. Ils sont **simples** et **répétables** (mêmes actions, mêmes entrées, pas de contenu aléatoire).  
Pour chaque scénario : on note le **module**, les **entrées exactes**, le **bouton cliqué** et le **résultat affiché**.

---

### Scénario 1 — Insecure Logging (action + message)
**Module :** `1. Insecure Logging`  
**Objectif :** lancer une action simple et observer le message affiché.

**Étapes :**
1. Ouvrir `1. Insecure Logging`
2. Laisser la valeur à `0`
3. Cliquer sur **CHECK OUT**
4. Observer le message en bas de l’écran

**Résultat observé :**
- Message : `An error occured. Please try again later`

**Capture :**
- Écran `1. Insecure Logging` avec **CHECK OUT** + message d’erreur.

![](lb2/snrio1.png)

---

### Scénario 2 — Input Validation Issues (action bouton)
**Module :** `13. Input Validation Issues - Part 3`  
**Objectif :** saisir une entrée et déclencher l’action principale.

**Entrée saisie (texte exact) :**
- Champ texte : `ew`

**Étapes :**
1. Ouvrir `13. Input Validation Issues - Part 3`
2. Saisir `ew`
3. Cliquer sur **PUSH THE RED BUTTON**
4. Observer le message affiché

**Résultat observé :**
- Message : `Access denied!`

**Capture :**
- Écran du module avec la valeur `ew` + bouton rouge + message.
 
![](lb2/snrio2.png)
---

### Scénario 3 — Insecure Data Storage (sauvegarde)
**Module :** `3. Insecure Data Storage - Part 1`  
**Objectif :** sauvegarder des identifiants de test et vérifier le retour.

**Entrées saisies (texte exact) :**
- Email : `ismai@emsi`
- Mot de passe : (valeur de test, masquée à l’écran)

**Étapes :**
1. Ouvrir `3. Insecure Data Storage - Part 1`
2. Saisir `ismai@emsi` + un mot de passe de test
3. Cliquer sur **SAVE**
4. Observer le message de confirmation

**Résultat observé :**
- Message : `3rd party credentials saved successfully!`

**Capture :**
- Écran du module avec les champs remplis + **SAVE** + message de succès.
![](lb2/snrio3.png)
---
## Étape 6 : Lire Android Security (6 lignes max)
La sécurité Android fonctionne en plusieurs couches, un peu comme un oignon.  
Le **sandboxing des applications** met chaque app dans son propre espace, pour limiter les interactions avec les autres.  
Le **modèle de permissions** sert de “filtre” : certaines ressources (caméra, contacts, stockage…) ne sont accessibles qu’après accord.  
L’**isolation** et l’**intégrité globale du système** visent à empêcher les retouches non autorisées du système.  
Analogie : chaque app est dans une salle fermée ; pour utiliser du matériel spécial, elle demande au prof ; et le bâtiment entier est protégé contre les modifications de sa structure.

---
## Étape 7 — Verified Boot (idée générale + check AVD)

### Check AVD (résultats observés)
- `adb shell getprop ro.boot.verifiedbootstate` : **vide** (propriété non exposée sur cette image AVD)
- `ro.boot.avb_version = 1.3`
- `ro.boot.veritymode = enforcing`
- `ro.boot.vbmeta.digest` présent (sha256)

### Interprétation (simple)
Même si `verifiedbootstate` n’est pas retourné, la présence de **AVB/vbmeta** et `veritymode=enforcing` indique que les mécanismes d’intégrité au démarrage sont actifs sur cet AVD.

### Résultat attendu
![](lb2/getGRp.png)

---
## Étape 8 : AVB (Android Verified Boot)
AVB est l’évolution moderne de Verified Boot : il renforce le contrôle d’intégrité au démarrage via des métadonnées signées (vbmeta).  
Il ajoute aussi une défense contre le **rollback**, pour éviter de revenir à une version plus ancienne du système.  
Protection anti-rollback : bloquer l’installation d’anciens builds potentiellement vulnérables, comme empêcher de remplacer une serrure récente par un modèle dépassé plus facile à forcer.

---
## Étape 9 : Définir le rooting (4 phrases)
Le **Root** correspond à l’obtention de privilèges **super-utilisateur** sur Android.  
Il change les barrières de sécurité et affaiblit la confiance accordée au système au démarrage et à l’exécution.  
En labo, c’est pratique pour observer des comportements et tester des contrôles normalement inaccessibles.  
C’est toutefois risqué : il faut un environnement isolé, garder des traces (logs/captures) et prévoir un reset, comme un passe-partout utile en maintenance mais dangereux s’il est mal utilisé ; le mot “root” vient d’UNIX où l’administrateur s’appelle root.

---
## Étape 10 : Intérêt labo (non opérationnel)
En labo, un environnement privilégié peut aider à **observer des artefacts système** normalement invisibles et à **suivre le comportement runtime** d’une application à un niveau plus bas.  
Il sert aussi à **éprouver la solidité du stockage** face à un attaquant privilégié (ex. fichiers, prefs, bases locales).  
Cas concret : avec les privilèges root, on peut vérifier si l’app compte seulement sur la protection du système (mauvaise pratique) ou si elle applique son propre chiffrement (bonne pratique).  
**Labo autorisé uniquement** : sur un appareil personnel/réel, cela peut casser la confiance du système et poser des problèmes de conformité.  
Contexte légal : selon le pays, le rooting peut enfreindre des conditions d’utilisation (voire des règles sur le contournement de protections), donc il faut une autorisation explicite pour tester.

---
