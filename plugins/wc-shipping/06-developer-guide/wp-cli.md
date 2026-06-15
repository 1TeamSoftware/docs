---
title: "WP-CLI Commands — WooCommerce Shipping Plugins | 1TeamSoftware"
description: "Complete WP-CLI reference for 1TeamSoftware WooCommerce shipping plugins (Shippo, EasyPost, FedEx, UPS, ShipStation, ShipEngine, Shipmondo, ChitChats, Stallion Express): status, settings, rates, products, boxes, labels, tracking, manifests, and license commands."
---

# WP-CLI Commands

Every 1TeamSoftware shipping plugin comes with a full set of WP-CLI commands. From the command line you can configure the plugin, check your products, and quote live rates; the PRO versions add shipments, labels, tracking, and manifests. This is the fast way to handle setup, deployment scripts, and bulk jobs — and it's what the [AI agent skills](ai-skills.md) use under the hood.

The commands are the same for every carrier. The only thing that changes between plugins is the command namespace.

## Command Namespace

Each plugin registers its commands under a namespace. It's the plugin slug without the `-pro` suffix — the same string as the plugin's [hook prefix](hooks-by-plugin.md), but with hyphens instead of underscores. The free and PRO versions share the same namespace.

| Plugin | CLI Namespace | Product |
|--------|--------------|---------|
| Shippo Shipping | `wc-shippo-shipping` | [Shippo Shipping PRO](https://1teamsoftware.com/product/wc-shippo-shipping-pro/) |
| EasyPost Shipping | `wc-easypost-shipping` | [EasyPost Shipping PRO](https://1teamsoftware.com/product/wc-easypost-shipping-pro/) |
| FedEx Shipping | `wc-fedex-shipping` | [FedEx Shipping PRO](https://1teamsoftware.com/product/wc-fedex-shipping-pro/) |
| UPS Shipping | `wc-ups-shipping` | [UPS Shipping PRO](https://1teamsoftware.com/product/wc-ups-shipping-pro/) |
| ShipStation Shipping | `wc-shipstation-shipping-v2` | [ShipStation Shipping PRO](https://1teamsoftware.com/product/wc-shipstation-shipping-pro/) |
| ShipEngine Shipping | `wc-shipengine-shipping` | [ShipEngine Shipping PRO](https://1teamsoftware.com/product/wc-shipengine-shipping-pro/) |
| Shipmondo Shipping | `wc-shipmondo-shipping` | [Shipmondo Shipping PRO](https://1teamsoftware.com/product/wc-shipmondo-shipping-pro/) |
| ChitChats Shipping | `wc-chitchats-shipping` | [ChitChats Shipping PRO](https://1teamsoftware.com/product/wc-chitchats-shipping-pro/) |
| Stallion Express Shipping | `wc-stallionexpress-shipping` | [Stallion Express Shipping PRO](https://1teamsoftware.com/product/wc-stallionexpress-shipping-pro/) |

The current ShipStation plugin is API v2, so its namespace is `wc-shipstation-shipping-v2`. The legacy v1 plugin uses `wc-shipstation-shipping` — confirm which one you have with `wp help | grep shipping`.

Not sure which one you have? Run `wp help | grep shipping`, or list every registered shipping command with `wp cli cmd-dump --format=json | grep -o 'wc-[a-z-]*shipping'`.

The examples below use `wc-shippo-shipping`. Swap in the namespace for whichever plugin you have installed.

## Quick Start

Four commands take you from "is this thing on?" to a working live rate:

```bash
# 1. See the current state: version, carrier, sandbox flag, origin
wp wc-shippo-shipping status

# 2. Find out what still needs configuring
wp wc-shippo-shipping validate

# 3. Add your API key and origin address, then check again
wp wc-shippo-shipping settings set liveApiToken "your-api-key"
wp wc-shippo-shipping settings set origin '{"name":"My Store","address":"123 Main St","city":"San Francisco","state":"CA","postcode":"94105","country":"US"}'
wp wc-shippo-shipping validate

# 4. Quote a real rate to confirm the carrier responds
wp wc-shippo-shipping rates quote --products="123" --destination="500 5th Ave, New York, NY 10001, US"
```

Prefer to describe what you want instead of running commands yourself? The [AI agent skills](ai-skills.md) do all of this for you.

## Output Formats

Most read commands take a `--format` option. `status`, `validate`, `features`, `services`, `zones`, and `rates get|quote` support all five formats:

```bash
wp wc-shippo-shipping status --format=table   # default, human-readable
wp wc-shippo-shipping status --format=json    # for scripts
wp wc-shippo-shipping status --format=csv
wp wc-shippo-shipping status --format=yaml
wp wc-shippo-shipping status --format=count
```

`settings list` and the `products` commands (`audit`, `stats`, `fit-box`, `orders`) accept only `--format=table` (default) or `--format=json` — the other three values return an error. `settings get` has no `--format`; it prints the raw value for piping.

One thing to watch when parsing JSON: a few commands print a log line or a `Warning:` on stderr before the JSON. Redirect stderr away and read only stdout, like this:

```bash
wp wc-shippo-shipping rates quote --products="123" --destination="..." --format=json 2>/dev/null | jq .
```

---

## System & Diagnostics

Works in both the free and PRO versions.

### Check Plugin Status
Shows the version, carrier, sandbox/debug/cache flags, whether live rates are on, the origin address, how many shipping-method instances are active, and any multi-vendor platform it detected.

```bash
wp wc-shippo-shipping status
wp wc-shippo-shipping status --format=json
```

### Validate Configuration
Runs every configuration check and reports `PASS`, `FAIL`, or `WARN` for each one. It exits with code `1` if anything fails, so you can drop it straight into a CI or post-deploy script.

```bash
wp wc-shippo-shipping validate
wp wc-shippo-shipping validate --format=json
```

### List Carrier Capabilities
Lists what the carrier can do: live rates, label purchase, returns, insurance, signature, manifests, address validation, and so on.

```bash
wp wc-shippo-shipping features
```

### Discover Carrier Services
Lists the carrier's service IDs and whether each is enabled. Use it to find the valid IDs for `settings set services`.

```bash
wp wc-shippo-shipping services
```

### Inspect Shipping Zones
Shows which WooCommerce shipping zones the plugin is active on. Zones where it's inactive return no rates at checkout, so this is the first place to look when rates go missing in one region.

```bash
wp wc-shippo-shipping zones
```

---

## Settings & Configuration

Works in both versions. API tokens and the license key are hidden in output unless you add `--raw`.

```bash
# List all settings (add --instance=<id> for a specific zone instance)
wp wc-shippo-shipping settings list
wp wc-shippo-shipping settings list --instance=3

# Read one setting
wp wc-shippo-shipping settings get origin
wp wc-shippo-shipping settings get sandbox
wp wc-shippo-shipping settings get testApiToken --raw

# Update settings
wp wc-shippo-shipping settings set sandbox yes
wp wc-shippo-shipping settings set enableLiveShippingRates yes
wp wc-shippo-shipping settings set testApiToken "shippo_test_abc123"
wp wc-shippo-shipping settings set liveApiToken "shippo_live_xyz789"

# Origin address (JSON)
wp wc-shippo-shipping settings set origin '{"name":"My Store","address":"123 Main St","city":"San Francisco","state":"CA","postcode":"94105","country":"US","phone":"+1 555 123 4567","email":"ship@example.com"}'

# Boxes (JSON array)
wp wc-shippo-shipping settings set boxes '[{"enabled":"yes","boxName":"Small Box","type":"parcel","length":"8","width":"6","height":"4","weight":"0.5","maxweight":"20"}]'

# Enabled services (JSON object keyed by service ID)
wp wc-shippo-shipping settings set services '{"usps_ground_advantage":{"enabled":true,"name":"USPS Ground Advantage"}}'

# Rate adjustments
wp wc-shippo-shipping settings set priceAdjustment 2.50          # flat surcharge (use a negative number to discount)
wp wc-shippo-shipping settings set priceAdjustmentPercent 1.10   # multiplier, so 1.10 adds 10%

# Force a value to be stored as a literal string (skips the automatic yes/no, number, and JSON inference)
wp wc-shippo-shipping settings set someCode "0042" --string
```

The settings you'll reach for most:

| Key | Values | What it does |
|-----|--------|--------------|
| `testApiToken` / `liveApiToken` | string | Sandbox / production API key (Shippo example — see note below) |
| `sandbox` | `yes`\|`no` | Use the test or the live key |
| `enableLiveShippingRates` | `yes`\|`no` | Turn live rate calculation on or off |
| `shippingZones` | `yes`\|`no` | Configure settings per zone |
| `debug` | `yes`\|`no` | Log rate requests and responses |
| `priceAdjustment` | number | Flat amount added to every quoted rate |
| `priceAdjustmentPercent` | number | Multiplier applied to every quoted rate |
| `origin` | JSON | Origin (From) address |
| `boxes` | JSON | Box configuration |
| `services` | JSON | Enabled carrier services |

The API-credential setting keys differ per carrier. Shippo uses `testApiToken` / `liveApiToken`, while others such as ShipStation use an `apiKey` + `apiSecret` pair. The reliable way to see a plugin's exact keys is `wp <namespace> settings list`.

You can also set most of these through `wp-config.php` constants or environment variables. See [Server-Level Configuration](wp-config.md).

---

## Product Readiness

Works in both versions. Rates are only as good as your product data, so it's worth fixing gaps before you go live.

```bash
# Readiness summary: completeness %, what's missing, suspicious values
wp wc-shippo-shipping products audit

# Add the actual lists of products that need fixing (lists only surface with --format=json;
# in the default table format, --detailed still shows just the summary)
wp wc-shippo-shipping products audit --detailed --format=json
wp wc-shippo-shipping products audit --detailed --format=json --limit=0   # no truncation on large catalogs

# Weight/dimension stats and a size distribution
wp wc-shippo-shipping products stats

# How many products fit a given box?
wp wc-shippo-shipping products fit-box --length=12 --width=8 --height=6

# Order patterns: items per order, top destinations, domestic vs international
wp wc-shippo-shipping products orders

# Bound the scan: --limit defaults to 1000 (0 = unlimited; warns when the cap is hit),
# --since filters to orders on or after a date
wp wc-shippo-shipping products orders --limit=5000 --since=2025-01-01
```

---

## Shipping Rates

Works in both versions.

```bash
# Live quote from the carrier API
wp wc-shippo-shipping rates quote --products="9405:2,9406:1" --destination="123 Main St, New York, NY 10001, US"

# Give slow carriers or big carts more time
wp wc-shippo-shipping rates quote --products="9405" --destination="..." --timeout=60

# Multi-vendor: quote against a specific vendor's origin
wp wc-shippo-shipping rates quote --products="9405" --destination="..." --vendor=12

# Saved/cached rates for an existing order
wp wc-shippo-shipping rates get --order=12345
```

There's no separate "test connection" command. A successful `rates quote` is the proof that the carrier API is reachable.

---

## Boxes (PRO)

Requires the PRO version.

```bash
# Carrier box presets with dimensions (no API key needed) — ready to paste into `settings set boxes`
wp wc-shippo-shipping boxes presets
wp wc-shippo-shipping boxes presets --carrier=USPS

# Available package type IDs from the carrier adapter's catalog
# (only requires that credentials are configured so the adapter initializes —
# this is NOT a connectivity check; use `rates quote` for that)
wp wc-shippo-shipping boxes types
```

---

## Shipments & Carrier Orders (PRO)

Requires the PRO version. Batch commands take `--orders=<ids>` (comma-separated WooCommerce order IDs). Per-shipment commands take a positional `<shipment-id>` plus `--order=<id>`.

```bash
# Create carrier orders for one or more WooCommerce orders (batch)
wp wc-shippo-shipping orders create --orders=123,456

# List shipments for one order
wp wc-shippo-shipping shipments list --order=123

# Get one shipment
wp wc-shippo-shipping shipments get <shipment-id> --order=123

# Create shipments for one or more orders (batch)
wp wc-shippo-shipping shipments create --orders=123,456

# Pull fresh shipment data from the carrier
wp wc-shippo-shipping shipments fetch <shipment-id> --order=123
```

---

## Shipping Labels (PRO)

Requires the PRO version.

```bash
# Buy labels for one or more orders (creates shipments first if needed)
wp wc-shippo-shipping labels purchase --orders=123,456

# Refund a purchased label
wp wc-shippo-shipping labels refund <shipment-id> --order=123

# Download one label PDF (--output is optional; defaults to a temp file)
wp wc-shippo-shipping labels download <shipment-id> --order=123 --output=/path/label.pdf

# Merge labels for many orders into a single PDF (batch; --output is required)
wp wc-shippo-shipping labels pdf --orders=123,456,789 --output=/path/batch-labels.pdf
```

> **Multi-vendor stores:** the PRO `shipments`, `labels`, `track`, and `orders create` commands accept `--vendor=<id>` to scope the action to a single vendor's portion of an order (default `0` = the whole order).

---

## Tracking (PRO)

Requires the PRO version.

```bash
# Tracking overview and event history for a shipment
wp wc-shippo-shipping track <shipment-id> --order=123
wp wc-shippo-shipping track <shipment-id> --order=123 --format=json
```

---

## Manifests / End-of-Day (PRO)

Requires the PRO version and a carrier that supports manifests.

```bash
# Create a manifest for specific shipments (you list the shipment IDs)
wp wc-shippo-shipping manifest create --shipments=abc123,def456
wp wc-shippo-shipping manifest create --shipments=abc123,def456 --ship-date=2026-01-15
```

---

## License Management (PRO)

Requires the PRO version. Handy for activating a license inside a deployment script.

```bash
# Current license status
wp wc-shippo-shipping license status

# Activate a key (it's a positional argument, not a --key= flag)
wp wc-shippo-shipping license activate YOUR-LICENSE-KEY

# Deactivate and clear the stored key
wp wc-shippo-shipping license deactivate
```

---

## Automation Examples

Fail a deploy if shipping isn't configured correctly:

```bash
#!/usr/bin/env bash
set -e
NS=wc-shippo-shipping
wp "$NS" validate --format=json 2>/dev/null \
  | jq -e 'all(.[]; .status != "FAIL")' >/dev/null \
  || { echo "Shipping configuration has failures:"; wp "$NS" validate; exit 1; }
```

Provision a store from a script:

```bash
NS=wc-shippo-shipping
wp "$NS" settings set liveApiToken "$SHIPPO_API_KEY"
wp "$NS" settings set sandbox no
wp "$NS" settings set enableLiveShippingRates yes
wp "$NS" settings set origin "$(cat origin.json)"
wp "$NS" settings set boxes "$(wp "$NS" boxes presets --carrier=USPS --format=json)"
wp "$NS" validate
```

Buy and print labels for a batch of orders:

```bash
NS=wc-shippo-shipping
wp "$NS" labels purchase --orders=123,124,125
wp "$NS" labels pdf --orders=123,124,125 --output=todays-labels.pdf
```

---

## See Also

- [AI Agent Skills](ai-skills.md) — let an assistant run these commands for you
- [Hooks API Reference](hooks-api.md) — 22 public hooks for customizing rates, labels, and tracking
- [Plugin Hook Prefixes](hooks-by-plugin.md) — namespace and prefix mapping for every supported plugin
- [Server-Level Configuration](wp-config.md) — pre-set values through `wp-config.php` constants or environment variables

---

## Related Plugins

Every 1TeamSoftware WooCommerce shipping plugin exposes the same WP-CLI command set — only the namespace changes. Learn it once, automate all nine:

| Plugin | Carrier / Platform | Free Version |
|--------|-------------------|-------------|
| [Shippo Shipping PRO](https://1teamsoftware.com/product/wc-shippo-shipping-pro/) | Shippo — 100+ carriers worldwide | [WordPress.org](https://wordpress.org/plugins/wc-shippo-shipping/) |
| [EasyPost Shipping PRO](https://1teamsoftware.com/product/wc-easypost-shipping-pro/) | EasyPost — 100+ carriers worldwide | [WordPress.org](https://wordpress.org/plugins/wc-easypost-shipping/) |
| [FedEx Shipping PRO](https://1teamsoftware.com/product/wc-fedex-shipping-pro/) | FedEx | — |
| [UPS Shipping PRO](https://1teamsoftware.com/product/wc-ups-shipping-pro/) | UPS | — |
| [ShipStation Shipping PRO](https://1teamsoftware.com/product/wc-shipstation-shipping-pro/) | ShipStation | [WordPress.org](https://wordpress.org/plugins/wc-shipstation-shipping/) |
| [ShipEngine Shipping PRO](https://1teamsoftware.com/product/wc-shipengine-shipping-pro/) | ShipEngine | [WordPress.org](https://wordpress.org/plugins/wc-shipengine-shipping/) |
| [Shipmondo Shipping PRO](https://1teamsoftware.com/product/wc-shipmondo-shipping-pro/) | Shipmondo | [WordPress.org](https://wordpress.org/plugins/wc-shipmondo-shipping/) |
| [ChitChats Shipping PRO](https://1teamsoftware.com/product/wc-chitchats-shipping-pro/) | ChitChats | — |
| [Stallion Express Shipping PRO](https://1teamsoftware.com/product/wc-stallionexpress-shipping-pro/) | Stallion Express | — |

---

*Nine carriers, one command vocabulary. Swap `wc-shippo-shipping` for any namespace above and your scripts just keep working. [PRO versions](https://1teamsoftware.com/product-category/woocommerce-plugins/) add the labels, shipments, tracking, and manifest commands.*

[🏠 Home](../README.md) | [Next ▶ AI Agent Skills](ai-skills.md)
