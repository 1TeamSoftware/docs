---
title: "WooCommerce Shipping Plugins Documentation — 1TeamSoftware"
description: "Documentation for all 1TeamSoftware WooCommerce shipping plugins: Shippo, EasyPost, FedEx, UPS, ShipStation, ShipEngine, Shipmondo, ChitChats, and Stallion Express."
---

# WooCommerce Shipping Plugins Documentation

This documentation covers all 1TeamSoftware WooCommerce multi-carrier shipping plugins. These plugins share a common architecture — each plugin connects WooCommerce to a different shipping carrier API while providing the same features and developer hooks.

## Supported Plugins

| Plugin | Carrier API | Free Version |
|--------|------------|--------------|
| [Shippo Shipping](https://1teamsoftware.com/product/wc-shippo-shipping-pro/) | Shippo | [WordPress.org](https://wordpress.org/plugins/wc-shippo-shipping/) |
| [EasyPost Shipping](https://1teamsoftware.com/product/wc-easypost-shipping-pro/) | EasyPost | [WordPress.org](https://wordpress.org/plugins/wc-easypost-shipping/) |
| [FedEx Shipping](https://1teamsoftware.com/product/wc-fedex-shipping-pro/) | FedEx | - |
| [UPS Shipping](https://1teamsoftware.com/product/wc-ups-shipping-pro/) | UPS | - |
| [ShipStation Shipping](https://1teamsoftware.com/product/wc-shipstation-shipping-pro/) | ShipStation | [WordPress.org](https://wordpress.org/plugins/wc-shipstation-shipping/) |
| [ShipEngine Shipping](https://1teamsoftware.com/product/wc-shipengine-shipping-pro/) | ShipEngine | [WordPress.org](https://wordpress.org/plugins/wc-shipengine-shipping/) |
| [Shipmondo Shipping](https://1teamsoftware.com/product/wc-shipmondo-shipping-pro/) | Shipmondo | [WordPress.org](https://wordpress.org/plugins/wc-shipmondo-shipping/) |
| [ChitChats Shipping](https://1teamsoftware.com/product/wc-chitchats-shipping-pro/) | ChitChats | - |
| [Stallion Express Shipping](https://1teamsoftware.com/product/wc-stallionexpress-shipping-pro/) | Stallion Express | - |

## Documentation Sections

| Section | Status | Description |
|---------|--------|-------------|
| 01 — Getting Started | Planned | Installation, activation, initial setup |
| 02 — Configuration | Planned | Settings, shipping zones, carrier accounts |
| 03 — Features | Planned | Rate calculation, address validation, multi-vendor |
| 04 — Shipping Labels | Planned | Label creation, bulk printing, manifests |
| 05 — Troubleshooting | Planned | Common issues, debugging, logs |
| [06 — Developer Guide](06-developer-guide/) | Available | Hooks API, plugin prefixes, customization |
| [Carriers](carriers/) | Planned | Per-carrier setup guides |

## Developer Guide (Available Now)

- [Hooks API Reference](06-developer-guide/hooks-api.md) — 22 public hooks for customizing shipping rates, labels, tracking, and more
- [Plugin Hook Prefixes](06-developer-guide/hooks-by-plugin.md) — Hook prefix mapping for all supported plugins, cross-plugin code patterns
- [Server-Level Configuration](06-developer-guide/wp-config.md) — Pre-configure settings via wp-config.php constants or environment variables (PRO)
