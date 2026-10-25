# 🛡️ anti-utiq-filters

> **Règles de filtrage personnalisées pour uBlock Origin, AdGuard, AdBlock et AdBlock Plus, destinées à bloquer le réseau de tracking Utiq.**

Ce dépôt fournit des règles de filtrage optimisées conçues pour neutraliser **Utiq** — le service de tracking au niveau réseau (souvent appelé « cookie télécom ») opéré par de grands opérateurs télécoms européens, dont Orange, SFR, Bouygues, Vodafone, et d'autres.

L'objectif est de bloquer les requêtes de tracking, qu'elles proviennent de l'infrastructure centrale d'Utiq ou de sous-domaines hébergés chez ses partenaires (`utiq.orange.fr`, `utiq.france.tv`, `utiq.allocine.fr`, etc.).

> 🇬🇧 *A [version of this README in English](README.md) is also available.*

---

## 📖 Sommaire

- [Fichiers du dépôt](#-fichiers-du-dépôt)
- [Installation](#-installation)
  - [Pour uBlock Origin & AdGuard](#1️⃣-pour-ublock-origin--adguard-filters-ubo-adguardtxt)
  - [Pour AdBlock & AdBlock Plus](#2️⃣-pour-adblock--adblock-plus-filters-abptxt)
- [Analyse technique](#-analyse-technique)
  - [Infrastructure centrale](#1-infrastructure-centrale)
  - [Interception des scripts & API](#2-interception-des-scripts--api)
  - [Blocage des sous-domaines partenaires](#3-blocage-des-sous-domaines-partenaires)
  - [Blocage de l'iframe inter-domaines](#4-blocage-de-liframe-inter-domaines)
- [Infrastructure Utiq connue](#️-infrastructure-utiq-connue)
- [Compatibilité](#-compatibilité)
- [Vérification](#️-vérification)
- [Défense en profondeur](#️-défense-en-profondeur)
- [Limites connues & mesures complémentaires](#️-limites-connues--mesures-complémentaires)
- [Contribuer](#-contribuer)
- [Changelog](#-changelog)
- [Licence](#-licence)

---

## 📁 Fichiers du dépôt

| Fichier | Cible | Lien brut |
|---|---|---|
| `filters-ubo-adguard.txt` | uBlock Origin, AdGuard | [Voir le brut](https://raw.githubusercontent.com/Polyum/anti-utiq-filters/main/filters-ubo-adguard.txt) |
| `filters-abp.txt` | AdBlock, AdBlock Plus | [Voir le brut](https://raw.githubusercontent.com/Polyum/anti-utiq-filters/main/filters-abp.txt) |

---

## 📥 Installation

Choisissez la liste de filtres adaptée à votre extension.

### 1️⃣ Pour uBlock Origin & AdGuard (`filters-ubo-adguard.txt`)

Optimisée pour les moteurs de filtrage modernes prenant en charge le wildcard avancé et les modificateurs courts.

#### ✅ Recommandé — S'abonner au fichier brut (mise à jour auto tous les 2 jours)

Les fichiers de filtres incluent une directive de métadonnée `Expires: 2 days`. Votre extension récupérera automatiquement la dernière version toutes les 2 jours, sans intervention manuelle.

**uBlock Origin :** Paramètres → Listes de filtres → Importer → coller l'URL ci-dessous → Appliquer les changements

**AdGuard :** Paramètres → Filtres → Personnalisé → Ajouter un filtre personnalisé → coller l'URL ci-dessous

~~~
https://raw.githubusercontent.com/Polyum/anti-utiq-filters/main/filters-ubo-adguard.txt
~~~

#### 🔁 Alternative — Copier/coller manuel (pas de mise à jour auto)

Copiez-collez les règles ci-dessous dans l'éditeur **« Mes filtres »** / **« Filtres personnalisés »** de votre extension. Cette méthode ne se met pas à jour automatiquement — il faudra vérifier manuellement les nouvelles versions.

~~~text
! Title: anti-utiq-filters (uBlock Origin & AdGuard)
! Description: Custom filtering rules to block the Utiq tracking network.
! Homepage: https://github.com/Polyum/anti-utiq-filters
! Expires: 2 days
! Version: 1.6

! 1. Central Infrastructure
||utiq.com^
||utiq-aws.net^
||utiqcontent.com^
||mno.link^

! 2. Script & API Interception
/utiqLoader.js$script,important
/utiqConsentManager.js$script,important
/op/idconnect/$xhr,subdocument,important
/op/privacy/$xhr,important

! 3. Partner-Hosted Subdomain Blocking
! Blocks utiq.orange.fr, utiq.france.tv, utiq.allocine.fr, etc.
! The * wildcard matches any parent domain, regardless of TLD.
||utiq.*^

! 4. Cross-Domain Iframe Blocking
##iframe[name="__utiqLocator"]
~~~

---

### 2️⃣ Pour AdBlock & AdBlock Plus (`filters-abp.txt`)

Version rétrocompatible, adaptée aux règles d'analyse plus strictes des moteurs historiques. Le modificateur `$important` est volontairement omis — c'est une extension propre à uBlock Origin, non prise en charge par les moteurs ABP, qui rendrait ces règles silencieusement ignorées ou rejetées.

#### ✅ Recommandé — S'abonner au fichier brut (mise à jour auto tous les 2 jours)

**AdBlock / AdBlock Plus :** Options → Listes de filtres → Ajouter une liste de filtres → coller l'URL ci-dessous

~~~
https://raw.githubusercontent.com/Polyum/anti-utiq-filters/main/filters-abp.txt
~~~

#### 🔁 Alternative — Copier/coller manuel (pas de mise à jour auto)

~~~text
! Title: anti-utiq-filters (AdBlock & AdBlock Plus)
! Description: Custom filtering rules to block the Utiq tracking network.
! Homepage: https://github.com/Polyum/anti-utiq-filters
! Expires: 2 days
! Version: 1.6

! 1. Central Infrastructure
||utiq.com^
||utiq-aws.net^
||utiqcontent.com^
||mno.link^

! 2. Script & API Interception
/utiqLoader.js$script
/utiqConsentManager.js$script
/op/idconnect/$xmlhttprequest,subdocument
/op/privacy/$xmlhttprequest

! 3. Partner-Hosted Subdomain Blocking
! Adapted for ABP: matches 'utiq.' at any domain or subdomain boundary
||utiq.

! 4. Cross-Domain Iframe Blocking
##iframe[name="__utiqLocator"]
~~~

---

## 🔍 Analyse technique

### 1. Infrastructure centrale

**`||utiq.com^`**
L'ancre `||` cible le domaine racine **et tous ses sous-domaines, de façon récursive**. Cette seule règle bloque tout le trafic vers des nœuds comme `consenthub.utiq.com`, `service.utiq.com`, `api.utiq.com` ou `cdn.utiq.com`. S'applique aux quatre moteurs.

**`||utiq-aws.net^`**
Le backend de production d'Utiq tourne sur AWS sous ce domaine dédié. Bloquer la racine coupe l'accès à la passerelle actuelle (`frontend.prod`) et couvre par avance tout futur sous-domaine que le réseau pourrait déployer. Notez que `utiq-aws.net` commence par `utiq-` (tiret, pas point) et n'est donc **pas** intercepté par les règles de sous-domaines partenaires — cette entrée explicite est nécessaire. S'applique aux quatre moteurs.

**`||utiqcontent.com^`**
Un domaine racine distinct utilisé par le SDK Utiq pour charger des ressources statiques (fichiers de configuration et SVG). Indépendant de `utiq.com` et `utiq-aws.net`, il ne serait intercepté par aucune autre règle. S'applique aux quatre moteurs.

**`||mno.link^`**
Pendant le handshake avec l'opérateur mobile (MNO), Utiq fait transiter les requêtes d'identification de l'abonné par des sous-domaines propres à chaque opérateur sous `mno.link` (`orangefr.mno.link`, `o2de.mno.link`, `vfde-fixed.mno.link`, etc.), qui pointent par CNAME vers `frontend.prod.utiq-aws.net`. Comme ces domaines ne commencent pas par `utiq.`, ils ne sont pas interceptés par les règles de sous-domaines partenaires, et les navigateurs sans démasquage CNAME natif (comme les navigateurs basés sur Chromium) ne voient pas la cible `utiq-aws.net` sous-jacente. Bloquer `mno.link` directement comble cet angle mort. S'applique aux quatre moteurs.

---

### 2. Interception des scripts & API

**`/utiqLoader.js$script,important`** *(uBlock Origin & AdGuard)*
**`/utiqLoader.js$script`** *(AdBlock & AdBlock Plus)*
Cible le script d'initialisation d'Utiq par son chemin, quel que soit le domaine hôte qui tente de l'injecter. Le modificateur `$important` n'est disponible que dans uBO et AdGuard : il assigne une priorité stricte, empêchant des règles d'autorisation propres à un site de contourner le blocage. Il est volontairement absent de la variante ABP, où il n'est pas pris en charge et invaliderait silencieusement la règle.

**`/utiqConsentManager.js$script,important`** *(uBlock Origin & AdGuard)*
**`/utiqConsentManager.js$script`** *(AdBlock & AdBlock Plus)*
Cible par son chemin le script de gestion du consentement d'Utiq. Il est normalement chargé par `utiqLoader.js`, mais le SDK Utiq permet aux éditeurs de le charger depuis une origine personnalisée distincte via l'option de configuration `consentManagerOrigin`. Le bloquer explicitement garantit la couverture dans tous les cas de figure.

**`/op/idconnect/$xhr,subdocument,important`** *(uBlock Origin & AdGuard)*
**`/op/idconnect/$xmlhttprequest,subdocument`** *(AdBlock & AdBlock Plus)*
Intercepte toute la famille de points d'accès `/op/idconnect/` (`/op/idconnect/mno-precheck`, `/op/idconnect/data`, `/op/idconnect/mno-selector`, `/op/idconnect/tmi-*`), utilisée pour interroger le réseau mobile ou fixe et récupérer en arrière-plan les tokens d'abonné. En plus des appels XHR/Fetch (`$xhr` dans uBO/AdGuard ; `$xmlhttprequest` dans AdBlock/ABP), Utiq charge plusieurs de ces points d'accès dans des iframes cachées, interceptées par le modificateur `$subdocument`. L'interception au niveau du chemin neutralise aussi les hôtes CNAME personnalisés configurés via `customUtiqHost`.

**`/op/privacy/$xhr,important`** *(uBlock Origin & AdGuard)*
**`/op/privacy/$xmlhttprequest`** *(AdBlock & AdBlock Plus)*
Bloque les points d'accès de synchronisation de la confidentialité et du consentement d'Utiq (`/op/privacy/info` et `/op/privacy/permission`) sur tous les domaines partenaires.

---

### 3. Blocage des sous-domaines partenaires

Pour étendre leur portée, les partenaires opérateurs et médias d'Utiq provisionnent un sous-domaine `utiq.` directement sur leurs propres domaines :

| Sous-domaine partenaire | Domaine parent |
|---|---|
| `utiq.orange.fr` | `orange.fr` |
| `utiq.france.tv` | `france.tv` |
| `utiq.allocine.fr` | `allocine.fr` |
| `utiq.lefigaro.fr` | `lefigaro.fr` |
| … | … |

Ces noms d'hôtes n'apparaissent pas dans les listes de blocage génériques ciblant `utiq.com`, car ils sont servis sous des domaines enregistrables entièrement différents.

**`||utiq.*^`** *(uBlock Origin & AdGuard)*
L'ancre `||` correspond au début d'un nom d'hôte ou après un séparateur `.`. Le wildcard `*` absorbe tout le domaine parent :

- `utiq.orange.fr` → `||` s'ancre en position 0, `utiq.` correspond, `*` absorbe `orange.fr` ✅
- `utiq.france.tv` → même mécanisme ✅
- `cdn.utiq.orange.fr` → `||` s'ancre après le premier `.`, `utiq.orange.fr` est reconnu ✅

**`||utiq.`** *(AdBlock & AdBlock Plus)*
ABP ne gère pas de façon fiable la combinaison wildcard `.*^` à cette position. Le `^` final et le wildcard sont supprimés : `||utiq.` correspond à toute requête où `utiq.` apparaît à une frontière de domaine, couvrant tous les sous-domaines partenaires avec une efficacité équivalente.

---

### 4. Blocage de l'iframe inter-domaines

**`##iframe[name="__utiqLocator"]`**
Lorsque des éditeurs font tourner de la publicité programmatique (Prebid.js v10.11+) dans des iframes inter-domaines, le SDK Utiq injecte dans le DOM parent une iframe vide nommée `__utiqLocator`, pour signaler la présence d'identifiants Utiq et répercuter les tokens `adtechpass` (`atid`) et `martechpass` (`mtid`) vers les frames enfants.

Il s'agit d'une **règle cosmétique : elle masque l'élément, elle n'empêche ni sa création ni son fonctionnement.** L'iframe est injectée par JavaScript ; la vraie protection vient donc du blocage du SDK en amont (`/utiqLoader.js`, `||utiq.com^`, etc. — voir sections 1 et 2) : si le loader ne s'exécute pas, le locator n'est jamais injecté. Considérez cette règle comme un filet de sécurité cosmétique, pas comme une protection en soi. Prise en charge par les quatre moteurs.

---

## 🗺️ Infrastructure Utiq connue

Cette section documente les domaines et points d'accès centraux connus d'Utiq, à titre de référence pour les contributeurs qui surveillent de nouveaux déploiements d'infrastructure.

> Pour une liste complète et maintenue des sous-domaines partenaires, référez-vous à la blocklist communautaire : [utiq-tracker.online](https://utiq-tracker.online/)

### Domaines racines
| Domaine | Rôle |
|---|---|
| `utiq.com` | Infrastructure principale — hub de consentement, API, CDN |
| `utiq-aws.net` | Backend de production AWS |
| `utiqcontent.com` | Ressources statiques — fichiers de configuration du SDK et SVG |
| `mno.link` | Passerelles de handshake MNO propres à chaque opérateur (`orangefr.mno.link`, `o2de.mno.link`, etc.) |

### Sous-domaines connus
| Sous-domaine | Rôle |
|---|---|
| `consenthub.utiq.com` | Portail de gestion du consentement utilisateur |
| `service.utiq.com` | Service backend du hub de consentement |
| `frontend.prod.utiq-aws.net` | Passerelle API de production — cible CNAME de tous les sous-domaines partenaires et de `*.mno.link` |

### Points d'accès API connus
| Chemin | Rôle |
|---|---|
| `/op/idconnect/mno-precheck` | Handshake Mobile Network Operator — validation de l'éligibilité de l'abonné |
| `/op/idconnect/data` | Récupération du token d'abonné et du `connectId` (XHR et iframe) |
| `/op/idconnect/mno-selector` | Point d'accès de sélection de l'opérateur télécom |
| `/op/idconnect/tmi-*` | Points d'accès d'identification propres à chaque opérateur (ex. `/op/idconnect/tmi-o2de`) |
| `/op/privacy/info` | Point d'accès des métadonnées de confidentialité |
| `/op/privacy/permission` | Point d'accès de synchronisation des permissions de consentement |

### Scripts connus
| Nom de fichier | Rôle |
|---|---|
| `utiqLoader.js` | Script d'initialisation principal injecté sur les sites partenaires |
| `utiqConsentManager.js` | Script de la bannière de consentement, chargé par `utiqLoader.js` ou via un `consentManagerOrigin` personnalisé |

### Cookies first-party et clés de Local Storage connus
| Type | Nom | Rôle |
|---|---|---|
| Cookie | `connectId` | Met en cache le signal du réseau de l'opérateur (90 jours) pour éviter d'interroger le FAI à chaque visite |
| Cookie | `utiq_consent_status` | Stocke l'état du consentement de l'utilisateur encodé en base64 (`true` / `false`, jusqu'à 180 jours) |
| Cookie | `utiq_consent_version` | Enregistre la version du consentement acceptée par l'utilisateur (390 jours) |
| Cookie | `utiq_logging_level` | Stocke le niveau de journalisation de débogage du SDK |
| `localStorage` | `utiqEligibility` | Stocke l'éligibilité auprès du FAI et le statut de révocation ConsentHub |
| `localStorage` | `utiqDataCallTimestamp` | Horodatage de la dernière vérification d'éligibilité MNO |
| `localStorage` | `utiqPass` | Stocke les tokens générés `martechpass` (`mtid`), `adtechpass` (`atid`) et `attributionpass` (`attrid`) |
| `localStorage` | `utiq_openrtb_eids` | Tableau JSON OpenRTB sérialisé des identifiants Utiq, lu directement par les modules Prebid.js (`utiqIdSystem` / `utiqMtpIdSystem`) |

---

## ✅ Compatibilité

| Règle | uBlock Origin | AdGuard | AdBlock | AdBlock Plus |
|---|---|---|---|---|
| `\|\|domaine^` (racine + sous-domaines) | ✅ | ✅ | ✅ | ✅ |
| Modificateur `$script` | ✅ | ✅ | ✅ | ✅ |
| Modificateur `$xhr` | ✅ | ✅ | ❌ | ❌ |
| Modificateur `$xmlhttprequest` | ✅ | ✅ | ✅ | ✅ |
| Modificateur `$subdocument` | ✅ | ✅ | ✅ | ✅ |
| Modificateur `$important` | ✅ | ✅ | ❌ | ❌ |
| `\|\|utiq.*^` (wildcard) | ✅ | ✅ | ❌ | ❌ |
| `\|\|utiq.` (correspondance de frontière) | ✅ | ✅ | ✅ | ✅ |
| `##iframe[name="..."]` (cosmétique — masque uniquement) | ✅ | ✅ | ✅ | ✅ |
| Démasquage CNAME (fonctionnalité du navigateur, **pas** une règle de filtre) | ✅ Firefox uniquement | — | — | — |

> `—` = aucun équivalent vérifié pour ce projet.
>
> Ces règles s'appliquent uniquement au filtrage **côté navigateur** (extension requise). Elles n'ont aucun effet sur les applications mobiles natives ou le trafic au niveau système.

---

## 🛠️ Vérification

Pour confirmer que les filtres sont actifs :

1. Naviguez vers un site partenaire utilisant le réseau Utiq.
2. Ouvrez le **Network Logger** de votre extension (tableau de bord uBlock Origin → onglet Logger ; AdGuard → journal de filtrage).
3. Recherchez `utiq` ou `idconnect`. Toutes les requêtes liées (`utiqLoader.js`, `utiqConsentManager.js`, `/op/idconnect/*`, `*.mno.link`, sous-domaines partenaires, etc.) doivent apparaître en **rouge** (bloquées), confirmant que le handshake a été intercepté avant toute transmission de données à l'opérateur.

---

## 🛡️ Défense en profondeur

Aucune couche unique n'offre une protection complète. Les angles morts de chaque couche sont couverts par les autres : empilez-en autant que votre configuration le permet.

| Couche | Ce qu'elle protège | Angles morts |
|---|---|---|
| Filtres (ce projet) | Requêtes HTTP, scripts, appels API, iframes | Navigateur uniquement ; inefficace dans les applications natives |
| Opt-out légal | Révocation du consentement au niveau opérateur | Peut être annulée accidentellement en cliquant sur « Accepter » sur un site partenaire |
| VPN | Niveau réseau — empêche entièrement la génération du token | Actif uniquement quand le tunnel VPN est établi |
| Blocklist DNS | Tous les appareils du réseau | Aveugle au CNAME cloaking ; ne peut pas bloquer par chemin ou type de requête |

### Couche 1 — Filtres navigateur (ce projet)
Ces règles interceptent le script, le handshake API et les sous-domaines partenaires d'Utiq au niveau HTTP. Les règles basées sur un chemin (`/utiqLoader.js`, `/utiqConsentManager.js`, `/op/idconnect/`, `/op/privacy/`) restent efficaces même contre le CNAME cloaking, contrairement au blocage au niveau DNS — sauf si un partenaire renomme aussi ces chemins. Voir [Limites connues](#️-limites-connues--mesures-complémentaires).

### Couche 2 — Opt-out légal (niveau opérateur)
Révoquez formellement votre consentement au niveau de l'infrastructure opérateur via le portail officiel de confidentialité d'Utiq :
👉 **[consenthub.utiq.com](https://consenthub.utiq.com/)**

### Couche 3 — VPN (niveau réseau)
Un VPN est la protection la plus fondamentale contre Utiq : le token est généré par votre opérateur via une injection au niveau réseau, liée à votre adresse IP et à votre identité d'abonné. Faire transiter votre trafic par un serveur VPN rend cette identification impossible — le fournisseur VPN n'est pas un partenaire opérateur, et votre FAI ne voit qu'un tunnel chiffré, sans pouvoir associer votre session à votre identifiant réseau. Cette protection est indépendante de la configuration du navigateur et couvre tous les appareils et applications sur la connexion.

> ⚠️ La protection n'est active que tant que le tunnel VPN est établi. Si votre appareil bascule sur une connexion directe (données mobiles, ou Wi-Fi sans VPN), Utiq peut reprendre son fonctionnement normal — c'est précisément pour cela qu'empiler plusieurs couches est important.

### Couche 4 — Blocklist DNS communautaire (maintenance automatisée)
Abonnez-vous à une blocklist maintenue par la communauté pour recevoir des mises à jour automatiques de domaines sans intervention manuelle :
- 📋 **URL d'abonnement :** [utiq-tracker.online/blocklists/utiq-standard-adblock.txt](https://utiq-tracker.online/blocklists/utiq-standard-adblock.txt)
- 🌐 **Référence du projet :** [utiq-tracker.online](https://utiq-tracker.online/)

> ⚠️ Vérifiez toujours la fiabilité d'une blocklist tierce avant de vous y abonner.

> 💡 **Utilisateurs de Pi-hole / AdGuard Home :** les filtres de ce projet reposent sur des règles au niveau HTTP (chemins de script, points d'accès API, types de requêtes) que les bloqueurs DNS ne peuvent pas traiter. La blocklist communautaire ci-dessus peut offrir des entrées DNS mieux adaptées à votre configuration.

---

## ⚠️ Limites connues & mesures complémentaires

Ces filtres arrêtent Utiq tant que son trafic est identifiable, soit par son nom d'hôte (`utiq.*`, `utiq.com`, `utiq-aws.net`, `utiqcontent.com`, `mno.link`), soit par le chemin du script ou du point d'accès. Deux situations dépassent ce qu'une liste de filtres statique peut résoudre entièrement.

### 1. CNAME cloaking derrière un nom d'hôte first-party arbitraire

Un partenaire peut exposer Utiq sous un nom d'hôte de son choix (par exemple `stats.example.com`) dont le CNAME DNS pointe vers `frontend.prod.utiq-aws.net`. Le moteur de filtrage ne voit alors que `stats.example.com` : **les règles de domaine ne peuvent pas correspondre** dans ce scénario, et aucune règle de domaine supplémentaire n'y changerait rien, puisque ce nom d'hôte n'appartient pas à Utiq.

Ce qui vous protège quand même :

- Les **règles de chemin** (`/utiqLoader.js`, `/utiqConsentManager.js`, `/op/idconnect/`, `/op/privacy/`) ne dépendent pas du nom d'hôte et continuent de fonctionner, sauf si un partenaire renomme aussi ces chemins.
- **uBlock Origin sur Firefox** peut résoudre lui-même le CNAME et réappliquer les filtres au vrai nom d'hôte, où `||utiq-aws.net^` correspond. Dans uBO, il s'agit de l'option **« Démasquer les noms canoniques »** (Paramètres → Confidentialité), qui est **activée par défaut**. Si vous l'avez désactivée, réactivez-la. Elle ne fonctionne que sur Firefox : elle s'appuie sur une API DNS que Firefox expose aux extensions et que les navigateurs basés sur Chromium n'exposent pas.

> Le trou restant est donc étroit mais réel : un **navigateur autre que Firefox** face à un partenaire qui utilise un nom d'hôte first-party arbitraire **et** des chemins renommés. Aucune liste de filtres ne peut le combler ; la couche VPN ci-dessus, si.

### 2. Données déjà stockées et fingerprinting complémentaire

- **Les données locales existantes ne sont pas effacées par les filtres.** Les filtres arrêtent le SDK et ses appels API, mais les valeurs stockées avant leur installation (le cookie `connectId`, `utiqPass` et les autres clés listées dans [Cookies first-party et clés de Local Storage connus](#cookies-first-party-et-clés-de-local-storage-connus)) restent dans votre navigateur tant que vous ne les supprimez pas. Videz une fois les données des sites utilisant Utiq après avoir installé les filtres.
- **Fingerprinting.** Des rapports indépendants indiquent que les sites utilisant Utiq le combinent souvent avec du fingerprinting par canvas et par police. Cela s'exécute côté client, dans des scripts par ailleurs légitimes : aucun filtre réseau, qu'il soit basé sur le domaine ou sur le chemin, ne peut l'intercepter. Utilisez en complément de ce projet un outil dédié : un navigateur résistant au fingerprinting (Mullvad Browser, Tor Browser) ou une extension comme CanvasBlocker.

---

## 🤝 Contribuer

Ce projet ne suit que les changements de l'infrastructure d'Utiq. Si vous détectez l'un des éléments suivants, n'hésitez pas à ouvrir une issue ou une pull request :

- Un nouveau domaine racine (alternative à `utiq.com`, `utiq-aws.net`, `utiqcontent.com` ou `mno.link`)
- Un nouveau chemin d'API (alternative à `/op/idconnect/` ou `/op/privacy/`)
- Un nouveau nom de script (alternative à `utiqLoader.js` ou `utiqConsentManager.js`)
- Un partenaire utilisant un préfixe de sous-domaine autre que `utiq.`

Merci d'inclure le domaine ou le chemin observé, le site partenaire où vous l'avez détecté, et idéalement une capture d'écran du Network Logger montrant la requête bloquée (ou non bloquée).

> Les demandes visant à « corriger » le CNAME cloaking ou le fingerprinting en ajoutant des règles de filtre ne peuvent pas être traitées ici : ce sont des limites au niveau du navigateur ou de l'outillage, pas des règles manquantes. Voir [Limites connues](#️-limites-connues--mesures-complémentaires).

---

## 📅 Changelog

> Les numéros de version suivent uniquement les changements des fichiers de filtres. Les mises à jour du seul README ne sont pas versionnées.

| Version | Date | Notes |
|---|---|---|
| 1.6 | 2026-10-25 | Ajout de `\|\|mno.link^` (domaine du handshake MNO), `/utiqConsentManager.js`, `/op/privacy/` et `##iframe[name="__utiqLocator"]` ; élargissement de `/op/idconnect/` à tous les points d'accès et aux requêtes `$subdocument` (iframe) |
| 1.5 | 2026-06-19 | Ajout de `\|\|utiqcontent.com^` aux deux fichiers de filtres (domaine des ressources statiques d'Utiq) |
| 1.4 | 2026-06-19 | Suppression de `$important` de `filters-abp.txt` (non pris en charge par les moteurs AdBlock/ABP) |
| 1.3 | 2026-06-18 | Ajout de `filters-abp.txt` (support AdBlock & AdBlock Plus) |
| 1.2 | 2026-06-18 | Jeu de filtres unifié (uBO + AdGuard) ; correction de la correspondance de préfixe de sous-domaine `\|\|utiq.*^` |
| 1.1 | 2026-06-18 | Séparation des variantes uBO / AdGuard ; ajout du tableau de compatibilité ; correction terminologique |
| 1.0 | — | Version initiale |

---

## 📄 Licence

Ce projet est open-source et sous licence **MIT**. Voir le fichier [LICENSE](LICENSE) pour plus de détails.
