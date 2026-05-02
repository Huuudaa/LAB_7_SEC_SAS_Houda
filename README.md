# TP Lab 7 — Analyse Dynamique Mobile avec MobSF

## Description

Travail pratique d'analyse dynamique Android avec **MobSF (Mobile Security Framework)**.
Le lab couvre la configuration d'un émulateur Android propre sans Play Store, le
déploiement de MobSF via Docker, l'analyse statique et dynamique de l'application
vulnérable **DIVA (Damn Insecure and Vulnerable Android App)**, l'interception du
trafic réseau, la capture des logs runtime, et l'injection de scripts Frida depuis
l'interface MobSF.

---

## Objectifs pédagogiques

| Objectif | Description |
|---|---|
| Émulateur propre | Configurer un AVD sans Play Store compatible MobSF |
| MobSF Docker | Installer et lancer MobSF via Docker |
| Analyse statique | Scanner l'APK DIVA (manifest, permissions, code) |
| Analyse dynamique | Observer le comportement de l'app en temps réel |
| Trafic réseau | Intercepter HTTP/HTTPS avec le proxy MobSF |
| Frida | Injecter des scripts depuis l'interface MobSF |
| Rapport | Générer un rapport d'analyse complet |

---

## Glossaire

| Terme | Définition |
|---|---|
| **MobSF** | Mobile Security Framework, outil d'analyse statique et dynamique |
| **DIVA** | Damn Insecure and Vulnerable Android App, app de test vulnérable |
| **AVD** | Android Virtual Device, émulateur Android d'Android Studio |
| **Proxy HTTPS** | Intermédiaire qui intercepte et déchiffre le trafic SSL/TLS |
| **Frida** | Framework d'instrumentation dynamique |
| **Logcat** | Système de logs Android temps réel |
| **Exported Activity** | Composant Android accessible depuis d'autres applications |
| **Docker** | Plateforme de conteneurisation pour déployer des applications |

---

## Outils utilisés

| Outil | Version | Rôle |
|---|---|---|
| MobSF | Latest (Docker) | Analyse statique + dynamique |
| Docker Desktop | 4.x | Conteneurisation de MobSF |
| Android Studio | Meerkat | Création et gestion de l'AVD |
| ADB | 35.0.2 | Communication PC ↔ émulateur |
| DIVA APK | Beta | Application Android vulnérable cible |
| Git | 2.x | Clonage du repo MobSF |

---

## Application analysée

| Champ | Valeur |
|---|---|
| Nom | DIVA (Damn Insecure and Vulnerable App) |
| Package | `jakhar.aseem.diva` |
| Source | https://github.com/payatu/diva-android |
| Challenges | 13 vulnérabilités intentionnelles |
| Émulateur | AVD Google APIs x86_64, API 29 |

<img width="983" height="205" alt="image" src="https://github.com/user-attachments/assets/eafe0fbc-a6fd-4940-af35-54d1c79c0a54" />

---

## Pourquoi un émulateur sans Play Store ?

| Avantage | Explication |
|---|---|
| Pas de bruit de fond | Pas de Google Play Services qui génèrent du trafic parasite |
| Proxy propre | MobSF applique son proxy HTTPS sans conflit |
| Compatible root | MobSF peut rooter l'émulateur pour Frida |
| Performances | x86_64 optimisé pour PC |
| Limite | API 30 maximum (MobSF ne supporte pas les versions plus récentes) |

---

## Étape 1 — Création de l'émulateur AVD

Dans Android Studio :
```
Tools → Device Manager → Create Virtual Device
→ Pixel 5
→ System Image : API 29, x86_64, "Google APIs" SANS "Google Play"
→ Nom : MobSF_DIVA_API_30
→ Finish
```

**Vérification :**
```cmd
adb devices
```
<img width="813" height="115" alt="image" src="https://github.com/user-attachments/assets/ee7b6517-18a4-470f-99a5-3cbdca226d2c" />


---

## Étape 2 — Cloner MobSF

```cmd
git clone https://github.com/MobSF/Mobile-Security-Framework-MobSF.git
cd Mobile-Security-Framework-MobSF
```

<img width="1384" height="462" alt="image" src="https://github.com/user-attachments/assets/696de7bc-f51f-451a-8bb8-f6f6b55b53dd" />

---

## Étape 3 — Lancer l'émulateur avec le script MobSF

