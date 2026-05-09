# Rapport de Laboratoire : Bypass du SSL Pinning sur Android (Lab 15)

**Étudiant :** Ziyad Daber
**Date :** 9 mai 2026
**Objectif :** Intercepter et analyser le trafic HTTPS d'une application Android en neutralisant les mécanismes de SSL Pinning via l'instrumentation dynamique.

---

## 1. Configuration de l'Environnement
Pour réaliser ce laboratoire, j'ai mis en place la chaîne d'outils suivante :
- **Hôte :** PC Windows (PowerShell) avec Python 3.8+.
- **Cible :** Appareil Android 8+ avec le Débogage USB activé.
- **Proxy TLS :** Burp Suite Community Edition (configuré pour intercepter le trafic).
- **Instrumentation :** Frida (Client PC et `frida-server` sur Android) avec des versions strictement alignées.
- **Connectivité :** ADB configuré et certificat CA du proxy installé sur l'appareil.

## 2. Déploiement et Mise en Route
Le déploiement a suivi les étapes techniques suivantes :
1. **Lancement du Serveur :** Identification de l'architecture CPU via `adb shell getprop ro.product.cpu.abi`, transfert du binaire `frida-server` vers `/data/local/tmp/` et exécution en arrière-plan.
2. **Vérification :** Validation de la communication entre le PC et l'appareil via la commande `frida-ps -Uai`.
3. **Configuration Réseau :** Configuration du proxy Android pour pointer vers l'adresse IP du PC et le port de Burp Suite.

## 3. Stratégies de Contournement du SSL Pinning

### 3.1 Bypass au niveau Java (High-Level)
J'ai injecté des scripts Frida pour neutraliser les implémentations courantes de pinning :
- **TrustManager & Conscrypt :** Modification des méthodes de vérification des certificats pour forcer l'acceptation de tous les certificats, y compris celui de Burp Suite.
- **OkHttp :** Hooking des `CertificatePinner` pour ignorer les erreurs de correspondance de certificats.
- **WebView :** Interception de `onReceivedSslError` pour forcer la poursuite du chargement de la page malgré l'erreur SSL.

### 3.2 Cas Avancés : Pinning Natif (BoringSSL/OpenSSL)
Pour les applications utilisant des bibliothèques natives, j'ai appliqué les techniques suivantes :
- **Analyse Native :** Utilisation de `frida-trace` pour identifier les appels aux fonctions `SSL_set_custom_verify` et `SSL_CTX_set_custom_verify`.
- **Hooking Libc :** Interception des appels système pour contourner la vérification du certificat au niveau du binaire.

## 4. Validation et Résultats
- **Sans Bypass :** L'application refusait de se connecter au serveur dès l'activation du proxy, affichant une erreur de connexion sécurisée (SSL Handshake failure).
- **Avec Bypass :** Après injection des scripts de neutralisation, la connexion a été établie avec succès. 
- **Résultat final :** Le trafic HTTPS a été déchiffré et intercepté dans Burp Suite, permettant l'analyse complète des requêtes et réponses API.

## 5. Analyse du Dépannage et Optimisations
- **Gestion des Crashs :** Pour éviter les crashs au démarrage avec l'option `-f` (spawn), j'ai utilisé l'option `-n` (attach) pour injecter le script une fois l'application stabilisée.
- **Obfuscation :** Face aux packages renommés, j'ai utilisé `Java.enumerateLoadedClasses` pour filtrer les classes contenant les mots-clés "okhttp" ou "trust" afin d'adapter les hooks.
- **Cronet/Chromium :** Identification d'une pile réseau spécifique nécessitant un traçage des fonctions `SSL_*` en natif.

---
**Conclusion :** Le Lab 15 a été accompli avec succès. Je maîtrise désormais le déploiement de Frida, la configuration d'un proxy TLS et les différentes méthodes (Java et Natives) pour contourner le SSL Pinning sur Android.

**Statut :** $\text{TERMINÉ} \quad \checkmark$
