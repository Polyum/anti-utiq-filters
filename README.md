# 🛡️ anti-utiq-filters

> **Custom filtering rules for uBlock Origin and AdGuard to block the Utiq tracking network.**

This repository provides optimized filter rules designed to neutralize **Utiq** — the network-level tracking service (commonly known as the "Telecom Cookie") operated by major European carriers, including Orange, SFR, Bouygues, Vodafone, and others.

The objective is to block tracking requests whether they originate from Utiq's central infrastructure or via partner-hosted subdomains (`utiq.orange.fr`, `utiq.france.tv`, etc.).

---

## 📋 Filter Rules

The following rules are compatible with both **uBlock Origin** and **AdGuard**. Copy and paste them into the **"My filters"** / **"Custom Filters"** editor of your application.

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
! Blocks utiq.orange.fr, utiq.france.tv, utiq.sfr.fr, etc.
! The * wildcard matches any parent domain, regardless of TLD.
||utiq.*^
```

---

## 🔍 Technical Analysis

### 1. Central Infrastructure

**`||utiq.com^`**
The `||` anchor targets the root domain **and all its subdomains recursively**. This single rule drops all traffic to nodes such as `consenthub.utiq.com`, `api.utiq.com`, or `cdn.utiq.com`.

**`||utiq-aws.net^`**
Utiq's production backend runs on AWS under this dedicated domain. Blocking the root severs access to the current gateway (`frontend.prod`) and pre-emptively covers any future subdomains the network may deploy.

---

### 2. Script & API Interception

**`/utiqLoader.js$script,important`**
Targets Utiq's initialization script by path, regardless of the host domain or TLD attempting to inject it. The `$important` modifier assigns strict priority, preventing site-specific allowlist rules from bypassing the block.

**`/op/idconnect/mno-precheck$xhr,important`**
The `mno-precheck` (*Mobile Network Operator*) endpoint is the API call that queries the cellular network to validate your subscriber identity in the background. Targeting this path with the `$xhr` type modifier ensures the network handshake fails immediately, halting advertising token generation.

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

**`||utiq.*^`**
The `||` anchor in both uBO and AdGuard matches at the beginning of a hostname **or after a `.` separator**. This means:

- For `utiq.orange.fr` → `||` anchors at position 0, `utiq.` matches, `*` matches `orange.fr` ✅
- For `utiq.france.tv` → `||` anchors at position 0, `utiq.` matches, `*` matches `france.tv` ✅
- For `cdn.utiq.orange.fr` → `||` anchors after the first `.`, `utiq.orange.fr` is matched ✅

This is **not** a TLD wildcard: the `*` matches the entire parent domain (including its TLD), not just the TLD itself. The behavior is identical in uBO and AdGuard.

> **Note:** This rule also incidentally catches `utiq.com` and `utiq-aws.net` subdomains, making rules 1 and 3 partially overlapping. Rules 1 and 2 are kept explicitly for clarity and as a strict safety net.

---

## ✅ Compatibility

| Rule / Feature | uBlock Origin | AdGuard |
|---|---|---|
| `\|\|domain^` (root + subdomains) | ✅ | ✅ |
| `$script`, `$xhr` type modifiers | ✅ | ✅ |
| `$important` priority override | ✅ | ✅ |
| `\|\|utiq.*^` (subdomain prefix wildcard) | ✅ | ✅ |

> These rules apply to **browser-based** filtering only (extension required). They have no effect on native mobile apps or system-level traffic.

---

## 🛠️ Verification

To confirm the filters are active:

1. Navigate to a partner website using the Utiq network.
2. Open the uBlock Origin or AdGuard **Network Logger**.
3. Search for `utiq`. All related requests (`utiqLoader.js`, `mno-precheck`, partner subdomains, etc.) must appear in **red** (Blocked), confirming the handshake was intercepted before any carrier data was transmitted.

---

## 🛡️ Defense in Depth

For maximum protection, combine these filters with the following layers:

### Layer 1 — Legal Opt-Out (Carrier Level)
Formally revoke your consent at the carrier infrastructure level via Utiq's official privacy portal:
👉 **[consenthub.utiq.com](https://consenthub.utiq.com/)**

### Layer 2 — Community Blocklist (Automated Maintenance)
Subscribe to a community-maintained blocklist to receive automatic domain updates:
- 📋 **Subscription URL:** [utiq-tracker.online/blocklists/utiq-standard-adblock.txt](https://utiq-tracker.online/blocklists/utiq-standard-adblock.txt)
- 🌐 **Project Reference:** [utiq-tracker.online](https://utiq-tracker.online/)

> ⚠️ Always verify the trustworthiness of a third-party blocklist before subscribing.

---

## 📄 License

This project is open-source and licensed under the **MIT License**. See the [LICENSE](LICENSE) file for more details.
