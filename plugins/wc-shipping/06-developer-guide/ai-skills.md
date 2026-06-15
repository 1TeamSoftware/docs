---
title: "AI Agent Skills — WooCommerce Shipping Plugins | 1TeamSoftware"
description: "How to set up and troubleshoot 1TeamSoftware WooCommerce shipping plugins with an AI assistant. Covers what the bundled skills are, how Claude Code, Cursor, and Copilot use them, example prompts, and a full setup walkthrough."
---

# AI Agent Skills

Setting up a shipping plugin means a lot of small, fiddly steps: enter the API key, set the origin address, pick which services to offer, build a box list, check that products have weights and dimensions, then prove that live rates actually come back. You can do all of it by hand. Or you can describe what you want and let an AI assistant do it for you.

Every 1TeamSoftware shipping plugin ships with a set of **skills** for exactly that. A skill is a written playbook the assistant reads and follows. It tells the assistant which [WP-CLI commands](wp-cli.md) to run, in what order, and how to read the results — so you get the same careful setup every time, without learning the commands yourself.

## What a Skill Actually Is

A skill is a folder with a `SKILL.md` file inside it. The file is plain Markdown: a description of a goal, the steps to reach it, and the commands involved. Nothing runs on its own. The assistant reads the relevant skill, then runs the plugin's WP-CLI commands on your site to get the job done.

The skills live inside the plugin once it's installed:

```
wp-content/plugins/<plugin-folder>/skills/1teamsoftware-wc-shipping/skills/
```

So if you have the Shippo plugin, look in `wp-content/plugins/wc-shippo-shipping-pro/skills/1teamsoftware-wc-shipping/skills/`. The free version carries the free skills; the PRO version carries those plus the PRO ones.

## Which Assistants This Works With

Any coding assistant that can read files in your project and run shell commands can use these skills. That includes:

- **Claude Code** — reads the `skills/` folder directly. This is the smoothest experience.
- **Cursor** — point it at the `skills/` directory or paste a `SKILL.md` into the chat as context.
- **GitHub Copilot (chat/agent)** — same idea: give it the skill file as context.

The assistant needs access to a terminal where `wp` (WP-CLI) can reach your WordPress install. That's usually your local dev environment or a staging server over SSH.

## The Available Skills

Six skills come with every plugin (free and PRO). Four more come with the PRO versions.

| Skill | Edition | Use it when you want to… | Example prompt |
|-------|---------|--------------------------|----------------|
| `setup-wizard` | Free | Configure the plugin from scratch, start to finish | "Set up Shippo shipping for my store." |
| `configure-shipping` | Free | Set the origin address, API keys, services, and zones | "Switch Shippo to my live API key and enable USPS Priority." |
| `audit-products` | Free | Find products missing weight or dimensions | "Which products will break rate calculation?" |
| `select-boxes` | Free | Get a box set that fits what you actually sell | "Recommend boxes based on my catalog." |
| `verify-shipping` | Free | Confirm the setup works and live rates come back | "Check that my shipping is configured correctly." |
| `cli-reference` | Free | Give the assistant the full command reference | (used automatically by the others) |
| `setup-shipping` | PRO | Full PRO setup including labels and tracking | "Set up Shippo end to end, including label printing." |
| `optimize-rates` | PRO | Tune price adjustments and which services show at checkout | "Add 8% to every rate and hide Media Mail." |
| `predict-boxes` | PRO | Model how real orders pack into boxes | "Will a 12x8x6 box cover most of my orders?" |
| `troubleshoot-shipping` | PRO | Diagnose why rates, labels, or the connection fail | "No rates are showing at checkout — find out why." |

`cli-reference` is the odd one out: you won't call it directly. The other skills pull it in when they need the exact command syntax.

## How to Use Them

### With Claude Code

1. Open your site's folder (or the plugin folder) in Claude Code, with the plugin installed and active.
2. Ask for what you want in plain language. For example: *"Set up the Shippo shipping plugin — here's my Shippo API key: shippo_live_…"*
3. Claude reads the matching skill, runs the commands, and shows you what it did at each step. It will pause and ask when it needs something only you can provide, like an API key or origin address.

### With Cursor or Copilot

1. Open the `skills/` folder so the assistant can see the `SKILL.md` files, or paste the one you need into the chat.
2. Make sure the assistant can run terminal commands against your WordPress install.
3. Ask the same way: describe the outcome, hand over any secrets it asks for, and let it run.

## A Full Walkthrough

Here's what a from-scratch setup looks like when you hand it to Claude Code. You say:

> Set up Shippo shipping for my store. API key is shippo_test_abc123 for now (sandbox). We ship from San Francisco.

Behind the scenes, the assistant works through the `setup-wizard` skill:

1. Runs `wp wc-shippo-shipping status` to see the current state.
2. Saves your API key and turns on sandbox mode with `settings set`.
3. Asks for the rest of your origin address, then saves it.
4. Runs `audit-products` to flag anything missing weights or dimensions, and lists the products you need to fix.
5. Suggests a box set from `boxes presets` and saves the ones you approve.
6. Enables the carrier services you want with `settings set services`.
7. Runs `validate`, then quotes a live rate to a test address to prove the connection works.

You answer a few questions along the way and end up with a working configuration — without touching a single command yourself. If something fails, the assistant switches to the `troubleshoot-shipping` skill and walks back through the likely causes.

## Skills and the CLI Are the Same Thing

There's no magic layer here. The skills run the exact [WP-CLI commands](wp-cli.md) documented for these plugins. Anything the assistant does, you can do by hand — and anything you script by hand, the assistant can do faster. Think of the skills as an experienced operator who already knows the commands and the order to run them in.

If you prefer to drive everything yourself, the [WP-CLI reference](wp-cli.md) has every command with examples.

## See Also

- [WP-CLI Commands](wp-cli.md) — the commands the skills run, with examples
- [Server-Level Configuration](wp-config.md) — pre-set values through `wp-config.php` so there's less to configure
- [Hooks API Reference](hooks-api.md) — change rates, labels, and tracking behavior in code

---

## Related Plugins

The same agent skills ship inside every 1TeamSoftware WooCommerce shipping plugin — teach your assistant once and it can set up any of these carriers without you reading a single doc:

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

*Your AI assistant already knows the playbook for all nine. [PRO versions](https://1teamsoftware.com/product-category/woocommerce-plugins/) add the setup, optimization, box-prediction, and troubleshooting skills on top of the free ones.*

[🏠 Home](../README.md) | [◀ WP-CLI Commands](wp-cli.md) | [Next ▶ Hooks API Reference](hooks-api.md)
