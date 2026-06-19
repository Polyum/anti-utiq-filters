# 🛡️ anti-utiq-filters

> **Custom filtering rules for uBlock Origin, AdGuard, AdBlock, and AdBlock Plus to block the Utiq tracking network.**

This repository provides optimized filter rules designed to neutralize **Utiq** — the network-level tracking service (commonly known as the "Telecom Cookie") operated by major European carriers, including Orange, SFR, Bouygues, Vodafone, and others.

The objective is to block tracking requests whether they originate from Utiq's central infrastructure or via partner-hosted subdomains (`utiq.orange.fr`, `utiq.france.tv`, `utiq.allocine.fr`, etc.).

---

## 📁 Repository Files

| File | Target | Raw link |
|---|---|---|
| `filters-ubo-adguard.txt` | uBlock Origin, AdGuard | [View raw](https://raw.githubusercontent.com/Polyum/anti-utiq-filters/main/filters-ubo-adguard.txt) |
| `filters-abp.txt` | AdBlock, AdBlock Plus | [View raw](https://raw.githubusercontent.com/Polyum/anti-utiq-filters/main/filters-abp.txt) |

---

## 📋 Filter Rules

Choose the appropriate filter list based on your browser extension. Copy and paste the rules into your extension's **"My filters"** / **"Custom Filters"** editor, or subscribe directly to the raw file URL.

### 1️⃣ For uBlock Origin & AdGuard (`filters-ubo-adguard.txt`)

Optimized for modern filter engines supporting advanced wildcard matching and short modifiers.

```text
! =====================================================================
! Utiq Tracker Mitigation — uBlock Origin & AdGuard compatible
! Source: https://github.com/Polyum/anti-utiq-filters
! =====================================================================

! 1. Central Infrastructure
||utiq.com^
||utiq-aws.net^

! 2. Script & API Interception
/utiqLoader.js$script,important
/op/idconnect/mno-precheck$xhr,important

! 3. Partner-Hosted Subdomain Blocking
! Blocks utiq.orange.fr, utiq.france.tv, utiq.allocine.fr, etc.
! The * wildcard matches any parent domain, regardless of TLD.
||utiq.*^
```

### 2️⃣ For AdBlock & AdBlock Plus (`filters-abp.txt`)

Backward-compatible version adjusted to comply with stricter legacy parsing rules. The `$important` modifier is intentionally omitted — it is a uBlock Origin extension not supported by ABP engines and would cause these rules to be silently ignored or rejected.

```text
! =====================================================================
! Utiq Tracker Mitigation — AdBlock & AdBlock Plus compatible
! Source: https://github.com/Polyum/anti-utiq-filters
! =====================================================================

! 1. Central Infrastructure
||utiq.com^
||utiq-aws.net^

! 2. Script & API Interception
/utiqLoader.js$script
/op/idconnect/mno-precheck$xmlhttprequest

! 3. Partner-Hosted Subdomain Blocking
! Adapted for ABP: matches 'utiq.' at any domain or subdomain boundary
||utiq.
```

---

## 🔍 Technical Analysis

### 1. Central Infrastructure

**`||utiq.com^`**
The `||` anchor targets the root domain **and all its subdomains recursively**. This single rule drops all traffic to nodes such as `consenthub.utiq.com`, `api.utiq.com`, or `cdn.utiq.com`. Applies to all four engines.

**`||utiq-aws.net^`**
Utiq's production backend runs on AWS under this dedicated domain. Blocking the root severs access to the current gateway (`frontend.prod`) and pre-emptively covers any future subdomains the network may deploy. Note that `utiq-aws.net` begins with `utiq-` (hyphen, not dot) and is therefore **not** caught by the partner-subdomain rules — this explicit entry is necessary. Applies to all four engines.

---

### 2. Script & API Interception

**`/utiqLoader.js$script,important`** *(uBlock Origin & AdGuard)*
**`/utiqLoader.js$script`** *(AdBlock & AdBlock Plus)*
Targets Utiq's initialization script by path, regardless of the host domain attempting to inject it. The `$important` modifier is available in uBO and AdGuard only: it assigns strict priority, preventing site-specific allowlist rules from bypassing the block. It is intentionally absent from the ABP variant, where it is not supported and would silently invalidate the rule.

**`/op/idconnect/mno-precheck$xhr,important`** *(uBlock Origin & AdGuard)*
**`/op/idconnect/mno-precheck$xmlhttprequest`** *(AdBlock & AdBlock Plus)*
The `mno-precheck` (*Mobile Network Operator*) endpoint is the API call that queries the cellular network to validate your subscriber identity in the background. Both rules target XHR/Fetch requests to this exact path. `$xhr` is the modern short form supported by uBO and AdGuard; `$xmlhttprequest` is the legacy equivalent required by AdBlock and AdBlock Plus. The `$important` modifier is again omitted from the ABP variant for the same reason.

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

> This is **not** first-party cloaking (which hides a tracker behind a neutral CNAME). Here, the Utiq brand is explicitly visible in the subdomain name — the blocking strategy is therefore straightforward: match the `utiq.` prefix at the start of any hostname.

**`||utiq.*^`** *(uBlock Origin & AdGuard)*
The `||` anchor matches at the beginning of a hostname or after a `.` separator. The `*` wildcard absorbs the entire parent domain:

- `utiq.orange.fr` → `||` anchors at position 0, `utiq.` matches, `*` absorbs `orange.fr` ✅
- `utiq.france.tv` → same mechanism ✅
- `cdn.utiq.orange.fr` → `||` anchors after the first `.`, `utiq.orange.fr` is matched ✅

**`||utiq.`** *(AdBlock & AdBlock Plus)*
ABP does not reliably handle the `.*^` wildcard combination in this position. The trailing `^` and wildcard are dropped: `||utiq.` matches any request where `utiq.` appears at a domain boundary, covering all partner-hosted subdomains with equivalent effectiveness.

---

## ✅ Compatibility

| Rule | uBlock Origin | AdGuard | AdBlock | AdBlock Plus |
|---|---|---|---|---|
| `\|\|domain^` (root + subdomains) | ✅ | ✅ | ✅ | ✅ |
| `$script` modifier | ✅ | ✅ | ✅ | ✅ |
| `$xhr` modifier | ✅ | ✅ | ❌ | ❌ |
| `$xmlhttprequest` modifier | ✅ | ✅ | ✅ | ✅ |
| `$important` modifier | ✅ | ✅ | ❌ | ❌ |
| `\|\|utiq.*^` (wildcard) | ✅ | ✅ | ❌ | ❌ |
| `\|\|utiq.` (boundary match) | ✅ | ✅ | ✅ | ✅ |

> These rules apply to **browser-based** filtering only (extension required). They have no effect on native mobile apps or system-level traffic.

---

## 🛠️ Verification

To confirm the filters are active:

1. Navigate to a partner website using the Utiq network.
2. Open your extension's **Network Logger** (uBlock Origin dashboard → Logger tab; AdGuard → Filtering Log).
3. Search for `utiq`. All related requests (`utiqLoader.js`, `mno-precheck`, partner subdomains, etc.) must appear in **red** (Blocked), confirming the handshake was intercepted before any carrier data was transmitted.

---

## 🛡️ Defense in Depth

For maximum protection, combine these filters with the following layers:

### Layer 1 — Legal Opt-Out (Carrier Level)
Formally revoke your consent at the carrier infrastructure level via Utiq's official privacy portal:
👉 **[consenthub.utiq.com](https://consenthub.utiq.com/)**

### Layer 2 — Community Blocklist (Automated Maintenance)
Subscribe to a community-maintained blocklist to receive automatic domain updates without manual intervention:
- 📋 **Subscription URL:** [utiq-tracker.online/blocklists/utiq-standard-adblock.txt](https://utiq-tracker.online/blocklists/utiq-standard-adblock.txt)
- 🌐 **Project Reference:** [utiq-tracker.online](https://utiq-tracker.online/)

> ⚠️ Always verify the trustworthiness of a third-party blocklist before subscribing.

---

## 📅 Changelog

| Version | Date | Notes |
|---|---|---|
| 1.4 | 2025-06-18 | Removed `$important` from `filters-abp.txt` (not supported by AdBlock/ABP engines); updated README and compatibility table accordingly |
| 1.3 | 2025-06-18 | Added AdBlock & AdBlock Plus support (`filters-abp.txt`); split into two dedicated filter files; documented `$xmlhttprequest` vs `$xhr` and `\|\|utiq.` vs `\|\|utiq.*^` differences |
| 1.2 | 2025-06-18 | Unified filter set (uBO + AdGuard); corrected `\|\|utiq.*^` subdomain-prefix matching documentation |
| 1.1 | 2025-06-18 | Split uBO / AdGuard variants; added compatibility table; corrected terminology |
| 1.0 | — | Initial release |

---

## 📄 License

This project is open-source and licensed under the **MIT License**. See the [LICENSE](LICENSE) file for more details.
