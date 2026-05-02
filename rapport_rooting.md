# Rapport de Lab — Android Rooting & Sécurité

---

## Étape 2 : Fiche périmètre

- App : app-debug.apk — version 1.0
- Support : AVD émulateur Android Studio
- Objectif : Comprendre le rooting et ses impacts sur la sécurité
- Données : Fictives uniquement
- Réseau : Test isolé

---

## Étape 3 : AVD propre

- AVD démarré via Android Studio → Device Manager
- Aucun compte personnel présent
- Commande vérification :

```
adb devices  →  emulator-5554 device ✅
```

---

## Étape 4 : Installation app de test

- App installée via `adb install app-debug.apk`
- Application lancée et fonctionnelle
- Version notée : 1.0

---

## Étape 5 : 3 scénarios de test

1. Ouvrir l'écran d'accueil de l'application
2. Rechercher un item dans l'application
3. Ouvrir le détail d'un item (fiche produit/profil)

---

## Étape 6 : Résumé Android Security

- **Sandboxing** : chaque application est isolée des autres applications
- **Modèle de permissions** : contrôle d'accès aux ressources sensibles du système
- **Isolation des processus** : chaque app tourne dans son propre processus Linux
- **Intégrité système** : protection contre les modifications non autorisées
- **Chiffrement** : protection des données stockées sur l'appareil
- **Mises à jour sécurité** : correctifs réguliers contre les vulnérabilités connues

---

## Étape 7 : Verified Boot

- **Objectif** : garantir que le système qui démarre est celui prévu par le fabricant
- **Chain of trust** : série de vérifications où chaque composant vérifie l'authenticité du suivant avant de lui faire confiance. Comme une chaîne de gardiens où chacun vérifie l'identité du suivant.
- L'intégrité au démarrage est critique : si le démarrage est compromis, toutes les protections ultérieures peuvent être contournées.

```
adb shell getprop ro.boot.verifiedbootstate  →  orange
```

---

## Étape 8 : AVB (Android Verified Boot)

- AVB est la version moderne de Verified Boot (v2.0)
- Ajoute une vérification d'intégrité plus robuste sur toutes les partitions
- Inclut une protection anti-rollback : empêche d'installer d'anciennes versions vulnérables du système

---

## Étape 9 : Définition du rooting

- Root désigne l'obtention des privilèges super-utilisateur sur Android.
- Cela modifie les protections système et compromet la confiance du système.
- En laboratoire, il est utile pour observer certains comportements sécurité.
- Risqué, il nécessite un environnement isolé, une traçabilité et un reset.

---

## Étape 10 : Intérêt labo (non opérationnel)

En labo, un environnement privilégié peut aider à :

- Observer des artefacts système normalement inaccessibles
- Analyser les comportements runtime de l'application à bas niveau
- Tester la robustesse du stockage face à un attaquant privilégié

> Note : labo autorisé uniquement.

---

## Étape 11 : Matrice de risques

1. Intégrité non garantie → conclusions biaisées sur la sécurité réelle
2. Surface d'attaque accrue si l'appareil sort du labo → exposition externe
3. Données sensibles exposées si présentes → violation de confidentialité
4. Instabilité système → tests non reproductibles et résultats incohérents
5. Mélange comptes perso/test → fuite possible d'informations personnelles
6. Mauvais nettoyage fin de séance → persistance de données sensibles
7. Réseau non isolé → effets involontaires sur systèmes externes
8. Traçabilité insuffisante → impossible de reproduire ou auditer les tests

---

## Étape 12 : Mesures défensives

1. Réseau isolé pour éviter toute communication non contrôlée
2. Données fictives uniquement pour éliminer tout risque de fuite réelle
3. Device/AVD dédié exclusivement aux tests de sécurité
4. Snapshots ou wipe en fin de séance pour ne laisser aucune trace
5. Journal de configuration détaillé pour assurer la reproductibilité
6. Aucun compte personnel pour éviter tout mélange de données
7. Contrôle strict des APK installées pour limiter les risques
8. Horodatage + captures des étapes pour une traçabilité complète

