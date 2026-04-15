
# WooCommerce Shipping Hook Prefixes — Shippo, EasyPost, FedEx, UPS, ShipStation, ShipEngine & More

All 1TeamSoftware WooCommerce shipping plugins share the same [Hooks API](hooks-api.md) with 22 public hooks. The only difference between plugins is the **hook prefix** — the string that identifies which shipping method fires the hook.

## Plugin Prefix Table

| Plugin | Hook Prefix | Example Hook Name |
|--------|------------|-------------------|
| [Shippo Shipping](https://1teamsoftware.com/product/wc-shippo-shipping-pro/) | `wc_shippo_shipping` | `wc_shippo_shipping_getServices` |
| [EasyPost Shipping](https://1teamsoftware.com/product/wc-easypost-shipping-pro/) | `wc_easypost_shipping` | `wc_easypost_shipping_getServices` |
| [FedEx Shipping](https://1teamsoftware.com/product/wc-fedex-shipping-pro/) | `wc_fedex_shipping` | `wc_fedex_shipping_getServices` |
| [UPS Shipping](https://1teamsoftware.com/product/wc-ups-shipping-pro/) | `wc_ups_shipping` | `wc_ups_shipping_getServices` |
| [ShipStation Shipping](https://1teamsoftware.com/product/wc-shipstation-shipping-pro/) | `wc_shipstation_shipping` | `wc_shipstation_shipping_getServices` |
| [ShipEngine Shipping](https://1teamsoftware.com/product/wc-shipengine-shipping-pro/) | `wc_shipengine_shipping` | `wc_shipengine_shipping_getServices` |
| [Shipmondo Shipping](https://1teamsoftware.com/product/wc-shipmondo-shipping-pro/) | `wc_shipmondo_shipping` | `wc_shipmondo_shipping_getServices` |
| [ChitChats Shipping](https://1teamsoftware.com/product/wc-chitchats-shipping-pro/) | `wc_chitchats_shipping` | `wc_chitchats_shipping_getServices` |
| [Stallion Express Shipping](https://1teamsoftware.com/product/wc-stallionexpress-shipping-pro/) | `wc_stallionexpress_shipping` | `wc_stallionexpress_shipping_getServices` |

> **Finding your prefix:** The prefix matches the plugin's text domain with hyphens replaced by underscores. For example, `wc-shippo-shipping` becomes `wc_shippo_shipping`.

## How to Write Cross-Plugin Hook Code

If you use multiple 1TeamSoftware shipping plugins, you can write a single function and apply it to all:

```php
/**
 * Apply a hook callback to one or more shipping plugins.
 *
 * @param array    $pluginIds Array of plugin prefixes.
 * @param string   $suffix    Hook suffix (e.g., 'getServices').
 * @param callable $callback  Your callback function.
 * @param int      $priority  Hook priority (default: 10).
 * @param int      $args      Number of arguments (default: 1).
 */
function my_shipping_hook(array $pluginIds, string $suffix, callable $callback, int $priority = 10, int $args = 1): void {
    foreach ($pluginIds as $id) {
        add_filter($id . '_' . $suffix, $callback, $priority, $args);
    }
}

// Example: Remove media mail from both Shippo and EasyPost
my_shipping_hook(
    ['wc_shippo_shipping', 'wc_easypost_shipping'],
    'getServices',
    function (array $services): array {
        unset($services['usps_media_mail']);
        return $services;
    }
);
```

### Apply to All Installed Plugins

```php
// Detect all installed 1TeamSoftware shipping plugins
function get_1ts_shipping_prefixes(): array {
    $prefixes = [];
    $plugins = [
        'wc-shippo-shipping/wc-shippo-shipping.php'           => 'wc_shippo_shipping',
        'wc-shippo-shipping-pro/wc-shippo-shipping-pro.php'   => 'wc_shippo_shipping',
        'wc-easypost-shipping/wc-easypost-shipping.php'       => 'wc_easypost_shipping',
        'wc-easypost-shipping-pro/wc-easypost-shipping-pro.php' => 'wc_easypost_shipping',
        'wc-fedex-shipping-pro/wc-fedex-shipping-pro.php'     => 'wc_fedex_shipping',
        'wc-ups-shipping-pro/wc-ups-shipping-pro.php'         => 'wc_ups_shipping',
        'wc-shipstation-shipping-pro/wc-shipstation-shipping-pro.php' => 'wc_shipstation_shipping',
        'wc-shipengine-shipping-pro/wc-shipengine-shipping-pro.php' => 'wc_shipengine_shipping',
        'wc-shipmondo-shipping-pro/wc-shipmondo-shipping-pro.php' => 'wc_shipmondo_shipping',
        'wc-chitchats-shipping-pro/wc-chitchats-shipping-pro.php' => 'wc_chitchats_shipping',
        'wc-stallionexpress-shipping-pro/wc-stallionexpress-shipping-pro.php' => 'wc_stallionexpress_shipping',
    ];

    foreach ($plugins as $file => $prefix) {
        if (is_plugin_active($file)) {
            $prefixes[$prefix] = true;
        }
    }

    return array_keys($prefixes);
}

// Example: Set minimum weight across all active shipping plugins
add_action('plugins_loaded', function () {
    foreach (get_1ts_shipping_prefixes() as $prefix) {
        add_filter($prefix . '_preparePackageForOrder', function (array $package, int $orderId, int $vendorId): array {
            $package['weight'] = max($package['weight'], 0.5);
            return $package;
        }, 10, 3);
    }
});
```

## Free vs Pro Hooks

The hooks API is available in both free and Pro versions of each plugin. However, some hooks only fire in Pro because the free version doesn't include the corresponding feature:

| Feature | Free | Pro | Hooks Available |
|---------|------|-----|-----------------|
| Rate calculation | Yes | Yes | `getServices`, `getCarriers`, `service_name`, `calculate_shipping`, data lookup hooks |
| Shipment creation | No | Yes | `getServicesForOrder`, `preparePackageForOrder`, `createShipmentForParcel`, `getShipDate` |
| Label purchase | No | Yes | `purchasePostageForShipment`, `createPdfFileForOrderShipments` |
| Shipment lifecycle | No | Yes | `shipment_status_changed`, `shipment_postage_purchased`, `shipment_postage_refunded`, `shipment_cancelled`, `shipment_delete` |
| Tracking display | No | Yes | `getShipmentUrl`, `max_tracking_events` |

## Related Resources

- [Hooks API Reference](hooks-api.md) — Full documentation for all 22 hooks with parameters and examples
- [Server-Level Configuration](wp-config.md) — Lock down API credentials and deploy plugin settings via wp-config.php
- [WooCommerce Shipping Method API](https://developer.woocommerce.com/docs/features/shipping/shipping-method-api/) — Official WooCommerce shipping developer docs
- [Best Multi-Carrier Shipping Solutions for WooCommerce](https://1teamsoftware.com/2025/08/17/ultimate-guide-to-woocommerce-multi-carrier-shipping-solutions-in-2025/) — Comparison guide for choosing a shipping plugin

---

## Related Plugins

All plugins listed below share the same hook API documented on this site. Click any plugin to purchase or learn more:

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

*All 1TeamSoftware WooCommerce shipping plugins share the same hook API. [PRO versions](https://1teamsoftware.com/product-category/woocommerce-plugins/) add shipment creation, label purchase, tracking display, and lifecycle event hooks not available in the free plugins.*

[🏠 Home](../README.md) | [◀ Hooks API Reference](hooks-api.md) | [Next ▶ wp-config.php Configuration](wp-config.md)
