
# Rapport de Lab 10 : Guide d'installation et d'utilisation de Frida
**Spécialité :** Cyber Défense (ENSA Marrakech)  
**Auteur :** AMSOU ISMAIL  
**Date :** 27 Avril 2026  
**Cible :** Android Emulator (ia32) - Application : FragFlow

---

## 1. Introduction et Objectifs
Ce laboratoire porte sur la mise en place d'un environnement d'instrumentation dynamique utilisant Frida. L'objectif est d'intercepter des fonctions natives et Java au sein d'une application Android (`FragFlow`) pour analyser son comportement en matière de sécurité.

---

## 2. Étape 1 : Installation et Preuves (Client)
L'installation du client Frida et des outils associés a été effectuée sur l'hôte Windows.

### Vérification des versions
* **Frida version :** 17.5.1
* **Python version :** 3.11.2
* **ADB version :** 1.0.41

```bash
# Commande de preuve
python -c "import frida; print(frida.__version__)"
```
> **Capture d'écran 1 :** <img width="1365" height="329" alt="image" src="https://github.com/user-attachments/assets/38584a6f-5864-4b93-9cd0-95b8780f3f8c" />
Concernant ce chemin C:\Users\lenovo\Desktop\ㅤㅤㅤㅤ\frida, j’ai un dossier avec des caractères invisibles.
---

## 3. Étape 2 & 3 : Déploiement du Frida-Server sur Android
L'analyse de l'appareil via `adb` a révélé une architecture **x86 (ia32)**.

### Déploiement du binaire
1. **Transfert :** `adb push frida-server /data/local/tmp/`
2. **Permissions :** `adb shell chmod 755 /data/local/tmp/frida-server`
3. **Exécution :** Lancé en arrière-plan via `nohup`.

### Redirection des ports
```bash
adb forward tcp:27042 tcp:27042
adb forward tcp:27043 tcp:27043
```

### Vérification de la connexion (frida-ps)
La commande `frida-ps -Uai` montre que l'application cible est bien installée :
* **Nom :** FragFlow
* **Identifier :** `com.example.fragflow`

> **Capture d'écran 2 :** [Insérer l'image du résultat de frida-ps -Uai montrant FragFlow]

---

## 4. Étape 4 & 5 : Tests d'Injection et Validation
### 4.1 Injection Java (Script hello.js)
Validation de la capacité de Frida à interagir avec la JVM.
* **Sortie obtenue :** `[+] 3ssila.Frida Java.perform OK`

### 4.2 Hook Natif (Script 3ssila_native.js)
Interception de la fonction `recv` dans la `libc.so`.
* **Adresse de recv détectée :** `0xed0f3520`

> **Capture d'écran 3 :** [Insérer l'image de la console Frida affichant l'adresse de recv]

---

## 5. Étape 6 & 7 : Analyse de Sécurité (Exploration)
### 5.1 Énumération des classes Java
Filtrage des classes liées à l'application `FragFlow` :
* `com.example.fragflow.MainActivity`
* `com.example.fragflow.FragmentOne`
* `com.example.fragflow.FragmentTwo`

### 5.2 Bibliothèques de chiffrement détectées
Recherche des modules SSL/Crypto chargés :
* `libcrypto.so` (Base: 0xec847000)
* `libssl.so` (Base: 0xe5e84000)
* `libjavacrypto.so`

### 5.3 Observation du système de fichiers
Utilisation de `hook_file.js`. L'application a été observée ouvrant :
* `/proc/self/cmdline`
* `/data/app/com.example.fragflow-.../base.apk`
* `/data/user/0/com.example.fragflow/files/profileInstalled`

> **Capture d'écran 4 :** [Insérer l'image des logs de hook_file.js montrant les accès fichiers]

---

## 6. Étape 8 : Hooking Avancé (Java)
Tests réussis sur l'interception des APIs de haut niveau :
* **SharedPreferences :** Interception des lectures/écritures.
* **SQLite :** Chargement du script de surveillance des requêtes SQL.

---

## 7. Diagnostic et Dépannage (FAQ)
### Erreur SELinux
* **Problème :** `Failed to open file /sys/fs/selinux/policy: Permission denied`.
* **Diagnostic :** Droits insuffisants du shell ADB.
* **Résolution :** Exécution de `adb root` avant de relancer le serveur.

### Erreur TypeError dans le script JS
* **Problème :** `TypeError: not a function` à la ligne 5.
* **Diagnostic :** Utilisation d'une API Frida inexistante ou appel sur un pointeur nul.
* **Résolution :** Utilisation de `Module.findExportByName` (plus robuste) et ajout de vérifications `if (ptr !== null)`.

---

## 8. Conclusion
Ce lab a permis de valider l'installation de Frida et de confirmer son efficacité pour l'analyse d'applications Android. L'application `FragFlow` a été instrumentée avec succès, révélant ses accès aux fichiers système et ses dépendances cryptographiques natives.

**AMSOU ISMAIL**
```