---

## Étape 13 : OWASP MASVS (2 exigences)

1. **STORAGE-1** : Les données sensibles (API keys, mots de passe, tokens) doivent être stockées de manière sécurisée avec un chiffrement approprié.
2. **NETWORK-1** : Les communications réseau doivent utiliser TLS avec une configuration correcte et vérifier les certificats.

---

## Étape 14 : OWASP MASTG (2 idées de tests)

1. Vérifier si les fichiers shared_prefs contiennent des données sensibles en clair dans `/data/data/[package_name]/shared_prefs/`
2. Analyser les logs avec `adb logcat` pour détecter des fuites d'informations sensibles pendant l'exécution de l'application.

---

## Étape 15 : Commandes rooting — Synthèse & Preuves

### Capture 1 — `adb root` + `adb remount` → Successfully disabled verity

![adb root + adb remount](1777680740411_image.png)

---

### Capture 2 — Remount complet (partitions /system, /vendor, /product en RW)

![adb remount complet](1777680747150_image.png)

---

### Capture 3 — `uid=0(root)` + `verifiedbootstate=orange` + `veritymode=enforcing`

![adb shell id + getprop](1777680754214_image.png)

---

### Capture 4 — `adb disable-verity` → Successfully disabled verity

![adb disable-verity](1777680760271_image.png)

---

### Récapitulatif des commandes

| Commande | Résultat |
|---|---|
| `adb devices` | emulator-5554 device ✅ |
| `adb root` | restarting adbd as root ✅ |
| `adb remount` | Remounted /system as RW ✅ |
| `adb shell id` | uid=0(root) ✅ |
| `adb shell getprop ro.boot.veritymode` | enforcing |
| `adb shell getprop ro.boot.verifiedbootstate` | orange |
| `adb shell getprop ro.boot.vbmeta.device_state` | (vide) |
| `adb shell "su -c id"` | erreur (normal émulateur Google) |
| `adb disable-verity` | Successfully disabled verity ✅ |

---

## Étape 16 : Fiche environnement (traçabilité)

| Champ | Valeur |
|---|---|
| Date/auteur | 26/04/2026 |
| Support | AVD émulateur Android Studio (emulator-5554) |
| Version Android/API | Android 16 / API 36 |
| App + version | app-debug.apk v1.0 |
| Scénario 1 | Ouvrir l'écran d'accueil de l'application |
| Scénario 2 | Rechercher un item dans l'application |
| Scénario 3 | Ouvrir le détail d'un item |
| Observations | uid=0(root) confirmé, veritymode=enforcing, verifiedbootstate=orange |
| Limites | su -c non supporté sur cet émulateur |
| Reset effectué | Oui |

---

## Étape 17 : Remise à zéro AVD

- Méthode : Android Studio → Device Manager → Wipe Data
- Résultat : AVD redémarré en état "neuf"
- Aucun compte personnel, aucune donnée résiduelle

---

## Étape 18 : Remise à zéro device labo

Non applicable (AVD uniquement, pas de device physique utilisé)

---

## Étape 19 : Livrables

- Définition rooting (4 phrases) ✅
- Verified Boot / AVB (chaîne de confiance) ✅
- 8 risques + 8 mesures défensives ✅
- MASVS : 2 exigences résumées ✅
- MASTG : 2 idées de tests ✅
- Fiche environnement remplie ✅
- Captures d'écran intégrées ✅
- Checklist reset signée + preuves ✅

---

## Étape 20 : Checklist finale

### Début de séance
- [x] Périmètre écrit
- [x] AVD neuf
- [x] App test installée
- [x] 3 scénarios notés
- [x] Versions Android/app notées (Android 16 / API 36)

### Fin de séance
- [x] Données de test supprimées
- [x] Reset effectué (Wipe AVD)
- [x] Preuve du reset (screenshot)
- [x] Rapport + traçabilité sauvegardés
- [x] Aucun compte personnel utilisé
