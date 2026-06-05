# hyva-upgrade-analyzer

A DDEV add-on for Magento 2 that detects which Hyvä upstream files changed after `composer update` and maps those changes directly to your custom theme overrides in `app/design/frontend/`.

## Requirements

- DDEV
- Magento 2 project with Hyvä installed (`hyva-themes/*` packages)

## Installation

```bash
ddev add-on get Luuksie/hyva-upgrade-analyzer
```

## Usage

### Step 1 — Before upgrading Hyvä

```bash
ddev hyva:snapshot
```

Fingerprints every file in every `hyva-themes/*` vendor package and saves `.hyva-snapshot.json`. Run this **before** `composer update`.

### Step 2 — Upgrade Hyvä

```bash
ddev composer update 'hyva-themes/*'
```

### Step 3 — Check the impact

```bash
ddev hyva:check
```

Produces `hyva-upgrade-report.md` in your project root.

---

## Reading the Report

### Override Impact Map
The most important section. Every file in `app/design/frontend/` that overrides a Hyvä file that changed upstream:

| Impact | Meaning |
|--------|---------|
| `critical mismatch` | Checkout/cart/payment path changed with Alpine.js changes — fix before deploying |
| `likely broken` | Alpine.js reactive behavior changed in the upstream file |
| `partially outdated` | Markup changed but no Alpine logic impact |
| `new upstream file` | New file added upstream you may want to override |
| `safe` | Upstream file unchanged |

### Risk Levels

| Risk | Triggered by |
|------|-------------|
| `critical` | File in `checkout/`, `cart/`, or `payment/` path AND Alpine.js changed |
| `high` | Any Alpine.js pattern changed (`x-data`, `x-init`, `$store`, `Alpine.store`, etc.) |
| `medium` | Markup or layout changed, no Alpine logic |
| `low` | CSS/LESS only, or comment-only change |

### All Report Sections

1. **Executive Summary** — file counts and risk breakdown at a glance
2. **Package Version Changes** — before/after versions per Hyvä package
3. **Consolidated Change List** — all changed files sorted by risk
4. **Override Impact Map** — your overrides mapped to upstream changes
5. **Critical Issues** — must-fix list with exact file paths and action items
6. **Review Recommended** — medium-risk items worth checking
7. **Safe to Ignore** — count of CSS/comment-only changes
8. **Cross-Module Interaction Risks** — Alpine store changes in one package consumed by another

---

## Options

```bash
# Keep a named snapshot before each upgrade
ddev hyva:snapshot --output=.hyva-snapshot-1.3.7.json

# Analyze against a named snapshot
ddev hyva:check --snapshot=.hyva-snapshot-1.3.7.json --output=report-1.3.8.md

# Include custom Hyvä-adjacent packages
ddev hyva:snapshot --patterns=hyva-themes/*,my-vendor/hyva-*
```

## Exit Codes

| Code | Meaning |
|------|---------|
| `0` | No critical issues |
| `1` | Critical issues found — useful for blocking CI deploys |

---

## How Override Detection Works

All Hyvä packages follow the Magento theme directory structure, so the mapping is always a direct 1:1 path:

```
vendor/hyva-themes/magento2-default-theme/Magento_Catalog/templates/product/view.phtml
                                           ↕
app/design/frontend/<Vendor>/<Theme>/Magento_Catalog/templates/product/view.phtml
```

The index is built during `hyva:snapshot` and looked up in O(1) during `hyva:check` — no configuration needed.