```powershell
.\scripts\start_avd.ps1
```
<img width="1070" height="191" alt="image" src="https://github.com/user-attachments/assets/a5945f62-0849-4a42-bc42-60d44d1bfcec" />

```cmd
adb devices
```
<img width="690" height="100" alt="image" src="https://github.com/user-attachments/assets/46ac8928-5d6c-4b6b-9b9d-a50b4eb0a12b" />

---

## Étape 4 — Lancer MobSF via Docker

```cmd
docker pull opensecurity/mobile-security-framework-mobsf:latest
```



```cmd
docker run -it --rm -p 8000:8000 -e MOBSF_ANALYZER_IDENTIFIER=emulator-5554 opensecurity/mobile-security-framework-mobsf:latest
```
<img width="1046" height="668" alt="image" src="https://github.com/user-attachments/assets/a2a8ac3c-7ccb-4e9f-83c2-f8d90af3c0c1" />


**Accès :**
```
URL      : http://127.0.0.1:8000
Login    : mobsf
Password : mobsf
```
<img width="1188" height="543" alt="image" src="https://github.com/user-attachments/assets/d61d4ac8-9337-431a-b262-24ca4c5de8ce" />

---

## Étape 5 — Analyse statique de DIVA

```
MobSF → Upload & Analyze → DIVA-debug.apk
```
<img width="728" height="622" alt="image" src="https://github.com/user-attachments/assets/0f12c350-4be5-41e7-b04a-f9494d8c6f20" />

### Résultats de l'analyse statique

| Élément | Valeur |
|---|---|
| Package | `jakhar.aseem.diva` |
| Version | 1.0 |
| minSdkVersion | 8 |
| targetSdkVersion | 22 |
| `android:debuggable` | `true` ⚠️ |
| `android:allowBackup` | `true` ⚠️ |
| Permissions | INTERNET, READ/WRITE_EXTERNAL_STORAGE |
| Activités exportées | 14 dont plusieurs sans protection |
| Score de sécurité | 35/100 ❌ |

<img width="1177" height="656" alt="image" src="https://github.com/user-attachments/assets/dcfe3998-1ec3-4ce8-b875-e09a56ee0f49" />


### Permissions dangereuses détectées

| Permission | Risque |
|---|---|
| `READ_EXTERNAL_STORAGE` | Accès à tous les fichiers du stockage |
| `WRITE_EXTERNAL_STORAGE` | Écriture de fichiers en dehors du sandbox |
| `INTERNET` | Communications réseau non restreintes |

---

## Étape 6 — Analyse dynamique

```
Rapport statique → Start Dynamic Analyzer
```

MobSF effectue automatiquement :
- ✅ Installation de DIVA sur l'émulateur
- ✅ Lancement de Frida Server
- ✅ Configuration du proxy HTTPS global
- ✅ Ouverture de l'interface Dynamic Analyzer
  
<img width="275" height="498" alt="image" src="https://github.com/user-attachments/assets/3273fbc5-6208-4834-883c-0041d89fcc4b" />

---

## Étape 7 — Exploration des challenges DIVA

### Challenge 1 — Insecure Logging

Action : DIVA → Insecure Logging → saisir `4111111111111111`

**MobSF Logcat Stream :**
```
D/DIVA: Credit card number: 4111111111111111
I/DIVA: User entered sensitive data in logs
```

**Vulnérabilité :** Données sensibles exposées dans les logs Android.

---

### Challenge 2 — Hardcoded Credentials

Action : DIVA → Hardcoded Credentials → saisir n'importe quelle valeur

**MobSF Runtime Logs :**
```
vendorCode = "Zgq4oLCsVN5ero5w3Cl9"
→ Secret codé en dur dans le bytecode
```

**Vulnérabilité :** Credentials statiques dans le code source.

---

### Challenge 3 — Insecure Data Storage (Part 1)

Action : DIVA → Insecure Data Storage → saisir `admin / password123`

**MobSF File Monitor :**
```
/data/data/jakhar.aseem.diva/shared_prefs/
  jakhar.aseem.diva_preferences.xml
  → username=admin
  → password=password123  (en clair !)
```

**Vulnérabilité :** SharedPreferences stockées sans chiffrement.

---

### Challenge 4 — Insecure Network Traffic

Action : DIVA → fonctions réseau → observer

