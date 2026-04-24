
# WooCommerce Shipping Hooks API — 22 Filters & Actions (Shippo, EasyPost, FedEx, UPS & More)

All 1TeamSoftware WooCommerce shipping plugins — including [Shippo](https://1teamsoftware.com/product/wc-shippo-shipping-pro/), [EasyPost](https://1teamsoftware.com/product/wc-easypost-shipping-pro/), [FedEx](https://1teamsoftware.com/product/wc-fedex-shipping-pro/), [UPS](https://1teamsoftware.com/product/wc-ups-shipping-pro/), [ShipStation](https://1teamsoftware.com/product/wc-shipstation-shipping-pro/), [ShipEngine](https://1teamsoftware.com/product/wc-shipengine-shipping-pro/), [Shipmondo](https://1teamsoftware.com/product/wc-shipmondo-shipping-pro/), [ChitChats](https://1teamsoftware.com/product/wc-chitchats-shipping-pro/), and [Stallion Express](https://1teamsoftware.com/product/wc-stallionexpress-shipping-pro/) — share a common hooks API built on WordPress `add_filter` and `add_action`.

This reference documents all **22 public hooks** that WooCommerce agencies and developers can use to customize shipping behavior without modifying plugin source code.

## How Hook Names Work

Every hook name follows the pattern:

```
{plugin_id}_{hook_suffix}
```

For example, the Shippo plugin uses the prefix `wc_shippo_shipping`, so the services hook becomes `wc_shippo_shipping_getServices`. The FedEx plugin uses `wc_fedex_shipping`, making it `wc_fedex_shipping_getServices`.

See the [complete prefix table](hooks-by-plugin.md) for all supported plugins.

> **Tip:** The prefix is the shipping method ID and stays the same across all WooCommerce shipping zones. You don't need different hooks for different zones.

### WordPress Hook Argument Counts

When hooking into filters or actions with multiple parameters, you must specify both the priority and the argument count:

```php
// Hook with 3 arguments — must specify priority (10) and arg count (3)
add_action('wc_shippo_shipping_shipment_cancelled', 'my_handler', 10, 3);
```

If you omit the argument count, WordPress only passes the first parameter to your callback.

---

## Customize Available Services and Carriers

Use these hooks to control which shipping services and carriers appear in your WooCommerce store settings and checkout.

### `{id}_getServices`

**Type:** Filter
**Applies to:** Admin settings, checkout rate calculation

| Parameter | Type | Description |
|-----------|------|-------------|
| `$services` | `array` | Associative array of `serviceCode => serviceLabel` |

**Returns:** `array` — Modified services array.

Filter the list of shipping services returned by the carrier API. Use this to remove services you don't want to offer, rename them, or reorder them before they appear in admin settings or checkout.

**Example: Remove a specific service from checkout**

```php
add_filter('wc_shippo_shipping_getServices', function (array $services): array {
    unset($services['usps_priority']);
    return $services;
});
```

**Example: Only allow specific services**

```php
add_filter('wc_easypost_shipping_getServices', function (array $services): array {
    $allowed = ['FedExGround', 'FedExHomeDelivery', 'UPSGround'];
    return array_intersect_key($services, array_flip($allowed));
});
```

---

### `{id}_getCarriers`

**Type:** Filter
**Applies to:** Admin settings, rate calculation

| Parameter | Type | Description |
|-----------|------|-------------|
| `$carriers` | `array` | Associative array of `carrierCode => carrierLabel` |

**Returns:** `array` — Modified carriers array.

Filter the list of carriers returned by the shipping provider. Useful when your account has access to carriers you don't want to use.

**Example: Remove a carrier**

```php
add_filter('wc_shippo_shipping_getCarriers', function (array $carriers): array {
    unset($carriers['dhl_express']);
    return $carriers;
});
```

---

### `{id}_service_name`

**Type:** Filter
**Applies to:** Checkout display, order details

| Parameter | Type | Description |
|-----------|------|-------------|
| `$name` | `string` | The service display name |
| `$serviceCode` | `string` | The service code identifier |

**Returns:** `string` — Modified display name.

Customize the shipping service name shown to customers at checkout and in order emails.

**Example: Use friendlier names at checkout**

```php
add_filter('wc_shippo_shipping_service_name', function (string $name, string $code): string {
    $friendly = [
        'usps_priority' => 'Standard Shipping (2-3 days)',
        'usps_express'  => 'Express Shipping (1-2 days)',
    ];
    return $friendly[$code] ?? $name;
}, 10, 2);
```

---

## Modify WooCommerce Shipping Rates at Checkout

These hooks let you programmatically customize shipping rates during WooCommerce checkout calculation.

### `{id}_calculate_shipping`

**Type:** Filter
**Applies to:** Cart and checkout rate calculation

| Parameter | Type | Description |
|-----------|------|-------------|
| `$package` | `array` | WooCommerce shipping package containing cart items, destination, and rates |

**Returns:** `array` — Modified shipping package.

Intercept the shipping package before rates are fetched from the carrier API. Use this to modify package contents, destination data, or add surcharges to calculated rates.

**Example: Add a handling fee to all rates**

```php
add_filter('wc_shippo_shipping_calculate_shipping', function (array $package): array {
    // Add $5 handling fee logic here
    return $package;
});
```

> **See also:** For modifying rates after they're calculated, WooCommerce provides the core [`woocommerce_package_rates`](https://developer.woocommerce.com/docs/features/shipping/shipping-method-api/) filter which works with all shipping methods.

---

## Customize Shipment Creation and Shipping Labels

These hooks fire during the shipment creation workflow when you create shipping labels from the WooCommerce admin order page. They follow this lifecycle order:

```
1. getServicesForOrder    → Choose available services for the order
2. preparePackageForOrder → Set package dimensions, weight, customs data
3. createShipmentForParcel → Customize per-parcel shipment parameters
4. purchasePostageForShipment → Last-minute modifications before label purchase
5. createPdfFileForOrderShipments → Customize the generated PDF label file
```

### `{id}_getServicesForOrder`

**Type:** Filter
**Applies to:** Admin shipment creation (order meta box)

| Parameter | Type | Description |
|-----------|------|-------------|
| `$services` | `array` | Available services (empty array as default) |
| `$orderId` | `int` | WooCommerce order ID |
| `$package` | `array` | Package data (dimensions, weight, items) |
| `$vendorId` | `int` | Vendor ID (0 for non-multivendor setups) |

**Returns:** `array` — Modified services list.

Filter which shipping services are available when creating a shipment from the admin order page. This is different from `getServices` — that hook affects the global service list, while this one is order-specific.

**Example: Restrict heavy orders to ground services only**

```php
add_filter('wc_shippo_shipping_getServicesForOrder', function (array $services, int $orderId, array $package, int $vendorId): array {
    if (($package['weight'] ?? 0) > 70) {
        // Only allow ground services for heavy packages
        return array_filter($services, function ($label, $code) {
            return stripos($code, 'ground') !== false;
        }, ARRAY_FILTER_USE_BOTH);
    }
    return $services;
}, 10, 4);
```

---

### `{id}_preparePackageForOrder`

**Type:** Filter
**Applies to:** Admin shipment creation

| Parameter | Type | Description |
|-----------|------|-------------|
| `$package` | `array` | Package data (dimensions, weight, items) |
| `$orderId` | `int` | WooCommerce order ID |
| `$vendorId` | `int` | Vendor ID (0 for non-multivendor) |

**Returns:** `array` — Modified package data.

Modify package dimensions, weight, or customs data before the shipment is created with the carrier.

**Example: Set minimum weight**

```php
add_filter('wc_shippo_shipping_preparePackageForOrder', function (array $package, int $orderId, int $vendorId): array {
    $package['weight'] = max($package['weight'], 1.0); // Minimum 1 lb
    return $package;
}, 10, 3);
```

---

### `{id}_createShipmentForParcel`

**Type:** Filter
**Applies to:** Admin shipment creation

| Parameter | Type | Description |
|-----------|------|-------------|
| `$shipment` | `array` | Shipment data (empty array as default) |
| `$package` | `array` | Package data |
| `$orderId` | `int` | WooCommerce order ID |
| `$vendorId` | `int` | Vendor ID (0 for non-multivendor) |

**Returns:** `array` — Modified shipment data.

Customize the shipment data for each parcel before it's sent to the carrier API. Use this to add insurance, signature requirements, or special handling instructions.

**Example: Add insurance to high-value orders**

```php
add_filter('wc_shippo_shipping_createShipmentForParcel', function (array $shipment, array $package, int $orderId, int $vendorId): array {
    $order = wc_get_order($orderId);
    if ($order && $order->get_total() > 200) {
        $shipment['insurance'] = [
            'amount'   => $order->get_total(),
            'currency' => $order->get_currency(),
        ];
    }
    return $shipment;
}, 10, 4);
```

---

### `{id}_purchasePostageForShipment`

**Type:** Filter
**Applies to:** Label purchase

| Parameter | Type | Description |
|-----------|------|-------------|
| `$shipment` | `array` | Shipment data |
| `$orderId` | `int` | WooCommerce order ID |
| `$vendorId` | `int` | Vendor ID (0 for non-multivendor) |

**Returns:** `array` — Modified shipment data.

Last chance to modify shipment data before the shipping label is purchased from the carrier. Fires after `createShipmentForParcel` and right before the API call.

**Example: Override label format**

```php
add_filter('wc_shippo_shipping_purchasePostageForShipment', function (array $shipment, int $orderId, int $vendorId): array {
    $shipment['label_format'] = 'PDF_4x6';
    return $shipment;
}, 10, 3);
```

---

### `{id}_createPdfFileForOrderShipments`

**Type:** Filter
**Applies to:** Label PDF generation

| Parameter | Type | Description |
|-----------|------|-------------|
| `$file` | `array` | File data (empty array as default) |
| `$orderShipments` | `array` | All shipments for the order |

**Returns:** `array` — Modified file data.

Customize the PDF file generated when downloading shipping labels for an order.

**Example: Log label generation**

```php
add_filter('wc_shippo_shipping_createPdfFileForOrderShipments', function (array $file, array $orderShipments): array {
    error_log('Generated label PDF for ' . count($orderShipments) . ' shipments');
    return $file;
}, 10, 2);
```

---

### `{id}_getShipDate`

**Type:** Filter
**Applies to:** Shipment creation

| Parameter | Type | Description |
|-----------|------|-------------|
| `$shipDate` | `string\|null` | Ship date in `Y-m-d` format, or null for today |

**Returns:** `string|null` — Modified date or null.

Override the ship date used when creating shipments. Useful for skipping weekends, holidays, or adding processing time.

**Example: Skip weekend ship dates**

```php
add_filter('wc_shippo_shipping_getShipDate', function (?string $date): ?string {
    $timestamp = strtotime($date ?? 'today');
    $dayOfWeek = (int) date('N', $timestamp);
    if ($dayOfWeek >= 6) { // Saturday or Sunday
        return date('Y-m-d', strtotime('next Monday', $timestamp));
    }
    return $date;
});
```

---

## React to Shipment Lifecycle Events

These hooks fire when shipment status changes occur. Use them to integrate WooCommerce shipping events with external systems like ERPs, custom notification services, or analytics platforms.

> **Actions vs Filters:** Most lifecycle hooks are **actions** (fire-and-forget). The exception is `shipment_postage_purchased`, which is a **filter** because it lets you modify shipment data after label purchase.

### `{id}_shipment_status_changed`

**Type:** Action
**Fires when:** Any shipment status transition (e.g., created, in transit, delivered)

| Parameter | Type | Description |
|-----------|------|-------------|
| `$orderId` | `int` | WooCommerce order ID |
| `$shipment` | `array` | Shipment data including new status |

The most commonly used lifecycle hook. Fires on every status change, letting you trigger custom workflows based on shipment progress.

**Example: Send delivery notification to external system**

```php
add_action('wc_shippo_shipping_shipment_status_changed', function (int $orderId, array $shipment): void {
    if ($shipment['status'] === 'DELIVERED') {
        wp_remote_post('https://erp.example.com/api/deliveries', [
            'body' => wp_json_encode([
                'order_id'        => $orderId,
                'tracking_number' => $shipment['tracking_number'] ?? '',
                'delivered_at'    => current_time('mysql'),
            ]),
            'headers' => ['Content-Type' => 'application/json'],
        ]);
    }
}, 10, 2);
```

---

### `{id}_shipment_postage_purchased`

**Type:** Filter
**Fires when:** A shipping label is successfully purchased

| Parameter | Type | Description |
|-----------|------|-------------|
| `$shipment` | `array` | Shipment data with tracking number, label URL, and cost |
| `$orderId` | `int` | WooCommerce order ID |

**Returns:** `array` — Modified shipment data.

This is a **filter** (not an action) because you can modify the shipment data after purchase — for example, to append custom metadata before it's stored.

**Example: Log purchased labels**

```php
add_filter('wc_shippo_shipping_shipment_postage_purchased', function (array $shipment, int $orderId): array {
    error_log(sprintf(
        'Label purchased for order #%d: %s (cost: %s)',
        $orderId,
        $shipment['tracking_number'] ?? 'N/A',
        $shipment['postage'] ?? 'N/A'
    ));
    return $shipment;
}, 10, 2);
```

---

### `{id}_shipment_postage_refunded`

**Type:** Action
**Fires when:** Shipping label postage is refunded

| Parameter | Type | Description |
|-----------|------|-------------|
| `$orderId` | `int` | WooCommerce order ID |
| `$shipmentId` | `string` | Shipment identifier |
| `$shipment` | `array` | Shipment data |

**Example: Update accounting system on refund**

```php
add_action('wc_shippo_shipping_shipment_postage_refunded', function (int $orderId, string $shipmentId, array $shipment): void {
    // Notify accounting system
    do_action('my_accounting_shipping_refund', $orderId, $shipment['postage'] ?? 0);
}, 10, 3);
```

---

### `{id}_shipment_cancelled`

**Type:** Action
**Fires when:** A shipment is cancelled

| Parameter | Type | Description |
|-----------|------|-------------|
| `$orderId` | `int` | WooCommerce order ID |
| `$shipmentId` | `string` | Shipment identifier |
| `$shipment` | `array` | Shipment data |

**Example: Add order note on cancellation**

```php
add_action('wc_shippo_shipping_shipment_cancelled', function (int $orderId, string $shipmentId, array $shipment): void {
    $order = wc_get_order($orderId);
    if ($order) {
        $order->add_order_note(sprintf('Shipment %s cancelled.', $shipmentId));
    }
}, 10, 3);
```

---

### `{id}_shipment_delete`

**Type:** Action
**Fires when:** A shipment is permanently deleted

| Parameter | Type | Description |
|-----------|------|-------------|
| `$orderId` | `int` | WooCommerce order ID |
| `$shipmentId` | `string` | Shipment identifier |
| `$shipment` | `array` | Shipment data |

**Example: Clean up external references**

```php
add_action('wc_shippo_shipping_shipment_delete', function (int $orderId, string $shipmentId, array $shipment): void {
    delete_post_meta($orderId, '_custom_shipment_ref_' . $shipmentId);
}, 10, 3);
```

---

## Customize Shipment Tracking Display

Control how shipment tracking information appears to customers in their WooCommerce account and order emails.

### `{id}_getShipmentUrl`

**Type:** Filter
**Applies to:** Customer-facing tracking links

| Parameter | Type | Description |
|-----------|------|-------------|
| `$url` | `string` | Carrier tracking URL (empty string if unavailable) |
| `$shipment` | `array` | Shipment data including tracking number and carrier |

**Returns:** `string` — Modified tracking URL.

Override the default carrier tracking URL with a custom tracking page. Useful for branded tracking experiences or aggregated tracking services.

**Example: Use branded tracking page**

```php
add_filter('wc_shippo_shipping_getShipmentUrl', function (string $url, array $shipment): string {
    $tracking = $shipment['tracking_number'] ?? '';
    if (!empty($tracking)) {
        return 'https://mystore.com/track/' . urlencode($tracking);
    }
    return $url;
}, 10, 2);
```

---

### `{id}_max_tracking_events`

**Type:** Filter
**Applies to:** Tracking history display

| Parameter | Type | Description |
|-----------|------|-------------|
| `$maxEvents` | `int` | Maximum number of tracking events to store and display |

**Returns:** `int` — Modified maximum.

Control how many tracking history entries are stored per shipment. Higher values show more detailed tracking timelines but use more database storage.

**Example: Show more tracking detail**

```php
add_filter('wc_shippo_shipping_max_tracking_events', function (int $max): int {
    return 50; // Default is typically 10-20
});
```

---

## Customize Data Lookups

Filter the reference data (currencies, package types, label formats, content types) used throughout the plugin's admin interface and API calls.

### `{id}_getCurrencies`

**Type:** Filter

| Parameter | Type | Description |
|-----------|------|-------------|
| `$currencies` | `array` | Associative array of `currencyCode => currencyLabel` |

**Returns:** `array` — Modified currencies.

**Example: Restrict to specific currencies**

```php
add_filter('wc_shippo_shipping_getCurrencies', function (array $currencies): array {
    return array_intersect_key($currencies, array_flip(['USD', 'CAD', 'EUR']));
});
```

---

### `{id}_getDefaultCurrency`

**Type:** Filter

| Parameter | Type | Description |
|-----------|------|-------------|
| `$currency` | `string` | Default currency code |

**Returns:** `string` — Modified currency code.

**Example: Force Euro as default**

```php
add_filter('wc_shippo_shipping_getDefaultCurrency', function (string $currency): string {
    return 'EUR';
});
```

---

### `{id}_getPackageTypes`

**Type:** Filter

| Parameter | Type | Description |
|-----------|------|-------------|
| `$packageTypes` | `array` | Associative array of `typeCode => typeLabel` |

**Returns:** `array` — Modified package types.

**Example: Remove pallet option**

```php
add_filter('wc_shippo_shipping_getPackageTypes', function (array $types): array {
    unset($types['pallet']);
    return $types;
});
```

---

### `{id}_getLabelFormats`

**Type:** Filter

| Parameter | Type | Description |
|-----------|------|-------------|
| `$formats` | `array` | Associative array of `formatCode => formatLabel` |

**Returns:** `array` — Modified label formats.

**Example: Only offer PDF labels**

```php
add_filter('wc_shippo_shipping_getLabelFormats', function (array $formats): array {
    return ['PDF' => 'PDF'];
});
```

---

### `{id}_getContentTypes`

**Type:** Filter

| Parameter | Type | Description |
|-----------|------|-------------|
| `$contentTypes` | `array` | Associative array of `typeCode => typeLabel` |

**Returns:** `array` — Modified content types for customs declarations.

**Example: Remove gift option for international shipments**

```php
add_filter('wc_shippo_shipping_getContentTypes', function (array $types): array {
    unset($types['gift']);
    return $types;
});
```

---

## Quick Reference Table

| Hook Suffix | Type | Parameters | Use Case |
|-------------|------|------------|----------|
| `getServices` | Filter | 1 | Control which shipping services are available |
| `getCarriers` | Filter | 1 | Control which carriers are available |
| `service_name` | Filter | 2 | Customize service display names |
| `calculate_shipping` | Filter | 1 | Modify rates at checkout |
| `getServicesForOrder` | Filter | 4 | Filter services per order |
| `preparePackageForOrder` | Filter | 3 | Modify package dimensions/weight |
| `createShipmentForParcel` | Filter | 4 | Customize shipment before creation |
| `purchasePostageForShipment` | Filter | 3 | Last-minute label modifications |
| `createPdfFileForOrderShipments` | Filter | 2 | Customize label PDF |
| `getShipDate` | Filter | 1 | Override ship date |
| `shipment_status_changed` | Action | 2 | React to status changes |
| `shipment_postage_purchased` | Filter | 2 | Post-purchase shipment modifications |
| `shipment_postage_refunded` | Action | 3 | Handle label refunds |
| `shipment_cancelled` | Action | 3 | Handle cancellations |
| `shipment_delete` | Action | 3 | Handle shipment deletion |
| `getShipmentUrl` | Filter | 2 | Customize tracking URLs |
| `max_tracking_events` | Filter | 1 | Control tracking history depth |
| `getCurrencies` | Filter | 1 | Filter available currencies |
| `getDefaultCurrency` | Filter | 1 | Override default currency |
| `getPackageTypes` | Filter | 1 | Filter package type options |
| `getLabelFormats` | Filter | 1 | Filter label format options |
| `getContentTypes` | Filter | 1 | Filter customs content types |

---

## Frequently Asked Questions

### How do I customize shipping rates for a specific WooCommerce shipping zone?

These hooks apply globally across all shipping zones. The hook prefix (`wc_shippo_shipping`) stays the same regardless of which zone triggers the calculation. To conditionally modify rates per zone, inspect the `$package['destination']` data inside your callback.

### Do these hooks work with the WooCommerce block checkout?

Yes. These hooks fire at the PHP/server level during rate calculation and shipment operations, which are independent of whether your store uses the classic checkout or the block-based checkout.

### Can I use the same hook code for different carrier plugins?

Yes. The hooks are identical across all 1TeamSoftware shipping plugins — only the prefix changes. You can write a generic helper that accepts the prefix:

```php
function my_customize_services(string $pluginId): void {
    add_filter($pluginId . '_getServices', function (array $services): array {
        unset($services['usps_media_mail']);
        return $services;
    });
}

// Apply to multiple plugins
my_customize_services('wc_shippo_shipping');
my_customize_services('wc_easypost_shipping');
```

### Where should I put this custom hook code?

Add your hook code in one of these locations:
- Your theme's `functions.php` file (simplest, but lost on theme change)
- A custom plugin (recommended for production)
- A code snippets plugin like [Code Snippets](https://wordpress.org/plugins/code-snippets/)

### How do I find the correct plugin prefix for my shipping plugin?

See the [Plugin Hook Prefixes](hooks-by-plugin.md) page for the complete mapping of all supported plugins and their hook prefixes.

---

## Related Plugins

These hooks work across all 1TeamSoftware WooCommerce shipping plugins. Each plugin connects to a different carrier API while sharing the same hook architecture:

| Plugin | Carrier | Hook Prefix |
|--------|---------|-------------|
| [Shippo Shipping PRO](https://1teamsoftware.com/product/wc-shippo-shipping-pro/) | Shippo (100+ carriers) | `wc_shippo_shipping` |
| [EasyPost Shipping PRO](https://1teamsoftware.com/product/wc-easypost-shipping-pro/) | EasyPost (100+ carriers) | `wc_easypost_shipping` |
| [FedEx Shipping PRO](https://1teamsoftware.com/product/wc-fedex-shipping-pro/) | FedEx | `wc_fedex_shipping` |
| [UPS Shipping PRO](https://1teamsoftware.com/product/wc-ups-shipping-pro/) | UPS | `wc_ups_shipping` |
| [ShipStation Shipping PRO](https://1teamsoftware.com/product/wc-shipstation-shipping-pro/) | ShipStation | `wc_shipstation_shipping` |
| [ShipEngine Shipping PRO](https://1teamsoftware.com/product/wc-shipengine-shipping-pro/) | ShipEngine | `wc_shipengine_shipping` |
| [Shipmondo Shipping PRO](https://1teamsoftware.com/product/wc-shipmondo-shipping-pro/) | Shipmondo | `wc_shipmondo_shipping` |
| [ChitChats Shipping PRO](https://1teamsoftware.com/product/wc-chitchats-shipping-pro/) | ChitChats | `wc_chitchats_shipping` |
| [Stallion Express Shipping PRO](https://1teamsoftware.com/product/wc-stallionexpress-shipping-pro/) | Stallion Express | `wc_stallionexpress_shipping` |

See the [Plugin Hook Prefixes](hooks-by-plugin.md) page for more detail on writing cross-plugin hook code. To lock down API credentials and plugin settings for managed deployments, see [Server-Level Configuration](wp-config.md).

---

*[1TeamSoftware WooCommerce Shipping PRO plugins](https://1teamsoftware.com/product-category/woocommerce-plugins/) unlock the full hooks API — shipment creation, label purchase, tracking display, and lifecycle event hooks are PRO-only features.*

[🏠 Home](../README.md) | [Next ▶ Plugin Hook Prefixes](hooks-by-plugin.md)
