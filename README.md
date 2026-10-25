# 🛡️ anti-utiq-filters

> **Custom filtering rules for uBlock Origin, AdGuard, AdBlock, and AdBlock Plus to block the Utiq tracking network.**

This repository provides optimized filter rules designed to neutralize **Utiq** — the network-level tracking service (commonly known as the "Telecom Cookie") operated by major European carriers, including Orange, SFR, Bouygues, Vodafone, and others.

The objective is to block tracking requests whether they originate from Utiq's central infrastructure or via partner-hosted subdomains (`utiq.orange.fr`, `utiq.france.tv`, `utiq.allocine.fr`, etc.).

> 🇫🇷 *Une [version française de ce README](README.fr.md) est aussi disponible.*

---

## 📖 Table of Contents

- [Repository Files](#-repository-files)
- [Installation](#-installation)
  - [For uBlock Origin & AdGuard](#1️⃣-for-ublock-origin--adguard-filters-ubo-adguardtxt)
  - [For AdBlock & AdBlock Plus](#2️⃣-for-adblock--adblock-plus-filters-abptxt)
- [Technical Analysis](#-technical-analysis)
  - [Central Infrastructure](#1-central-infrastructure)
  - [Script & API Interception](#2-script--api-interception)
  - [Partner-Hosted Subdomain Blocking](#3-partner-hosted-subdomain-blocking)
  - [Cross-Domain Iframe Blocking](#4-cross-domain-iframe-blocking)
- [Known Utiq Infrastructure](#️-known-utiq-infrastructure)
- [Compatibility](#-compatibility)
- [Verification](#️-verification)
- [Defense in Depth](#️-defense-in-depth)
- [Known Limitations & Complementary Measures](#️-known-limitations--complementary-measures)
- [Contributing](#-contributing)
- [Changelog](#-changelog)
- [License](#-license)

---

## 📁 Repository Files

| File | Target | Raw link |
|---|---|---|
| `filters-ubo-adguard.txt` | uBlock Origin, AdGuard | [View raw](https://raw.githubusercontent.com/Polyum/anti-utiq-filters/main/filters-ubo-adguard.txt) |
| `filters-abp.txt` | AdBlock, AdBlock Plus | [View raw](https://raw.githubusercontent.com/Polyum/anti-utiq-filters/main/filters-abp.txt) |

---

## 📥 Installation

Choose the appropriate filter list based on your browser extension.

### 1️⃣ For uBlock Origin & AdGuard (`filters-ubo-adguard.txt`)

Optimized for modern filter engines supporting advanced wildcard matching and short modifiers.

#### ✅ Recommended — Subscribe to the raw file (auto-updates every 2 days)

The filter files include an `Expires: 2 days` metadata directive. Your extension will automatically fetch the latest version every 2 days, without any manual intervention.

**uBlock Origin:** Settings → Filter lists → Import → Paste the URL below → Apply changes

**AdGuard:** Settings → Filters → Custom → Add custom filter → Paste the URL below

~~~
https://raw.githubusercontent.com/Polyum/anti-utiq-filters/main/filters-ubo-adguard.txt
~~~

#### 🔁 Alternative — Manual copy/paste (no auto-update)

Copy and paste the rules below into your extension's **"My filters"** / **"Custom Filters"** editor. Note that this method does not auto-update — you will need to check for new versions manually.

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

### 2️⃣ For AdBlock & AdBlock Plus (`filters-abp.txt`)

Backward-compatible version adjusted to comply with stricter legacy parsing rules. The `$important` modifier is intentionally omitted — it is a uBlock Origin extension not supported by ABP engines and would cause these rules to be silently ignored or rejected.

#### ✅ Recommended — Subscribe to the raw file (auto-updates every 2 days)

**AdBlock / AdBlock Plus:** Options → Filter lists → Add a filter list → Paste the URL below

~~~
https://raw.githubusercontent.com/Polyum/anti-utiq-filters/main/filters-abp.txt
~~~

#### 🔁 Alternative — Manual copy/paste (no auto-update)

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

## 🔍 Technical Analysis

### 1. Central Infrastructure

**`||utiq.com^`**
The `||` anchor targets the root domain **and all its subdomains recursively**. This single rule drops all traffic to nodes such as `consenthub.utiq.com`, `service.utiq.com`, `api.utiq.com`, or `cdn.utiq.com`. Applies to all four engines.

**`||utiq-aws.net^`**
Utiq's production backend runs on AWS under this dedicated domain. Blocking the root severs access to the current gateway (`frontend.prod`) and pre-emptively covers any future subdomains the network may deploy. Note that `utiq-aws.net` begins with `utiq-` (hyphen, not dot) and is therefore **not** caught by the partner-subdomain rules — this explicit entry is necessary. Applies to all four engines.

**`||utiqcontent.com^`**
A separate root domain used by the Utiq SDK to load static assets (configuration files and SVGs). Independent from `utiq.com` and `utiq-aws.net`, it would not be caught by any other rule. Applies to all four engines.

**`||mno.link^`**
During the Mobile Network Operator (MNO) handshake, Utiq routes subscriber identification requests through carrier-specific subdomains under `mno.link` (`orangefr.mno.link`, `o2de.mno.link`, `vfde-fixed.mno.link`, etc.), which point via CNAME to `frontend.prod.utiq-aws.net`. Because these domains do not start with `utiq.`, they are not matched by partner-subdomain rules, and browsers without native CNAME uncloaking (such as Chromium-based browsers) cannot see the underlying `utiq-aws.net` target. Blocking `mno.link` directly closes this blind spot. Applies to all four engines.

---

### 2. Script & API Interception

**`/utiqLoader.js$script,important`** *(uBlock Origin & AdGuard)*
**`/utiqLoader.js$script`** *(AdBlock & AdBlock Plus)*
Targets Utiq's initialization script by path, regardless of the host domain attempting to inject it. The `$important` modifier is available in uBO and AdGuard only: it assigns strict priority, preventing site-specific allowlist rules from bypassing the block. It is intentionally absent from the ABP variant, where it is not supported and would silently invalidate the rule.

**`/utiqConsentManager.js$script,important`** *(uBlock Origin & AdGuard)*
**`/utiqConsentManager.js$script`** *(AdBlock & AdBlock Plus)*
Targets Utiq's consent management script by path. While normally loaded by `utiqLoader.js`, the Utiq SDK allows publishers to load the consent manager from a separate custom origin via the `consentManagerOrigin` configuration option. Blocking it explicitly ensures coverage in all setups.

**`/op/idconnect/$xhr,subdocument,important`** *(uBlock Origin & AdGuard)*
**`/op/idconnect/$xmlhttprequest,subdocument`** *(AdBlock & AdBlock Plus)*
Intercepts the entire `/op/idconnect/` endpoint family (`/op/idconnect/mno-precheck`, `/op/idconnect/data`, `/op/idconnect/mno-selector`, `/op/idconnect/tmi-*`) used to query the cellular or fixed network and retrieve subscriber tokens in the background. In addition to XHR/Fetch calls (`$xhr` in uBO/AdGuard; `$xmlhttprequest` in AdBlock/ABP), Utiq loads several `/op/idconnect/` endpoints inside hidden iframes, which are caught by the `$subdocument` modifier. Path-level interception also neutralizes custom CNAME hosts configured via `customUtiqHost`.

**`/op/privacy/$xhr,important`** *(uBlock Origin & AdGuard)*
**`/op/privacy/$xmlhttprequest`** *(AdBlock & AdBlock Plus)*
Blocks Utiq's privacy and consent synchronization endpoints (`/op/privacy/info` and `/op/privacy/permission`) across all partner domains.

---

### 3. Partner-Hosted Subdomain Blocking

To extend their reach, Utiq's carrier and media partners provision a `utiq.` subdomain directly on their own domains:

| Partner subdomain | Parent domain |
|---|---|
| `utiq.orange.fr` | `orange.fr` |
| `utiq.france.tv` | `france.tv` |
| `utiq.allocine.fr` | `allocine.fr` |
| `utiq.lefigaro.fr` | `lefigaro.fr` |
| … | … |

These hostnames do not appear in generic `utiq.com` blocklists because they are served under entirely different registrable domains.

**`||utiq.*^`** *(uBlock Origin & AdGuard)*
The `||` anchor matches at the beginning of a hostname or after a `.` separator. The `*` wildcard absorbs the entire parent domain:

- `utiq.orange.fr` → `||` anchors at position 0, `utiq.` matches, `*` absorbs `orange.fr` ✅
- `utiq.france.tv` → same mechanism ✅
- `cdn.utiq.orange.fr` → `||` anchors after the first `.`, `utiq.orange.fr` is matched ✅

**`||utiq.`** *(AdBlock & AdBlock Plus)*
ABP does not reliably handle the `.*^` wildcard combination in this position. The trailing `^` and wildcard are dropped: `||utiq.` matches any request where `utiq.` appears at a domain boundary, covering all partner-hosted subdomains with equivalent effectiveness.

---

### 4. Cross-Domain Iframe Blocking

**`##iframe[name="__utiqLocator"]`**
When publishers run programmatic advertising (Prebid.js v10.11+) inside cross-domain iframes, the Utiq SDK injects a void iframe named `__utiqLocator` into the parent DOM to signal the presence of Utiq IDs and cascade `adtechpass` (`atid`) and `martechpass` (`mtid`) tokens to child frames.

This is a **cosmetic rule: it hides the element, it does not prevent it from being created or from doing its job.** The iframe is injected by JavaScript, so the real protection comes from stopping the SDK upstream (`/utiqLoader.js`, `||utiq.com^`, etc. — see sections 1 and 2): if the loader never runs, the locator is never injected. Treat this rule as a cosmetic safety net, not as protection in itself. Supported by all four engines.

---

## 🗺️ Known Utiq Infrastructure

This section documents Utiq's known core domains and endpoints, as a reference for contributors monitoring new infrastructure deployments.

> For a comprehensive and maintained list of partner-hosted subdomains, refer to the community blocklist: [utiq-tracker.online](https://utiq-tracker.online/)

### Root Domains
| Domain | Role |
|---|---|
| `utiq.com` | Main infrastructure — consent hub, APIs, CDN |
| `utiq-aws.net` | AWS production backend |
| `utiqcontent.com` | Static assets — SDK configuration files and SVGs |
| `mno.link` | Carrier-specific MNO handshake gateways (`orangefr.mno.link`, `o2de.mno.link`, etc.) |

### Known Subdomains
| Subdomain | Role |
|---|---|
| `consenthub.utiq.com` | User consent management portal |
| `service.utiq.com` | Consent hub backend service |
| `frontend.prod.utiq-aws.net` | Production API gateway — CNAME target for all partner-hosted subdomains and `*.mno.link` |

### Known API Endpoints
| Path | Role |
|---|---|
| `/op/idconnect/mno-precheck` | Mobile Network Operator handshake — subscriber eligibility validation |
| `/op/idconnect/data` | Subscriber token & `connectId` retrieval (XHR and iframe) |
| `/op/idconnect/mno-selector` | Telecom operator selection endpoint |
| `/op/idconnect/tmi-*` | Carrier-specific identification endpoints (e.g., `/op/idconnect/tmi-o2de`) |
| `/op/privacy/info` | Privacy metadata endpoint |
| `/op/privacy/permission` | Consent permission synchronization endpoint |

### Known Scripts
| Filename | Role |
|---|---|
| `utiqLoader.js` | Main initialization script injected on partner sites |
| `utiqConsentManager.js` | Consent popup script, loaded by `utiqLoader.js` or via a custom `consentManagerOrigin` |

### Known First-Party Cookies & Local Storage Keys
| Type | Name | Role |
|---|---|---|
| Cookie | `connectId` | Caches the carrier network signal (90 days) to avoid querying the ISP on every visit |
| Cookie | `utiq_consent_status` | Stores base64-encoded user consent state (`true` / `false`, up to 180 days) |
| Cookie | `utiq_consent_version` | Records the consent version accepted by the user (390 days) |
| Cookie | `utiq_logging_level` | Stores SDK debug logging level |
| `localStorage` | `utiqEligibility` | Stores ISP eligibility and ConsentHub revocation status |
| `localStorage` | `utiqDataCallTimestamp` | Timestamp of the last MNO eligibility check |
| `localStorage` | `utiqPass` | Stores generated `martechpass` (`mtid`), `adtechpass` (`atid`), and `attributionpass` (`attrid`) tokens |
| `localStorage` | `utiq_openrtb_eids` | Serialized OpenRTB JSON array of Utiq IDs read directly by Prebid.js modules (`utiqIdSystem` / `utiqMtpIdSystem`) |

---

## ✅ Compatibility

| Rule | uBlock Origin | AdGuard | AdBlock | AdBlock Plus |
|---|---|---|---|---|
| `\|\|domain^` (root + subdomains) | ✅ | ✅ | ✅ | ✅ |
| `$script` modifier | ✅ | ✅ | ✅ | ✅ |
| `$xhr` modifier | ✅ | ✅ | ❌ | ❌ |
| `$xmlhttprequest` modifier | ✅ | ✅ | ✅ | ✅ |
| `$subdocument` modifier | ✅ | ✅ | ✅ | ✅ |
| `$important` modifier | ✅ | ✅ | ❌ | ❌ |
| `\|\|utiq.*^` (wildcard) | ✅ | ✅ | ❌ | ❌ |
| `\|\|utiq.` (boundary match) | ✅ | ✅ | ✅ | ✅ |
| `##iframe[name="..."]` (cosmetic — hides only) | ✅ | ✅ | ✅ | ✅ |
| CNAME uncloaking (browser feature, **not** a filter rule) | ✅ Firefox only | — | — | — |

> `—` = no equivalent verified for this project.
>
> These rules apply to **browser-based** filtering only (extension required). They have no effect on native mobile apps or system-level traffic.

---

## 🛠️ Verification

To confirm the filters are active:

1. Navigate to a partner website using the Utiq network.
2. Open your extension's **Network Logger** (uBlock Origin dashboard → Logger tab; AdGuard → Filtering Log).
3. Search for `utiq` or `idconnect`. All related requests (`utiqLoader.js`, `utiqConsentManager.js`, `/op/idconnect/*`, `*.mno.link`, partner subdomains, etc.) must appear in **red** (Blocked), confirming the handshake was intercepted before any carrier data was transmitted.

---

## 🛡️ Defense in Depth

No single layer provides complete protection. Each layer's blind spots are covered by the others: stack as many as your setup allows.

| Layer | What it protects | Blind spots |
|---|---|---|
| Filters (this project) | HTTP requests, scripts, API calls, iframes | Browser only; ineffective in native apps |
| Legal opt-out | Carrier-level consent revocation | Can be accidentally overridden by clicking "Accept" on a partner site |
| VPN | Network level — prevents token generation entirely | Only active when the VPN tunnel is up |
| DNS blocklist | All devices on the network | Blind to CNAME cloaking; cannot block by path or request type |

### Layer 1 — Browser Filters (this project)
These rules intercept Utiq's script, API handshake, and partner subdomains at the HTTP level. Path-based rules (`/utiqLoader.js`, `/utiqConsentManager.js`, `/op/idconnect/`, `/op/privacy/`) remain effective even against CNAME cloaking, unlike DNS-level blocking — unless a partner also renames those paths. See [Known Limitations](#️-known-limitations--complementary-measures).

### Layer 2 — Legal Opt-Out (Carrier Level)
Formally revoke your consent at the carrier infrastructure level via Utiq's official privacy portal:
👉 **[consenthub.utiq.com](https://consenthub.utiq.com/)**

### Layer 3 — VPN (Network Level)
A VPN is the most fundamental protection against Utiq: the token is generated by your carrier via a network-level injection tied to your IP address and subscriber identity. Routing your traffic through a VPN server makes this identification impossible — the VPN provider is not a carrier partner, and your ISP only sees an encrypted tunnel and cannot associate your session with your network identifier. This protection is independent of browser configuration and covers all devices and apps on the connection.

> ⚠️ Protection is only active while the VPN tunnel is up. If your device falls back to a direct connection (mobile data, or Wi-Fi without VPN), Utiq may resume operating normally — which is precisely why stacking multiple layers matters.

### Layer 4 — Community DNS Blocklist (Automated Maintenance)
Subscribe to a community-maintained blocklist to receive automatic domain updates without manual intervention:
- 📋 **Subscription URL:** [utiq-tracker.online/blocklists/utiq-standard-adblock.txt](https://utiq-tracker.online/blocklists/utiq-standard-adblock.txt)
- 🌐 **Project Reference:** [utiq-tracker.online](https://utiq-tracker.online/)

> ⚠️ Always verify the trustworthiness of a third-party blocklist before subscribing.

> 💡 **Pi-hole / AdGuard Home users:** this project's filters rely on HTTP-level rules (script paths, API endpoints, request types) that DNS-based blockers cannot process. The community blocklist above may offer better-suited DNS entries for your setup.

---

## ⚠️ Known Limitations & Complementary Measures

These filters stop Utiq as long as its traffic is identifiable either by hostname (`utiq.*`, `utiq.com`, `utiq-aws.net`, `utiqcontent.com`, `mno.link`) or by script/endpoint path. Two situations sit outside what a static filter list can fully solve.

### 1. CNAME cloaking behind an arbitrary first-party hostname

A partner can expose Utiq under a hostname of its own choosing (for example `stats.example.com`) that is DNS-CNAME'd to `frontend.prod.utiq-aws.net`. The filter engine only sees `stats.example.com`, so **domain rules cannot match** in this scenario, and no additional domain rule can fix it, because the hostname is not Utiq's.

What still protects you:

- **Path rules** (`/utiqLoader.js`, `/utiqConsentManager.js`, `/op/idconnect/`, `/op/privacy/`) do not depend on the hostname and keep working, unless a partner also renames those paths.
- **uBlock Origin on Firefox** can resolve the CNAME itself and re-apply the filters to the real hostname, where `||utiq-aws.net^` matches. In uBO this is the **"Uncloak canonical names"** option (Settings → Privacy), which is **enabled by default**. If you disabled it, re-enable it. It is Firefox-only: it relies on a DNS API that Firefox exposes to extensions and Chromium-based browsers do not.

> The remaining gap is therefore narrow but real: a **non-Firefox browser** facing a partner that uses an arbitrary first-party hostname **and** renamed paths. No filter list can close it; the VPN layer above does.

### 2. Data already stored, and complementary fingerprinting

- **Existing local data is not erased by filters.** Filters stop the SDK and its API calls, but values stored before the filters were installed (the `connectId` cookie, `utiqPass` and other keys listed in [Known First-Party Cookies & Local Storage Keys](#known-first-party-cookies--local-storage-keys)) stay in your browser until you clear them. Clear the site data of Utiq-using sites once after installing the filters.
- **Fingerprinting.** Independent reports indicate that sites using Utiq often combine it with canvas and font fingerprinting. That runs client-side inside otherwise-legitimate scripts, so no network filter, domain- or path-based, can intercept it. Use a dedicated tool in addition to this project: a fingerprint-resistant browser (Mullvad Browser, Tor Browser) or an extension such as CanvasBlocker.

---

## 🤝 Contributing

This project only tracks changes to Utiq's infrastructure. If you detect any of the following, feel free to open an issue or a pull request:

- A new root domain (alternative to `utiq.com`, `utiq-aws.net`, `utiqcontent.com`, or `mno.link`)
- A new API path (alternative to `/op/idconnect/` or `/op/privacy/`)
- A new script name (alternative to `utiqLoader.js` or `utiqConsentManager.js`)
- A partner using a subdomain prefix other than `utiq.`

Please include the domain or path observed, the partner site where you detected it, and ideally a screenshot of the Network Logger showing the blocked (or unblocked) request.

> Requests to "fix" CNAME cloaking or fingerprinting by adding filter rules cannot be actioned here: these are browser-level or tooling-level limits, not missing rules. See [Known Limitations](#️-known-limitations--complementary-measures).

---

## 📅 Changelog

> Version numbers track changes to filter files only. README-only updates are not versioned.

| Version | Date | Notes |
|---|---|---|
| 1.6 | 2026-10-25 | Added `\|\|mno.link^` (MNO handshake domain), `/utiqConsentManager.js`, `/op/privacy/`, and `##iframe[name="__utiqLocator"]`; broadened `/op/idconnect/` to cover all endpoints and `$subdocument` (iframe) requests |
| 1.5 | 2026-06-19 | Added `\|\|utiqcontent.com^` to both filter files (Utiq static assets domain) |
| 1.4 | 2026-06-19 | Removed `$important` from `filters-abp.txt` (not supported by AdBlock/ABP engines) |
| 1.3 | 2026-06-18 | Added `filters-abp.txt` (AdBlock & AdBlock Plus support) |
| 1.2 | 2026-06-18 | Unified filter set (uBO + AdGuard); corrected `\|\|utiq.*^` subdomain-prefix matching |
| 1.1 | 2026-06-18 | Split uBO / AdGuard variants; added compatibility table; corrected terminology |
| 1.0 | — | Initial release |

---

## 📄 License

This project is open-source and licensed under the **MIT License**. See the [LICENSE](LICENSE) file for more details.