**MobSF Network Traffic :**
```
GET http://dirtyunicorns.com/wp-json/wp/v2/posts HTTP/1.1
Host: dirtyunicorns.com
→ Trafic HTTP non chiffré intercepté ⚠️
```

**Vulnérabilité :** Communications réseau non chiffrées (HTTP au lieu de HTTPS).

---

### Challenge 5 — Access Control

Action : DIVA → Access Control Issues

**MobSF Exported Activity Tester :**
```
jakhar.aseem.diva.APICredsActivity      → accessible sans auth ⚠️
jakhar.aseem.diva.InsecureDataStorage3  → accessible sans auth ⚠️
```

**Vulnérabilité :** Activités exportées sans contrôle d'accès.

---

## Étape 8 — Menu Dynamic Analyzer

| Menu | Action effectuée | Résultat observé |
|---|---|---|
| **Activity Tester** | Clic | Liste de toutes les activités lancées sur l'émulateur |
| **Exported Activity Tester** | Clic | 14 activités testées, 6 sans protection |
| **TLS/SSL Security Tester** | Clic | Certificat MobSF accepté, SSL pinning absent |
| **Logcat Stream** | Clic | Logs en temps réel avec données sensibles visibles |
| **Take a Screenshot** | Clic | Capture de l'écran DIVA pour documentation |
| **Get Dependencies** | Clic | Bibliothèques et composants identifiés |
| **Generate Report** | Clic | Rapport PDF/JSON généré avec tous les résultats |

---

## Étape 9 — Injection Frida depuis MobSF

Dans MobSF → **Frida** → **Spawn & Inject** :

```javascript
Java.perform(function () {
  console.log("[+] Frida injecté depuis MobSF");

  var SharedPreferences = Java.use("android.app.SharedPreferencesImpl");
  SharedPreferences.getString.overload(
    "java.lang.String", "java.lang.String"
  ).implementation = function (key, defValue) {
    var result = this.getString(key, defValue);
    console.log("[SharedPrefs] key=" + key + " => " + result);
    return result;
  };
});
```

**Résultat dans MobSF :**
```
[+] Frida injecté depuis MobSF
[SharedPrefs] key=username => admin
[SharedPrefs] key=password => password123
```

---

## Étape 10 — Rapport final

```
MobSF Dynamic Analyzer → Generate Report
```

**Contenu du rapport généré :**
- Logs runtime capturés (Logcat)
- Trafic réseau intercepté (HTTP + HTTPS déchiffré)
- Fichiers accédés pendant l'exécution
- Activités exportées testées
- Screenshots de chaque étape
- Score de sécurité global

---

## Récapitulatif des vulnérabilités détectées

| # | Vulnérabilité | Challenge | Sévérité |
|---|---|---|---|
| 1 | Données sensibles dans les logs | Insecure Logging | **Élevée** |
| 2 | Credentials codés en dur | Hardcoded Credentials | **Élevée** |
| 3 | SharedPreferences en clair | Insecure Data Storage | **Élevée** |
| 4 | Trafic HTTP non chiffré | Insecure Network | **Moyenne** |
| 5 | Activités exportées sans auth | Access Control | **Moyenne** |
| 6 | `android:debuggable=true` | Manifest | **Élevée** |
| 7 | `android:allowBackup=true` | Manifest | **Moyenne** |

---

## Nettoyage

```cmd
docker stop $(docker ps -q)
adb emu kill
```

---

## Checklist finale

- [x] AVD créé sans Google Play (API 29, x86_64)
- [x] MobSF cloné depuis GitHub
- [x] Émulateur lancé avec le script MobSF
- [x] MobSF démarré via Docker sur le port 8000
- [x] DIVA uploadé et analysé (statique + dynamique)
- [x] 5 challenges DIVA explorés avec logs capturés
- [x] Trafic réseau intercepté
- [x] Script Frida injecté depuis MobSF
- [x] Rapport final généré
- [x] Nettoyage effectué

---

## Environnement

- **OS** : Windows 11
- **Android Studio** : Meerkat
- **Émulateur** : AVD Google APIs x86_64, API 29 (sans Google Play)
- **MobSF** : Latest (Docker)
- **ADB** : 35.0.2
- **APK analysé** : DIVA Beta (jakhar.aseem.diva)
- **Niveau** : Intermédiaire
- **Catégorie** : Mobile Security / Dynamic Analysis / Network Interception
