
# WooCommerce Shipping wp-config.php — Lock Settings, Hide API Keys & Deploy at Scale

All 1TeamSoftware WooCommerce shipping plugins — including [Shippo](https://1teamsoftware.com/product/wc-shippo-shipping-pro/), [EasyPost](https://1teamsoftware.com/product/wc-easypost-shipping-pro/), [FedEx](https://1teamsoftware.com/product/wc-fedex-shipping-pro/), [UPS](https://1teamsoftware.com/product/wc-ups-shipping-pro/), [ShipStation](https://1teamsoftware.com/product/wc-shipstation-shipping-pro/), [ShipEngine](https://1teamsoftware.com/product/wc-shipengine-shipping-pro/), [Shipmondo](https://1teamsoftware.com/product/wc-shipmondo-shipping-pro/), [ChitChats](https://1teamsoftware.com/product/wc-chitchats-shipping-pro/), and [Stallion Express](https://1teamsoftware.com/product/wc-stallionexpress-shipping-pro/) — support pre-configuring settings via PHP constants in `wp-config.php`.

This is a **PRO-only feature** designed for:

- **Managed hosting** — agencies deploying the same configuration across multiple client sites
- **Security** — keeping API keys and license keys out of the WordPress database and admin UI
- **Docker/CI deployments** — injecting configuration via constants defined in a mounted `wp-config.php`

> **How it works:** PHP constants defined in `wp-config.php` are merged into the plugin settings at runtime. They are never saved to the database. The admin UI reflects the locked-down state — external fields are disabled or hidden depending on the lockdown level.

---

## Quick Start

Add constants to your `wp-config.php` **before** the line `/* That's all, stop editing! */`:

```php
// Override specific settings
define('ONETEAMSOFTWARE_WC_SHIPPO_SHIPPING_SETTINGS', [
    'liveApiToken' => 'shippo_live_abc123',
    'origin' => [
        'country' => 'US',
        'postcode' => '90210',
    ],
]);

// Set the license key
define('ONETEAMSOFTWARE_WC_SHIPPO_SHIPPING_LICENSE_KEY', 'your-pro-license-key');
```

That's it. On the next page load, these settings take effect and the corresponding fields become read-only or hidden in the admin UI.

---

## Constant Naming

All constants follow the pattern:

```
ONETEAMSOFTWARE_{PLUGIN_ID_UPPER}_{SUFFIX}
```

Where `{PLUGIN_ID_UPPER}` is the plugin ID with hyphens replaced by underscores, uppercased.

### Plugin ID Reference

| Plugin | Plugin ID | Constant Prefix |
|--------|-----------|-----------------|
| Shippo | `wc-shippo-shipping` | `ONETEAMSOFTWARE_WC_SHIPPO_SHIPPING_` |
| EasyPost | `wc-easypost-shipping` | `ONETEAMSOFTWARE_WC_EASYPOST_SHIPPING_` |
| FedEx | `wc-fedex-shipping` | `ONETEAMSOFTWARE_WC_FEDEX_SHIPPING_` |
| UPS | `wc-ups-shipping` | `ONETEAMSOFTWARE_WC_UPS_SHIPPING_` |
| ShipStation | `wc-shipstation-shipping` | `ONETEAMSOFTWARE_WC_SHIPSTATION_SHIPPING_` |
| ShipEngine | `wc-shipengine-shipping` | `ONETEAMSOFTWARE_WC_SHIPENGINE_SHIPPING_` |
| Shipmondo | `wc-shipmondo-shipping` | `ONETEAMSOFTWARE_WC_SHIPMONDO_SHIPPING_` |
| ChitChats | `wc-chitchats-shipping` | `ONETEAMSOFTWARE_WC_CHITCHATS_SHIPPING_` |
| Stallion Express | `wc-stallionexpress-shipping` | `ONETEAMSOFTWARE_WC_STALLIONEXPRESS_SHIPPING_` |

---

## Available Constants

### `_SETTINGS` — Override Plugin Settings

A PHP array where keys are setting field IDs and values are the desired values. Any setting configurable through the admin UI can be overridden here.

```php
define('ONETEAMSOFTWARE_WC_SHIPPO_SHIPPING_SETTINGS', [
    'liveApiToken' => 'shippo_live_abc123',
    'sandbox' => 'no',
    'origin' => [
        'country' => 'US',
        'state' => 'CA',
        'city' => 'Los Angeles',
        'postcode' => '90210',
        'phone' => '+1 555 123 4567',
    ],
]);
```

Address fields, return address, and array settings are all nested arrays:

```php
define('ONETEAMSOFTWARE_WC_SHIPPO_SHIPPING_SETTINGS', [
    'liveApiToken' => 'shippo_live_abc123',
    // Override the return address
    'return_address' => [
        'enabled' => 'yes',
        'country' => 'US',
        'postcode' => '10001',
    ],
    // Lock the carrier list
    'carriers' => ['usps', 'fedex'],
]);
```

**UI behavior:** Each overridden field is shown as disabled (grayed out) with a notice: *"This setting is configured via wp-config.php and cannot be changed here."* Credential fields (API keys, tokens — any field with `type: password`) are completely hidden from the admin UI when they appear in `_SETTINGS`.

### `_LICENSE_KEY` — Set the License Key

```php
define('ONETEAMSOFTWARE_WC_SHIPPO_SHIPPING_LICENSE_KEY', 'your-pro-license-key');
```

**UI behavior:** The license key field and its entire section are removed from the admin UI. The license is validated automatically on `admin_init` (once per day, cached via transient).

### `_DISABLE_SETTINGS` — Lock All Settings (Read-Only)

Makes **all** settings fields read-only. The settings page is still visible so administrators can inspect the configuration, but nothing can be changed or saved.

```php
define('ONETEAMSOFTWARE_WC_SHIPPO_SHIPPING_DISABLE_SETTINGS', true);
```

Accepts truthy values: `true`, `1`, `'true'`, `'yes'`.

**UI behavior:** All fields are disabled (grayed out). A notice at the top reads: *"All settings are managed externally. This plugin's configuration is locked and cannot be changed here."* Credential fields that are externally configured are still hidden (not just disabled).

### `_HIDE_SETTINGS` — Hide the Settings Page Entirely

The maximum lockdown level. The settings page shows only a notice — no fields, no sections, no save button.

```php
define('ONETEAMSOFTWARE_WC_SHIPPO_SHIPPING_HIDE_SETTINGS', true);
```

Accepts truthy values: `true`, `1`, `'true'`, `'yes'`.

**UI behavior:** The entire form is replaced with: *"This plugin is fully managed via server configuration. Settings cannot be viewed or changed here."*

---

## Lockdown Levels

The constants compose into three lockdown levels. Use whichever fits your deployment scenario:

| Level | Constants | What Admins See | Can Admin Change Settings? |
|-------|-----------|-----------------|---------------------------|
| **Per-field** | `_SETTINGS` and/or `_LICENSE_KEY` | Overridden fields disabled, credentials hidden, rest normal | Only non-overridden fields |
| **All disabled** | `_DISABLE_SETTINGS` | All fields visible but disabled, credentials hidden | No |
| **Fully hidden** | `_HIDE_SETTINGS` | Only a "managed externally" notice | No (nothing visible) |

`_DISABLE_SETTINGS` and `_HIDE_SETTINGS` work independently of `_SETTINGS` / `_LICENSE_KEY`. You can lock down or hide the settings page without providing any value overrides — the database values are used but cannot be changed via the UI.

---

## Configuration Priority

When the same setting is defined in multiple places, the constant wins:

```
PHP constant (wp-config.php)  >  Database (wp_options)
```

External settings are merged at runtime and **never saved to the database**. The database always reflects what was configured through the admin UI. If you remove a constant from `wp-config.php`, the plugin falls back to the database value on the next page load.

---

## Examples

### Agency deployment — override credentials only

```php
// wp-config.php
define('ONETEAMSOFTWARE_WC_SHIPPO_SHIPPING_LICENSE_KEY', 'agency-license-key');
define('ONETEAMSOFTWARE_WC_SHIPPO_SHIPPING_SETTINGS', [
    'liveApiToken' => 'shippo_live_abc123',
]);
```

The license key and API token are hidden from the admin UI. The shop owner can still configure origin address, services, boxes, and other settings normally.

### Full lockdown — managed hosting

```php
// wp-config.php
define('ONETEAMSOFTWARE_WC_SHIPPO_SHIPPING_LICENSE_KEY', 'managed-license-key');
define('ONETEAMSOFTWARE_WC_SHIPPO_SHIPPING_SETTINGS', [
    'liveApiToken' => 'shippo_live_abc123',
    'sandbox' => 'no',
    'origin' => [
        'country' => 'US',
        'state' => 'CA',
        'city' => 'Los Angeles',
        'postcode' => '90210',
    ],
]);
define('ONETEAMSOFTWARE_WC_SHIPPO_SHIPPING_DISABLE_SETTINGS', true);
```

All settings are visible but locked. Credentials are hidden. Admins can see the configuration but cannot change anything.

### Maximum security — hide everything

```php
// wp-config.php
define('ONETEAMSOFTWARE_WC_SHIPPO_SHIPPING_LICENSE_KEY', 'secret-key');
define('ONETEAMSOFTWARE_WC_SHIPPO_SHIPPING_SETTINGS', [
    'liveApiToken' => 'shippo_live_abc123',
    'sandbox' => 'no',
    'origin' => [
        'country' => 'US',
        'state' => 'CA',
        'city' => 'Los Angeles',
        'postcode' => '90210',
    ],
]);
define('ONETEAMSOFTWARE_WC_SHIPPO_SHIPPING_HIDE_SETTINGS', true);
```

The settings page shows only a notice. No configuration is visible or editable.

### Non-sensitive override — lock the origin address

```php
// wp-config.php
define('ONETEAMSOFTWARE_WC_SHIPPO_SHIPPING_SETTINGS', [
    'origin' => [
        'country' => 'US',
        'state' => 'CA',
        'city' => 'San Francisco',
        'postcode' => '94117',
        'address' => '215 Clayton St',
    ],
]);
```

Only the origin address is locked. The license key, API token, services, boxes, and all other settings remain fully editable through the admin UI.

### Docker / containerized deployment

Mount a custom `wp-config.php` (or an additional config file included by `wp-config.php`) that defines the constants. For example, add an include at the end of `wp-config.php`:

```php
// wp-config.php (inside the container)
if (file_exists(__DIR__ . '/wp-config-shipping.php')) {
    require_once __DIR__ . '/wp-config-shipping.php';
}
```

Then mount `wp-config-shipping.php` via Docker Compose:

```yaml
services:
  wordpress:
    image: wordpress:latest
    volumes:
      - ./wp-config-shipping.php:/var/www/html/wp-config-shipping.php:ro
```

```php
// wp-config-shipping.php
define('ONETEAMSOFTWARE_WC_SHIPPO_SHIPPING_LICENSE_KEY', 'your-license-key');
define('ONETEAMSOFTWARE_WC_SHIPPO_SHIPPING_SETTINGS', [
    'liveApiToken' => 'shippo_live_abc123',
    'sandbox' => 'no',
]);
define('ONETEAMSOFTWARE_WC_SHIPPO_SHIPPING_DISABLE_SETTINGS', true);
```

---

## Save Protection

External settings cannot be overwritten through the admin UI, regardless of how the save is triggered:

- **Standard form save** — external fields are skipped during POST extraction (disabled fields are not submitted by the browser) and stripped from the settings array before database write
- **AJAX tab save** — external fields are stripped via a filter before `update_option()`
- **License activation** — the activation endpoint returns an error when the license key is externally configured

This protection is active at the PHP level. Even if a user manipulates the HTML to remove the `disabled` attribute, the server-side save filters prevent external values from being overwritten.

---

## Settings Reference

This section documents all available setting field IDs, their types, and accepted values. Use these keys in the `_SETTINGS` array.

> **Tip:** You can also find field IDs by inspecting the settings form HTML in your browser (look for `id="woocommerce_{plugin_id}_{fieldKey}"`) or by reading the current settings from the database: `wp option get woocommerce_wc-shippo-shipping_settings --format=json`.

### Carrier API Credentials (per plugin)

These fields vary by carrier. All credential fields use `type: password` and are **automatically hidden** from the admin UI when included in `_SETTINGS`.

#### Shippo

| Field ID | Label | Type |
|----------|-------|------|
| `liveApiToken` | Live API Token | password |
| `testApiToken` | Sandbox / Test API Token | password |
| `aesItn` | AES / ITN Reference | text |
| `sandbox` | Sandbox Mode | checkbox (`yes`/`no`) |

#### EasyPost

| Field ID | Label | Type |
|----------|-------|------|
| `apiKey` | Production API Key | password |
| `testApiKey` | Sandbox / Test API Key | password |
| `merchant_id` | Merchant ID | text |
| `incoterm` | Incoterm | select (`''`, `DDU`, `DDP`) |
| `sandbox` | Sandbox Mode | checkbox (`yes`/`no`) |

#### FedEx

| Field ID | Label | Type |
|----------|-------|------|
| `clientId` | API Key (Production) | password |
| `clientSecret` | Secret Key (Production) | password |
| `accountNumber` | Account Number (Production) | text |
| `testClientId` | API Key (Test) | password |
| `testClientSecret` | Secret Key (Test) | password |
| `testAccountNumber` | Account Number (Test) | text |
| `sandbox` | Sandbox Mode | checkbox (`yes`/`no`) |

#### ShipStation

| Field ID | Label | Type |
|----------|-------|------|
| `apiKey` | API Key | password |
| `apiSecret` | API Secret | password |
| `partner` | Partner Code | text |

#### ShipEngine

| Field ID | Label | Type |
|----------|-------|------|
| `apiKey` | API Key | password |
| `testApiKey` | Sandbox API Key | password |
| `sandbox` | Sandbox Mode | checkbox (`yes`/`no`) |

#### ChitChats

| Field ID | Label | Type |
|----------|-------|------|
| `clientId` | Client ID | password |
| `accessToken` | Access Token | password |
| `testClientId` | Test Client ID | password |
| `testAccessToken` | Test Access Token | password |

#### Stallion Express

| Field ID | Label | Type |
|----------|-------|------|
| `apiKey` | API Key | password |
| `sandboxApiKey` | Sandbox API Key | password |
| `sandbox` | Sandbox Mode | checkbox (`yes`/`no`) |

#### Shipmondo / UPS

These plugins use the shared adapter form. Check your plugin's settings page for the specific field IDs, or inspect the database option.

---

### License (PRO)

| Field ID | Type | Accepted Values |
|----------|------|-----------------|
| `licenseKey` | password | License key string |

Use the dedicated `_LICENSE_KEY` constant instead of including in `_SETTINGS`:
```php
define('ONETEAMSOFTWARE_WC_SHIPPO_SHIPPING_LICENSE_KEY', 'your-key');
```

---

### Ship From (Origin Address)

| Field ID | Type | Accepted Values |
|----------|------|-----------------|
| `origin[name]` | text | Contact name |
| `origin[company]` | text | Company name |
| `origin[email]` | email | Email address |
| `origin[phone]` | text | Phone number |
| `origin[country]` | select | Two-letter country code (`US`, `CA`, `GB`, etc.) |
| `origin[state]` | select/text | State/province code (`CA`, `NY`, `ON`, etc.) |
| `origin[city]` | text | City name |
| `origin[postcode]` | text | Postal/ZIP code |
| `origin[address]` | text | Street address line 1 |
| `origin[address_2]` | text | Street address line 2 |

```php
define('ONETEAMSOFTWARE_WC_SHIPPO_SHIPPING_SETTINGS', [
    'origin' => [
        'name' => 'John Smith',
        'company' => 'Acme Inc',
        'country' => 'US',
        'state' => 'CA',
        'city' => 'San Francisco',
        'postcode' => '94117',
        'address' => '215 Clayton St',
        'phone' => '+1 555 123 4567',
        'email' => 'shipping@acme.com',
    ],
]);
```

---

### Return Address

Only available when the carrier supports return addresses. Same field structure as origin, under the `return_address` key:

| Field ID | Type | Accepted Values |
|----------|------|-----------------|
| `return_address[enabled]` | checkbox | `yes` / `no` |
| `return_address[name]` | text | Contact name |
| `return_address[company]` | text | Company name |
| `return_address[country]` | select | Two-letter country code |
| `return_address[state]` | select/text | State/province code |
| `return_address[city]` | text | City name |
| `return_address[postcode]` | text | Postal/ZIP code |
| `return_address[address]` | text | Street address line 1 |
| `return_address[address_2]` | text | Street address line 2 |

---

### Cart Rates & Shipping Zones

| Field ID | Type | Default | Accepted Values |
|----------|------|---------|-----------------|
| `enableLiveShippingRates` | checkbox | `yes` | `yes` / `no` |
| `shippingZones` | checkbox | `no` | `yes` / `no` |
| `shippingZoneFields` | multiselect | `[]` | Array: `origin`, `boxes`, `carriers`, `services`, `integration`, `signature`, `insurance`, `cod` |
| `requireCountry` | checkbox | | `yes` / `no` |
| `requireCity` | checkbox | | `yes` / `no` |
| `requirePostalCode` | checkbox | | `yes` / `no` |
| `requireAddress` | checkbox | | `yes` / `no` |
| `maxShippingRates` | number | | Max number of rates to display |
| `minRateCost` | number | | Minimum rate cost filter |
| `maxRateCost` | number | | Maximum rate cost filter |

---

### Adjustments

| Field ID | Type | Default | Description |
|----------|------|---------|-------------|
| `weightAdjustment` | number | `0` | Add/subtract weight (in store's weight unit) |
| `weightAdjustmentPercent` | number | `0` | Add/subtract weight by percentage |
| `priceAdjustment` | number | `0` | Add/subtract from shipping rate (in store currency) |
| `priceAdjustmentPercent` | number | `0` | Add/subtract from shipping rate by percentage |

---

### Shipment Defaults

| Field ID | Type | Accepted Values |
|----------|------|-----------------|
| `defaultShipmentDescription` | text | Description for customs |
| `defaultContents` | select | Carrier-specific content types |
| `defaultTariff` | text | HS tariff code |
| `defaultCountryOfOrigin` | select | Two-letter country code |
| `validateAddress` | checkbox | `yes` / `no` |
| `combineBoxes` | checkbox | `yes` / `no` (PRO) |
| `useCubeDimensions` | checkbox | `yes` / `no` (PRO) |
| `insurance` | checkbox | `yes` / `no` |
| `saturday_delivery` | checkbox | `yes` / `no` |
| `hold_for_pickup` | checkbox | `yes` / `no` |
| `cod` | checkbox | `yes` / `no` |
| `cod_method` | select | Carrier-specific COD methods |
| `signature` | checkbox | `yes` / `no` |
| `signature_type` | select | Carrier-specific signature types |
| `signatureCondition` | multiselect | Product category/tag conditions |
| `signatureMinTotalPrice` | number | Minimum order total for signature |
| `requireCompanyName` | checkbox | `yes` / `no` |

---

### Default Dimensions & Weight

| Field ID | Type | Description |
|----------|------|-------------|
| `defaultLength` | number | Default length (in store's dimension unit) |
| `defaultWidth` | number | Default width |
| `defaultHeight` | number | Default height |
| `defaultWeight` | number | Default weight (in store's weight unit) |
| `minWeight` | number | Minimum weight for parcels |

---

### Default Services (PRO)

| Field ID | Type | Accepted Values |
|----------|------|-----------------|
| `defaultDomesticService` | select | Carrier-specific service code |
| `defaultInternationalService` | select | Carrier-specific service code |
| `defaultDomesticReturnService` | select | Carrier-specific service code |
| `defaultInternationalReturnService` | select | Carrier-specific service code |

---

### Delivery & Tracking Display (PRO)

| Field ID | Type | Accepted Values |
|----------|------|-----------------|
| `displayDeliveryTime` | checkbox | `yes` / `no` |
| `addToDeliveryDays` | number | Extra days to add to estimates |
| `displayTrackingType` | checkbox | `yes` / `no` |

---

### Labels (PRO)

| Field ID | Type | Accepted Values |
|----------|------|-----------------|
| `pdfAutoPrint` | checkbox | `yes` / `no` |
| `pdfAutoPrintDialog` | checkbox | `yes` / `no` |
| `allowMultipleShipments` | checkbox | `yes` / `no` |
| `requireToCreateShipment` | checkbox | `yes` / `no` |
| `requireToPurchasePostage` | checkbox | `yes` / `no` |
| `requireToFetchShippingRates` | checkbox | `yes` / `no` |
| `requireToConfirmShipmentDetails` | checkbox | `yes` / `no` |
| `label_format` | select | Carrier-specific label formats |
| `enableBulkShipmentCreation` | checkbox | `yes` / `no` |
| `useServiceSettingsForBulkShipmentCreation` | checkbox | `yes` / `no` |
| `requireToBulkCreateShipments` | checkbox | `yes` / `no` |
| `requireToBulkPurchasePostage` | checkbox | `yes` / `no` |
| `requireToBulkPurchaseNewPostage` | checkbox | `yes` / `no` |

---

### Automation (PRO)

| Field ID | Type | Accepted Values |
|----------|------|-----------------|
| `updateShipmentsEnabled` | checkbox | `yes` / `no` |
| `updateShipmentsCronTaskInterval` | number | Seconds between tracking updates |
| `updateShipmentsFetchInterval` | number | Seconds between fetch cycles |
| `updateShipmentsStatusChangeTimeout` | number | Seconds to wait for status change |
| `updateShipmentsMaxLimit` | number | Max shipments per batch |
| `notifyForStatus` | multiselect | Shipment status codes |
| `completeOrderForStatuses` | multiselect | Shipment status codes |
| `createOrderEnabled` | checkbox | `yes` / `no` |
| `exportOrders` | checkbox | `yes` / `no` |
| `createOrderForStatuses` | multiselect | WooCommerce order statuses |
| `createOrderCronTaskInterval` | number | Seconds between export cycles |
| `createOrderDelayBetweenAttempts` | number | Seconds between retries |
| `createOrderMaxLimit` | number | Max orders per batch |
| `importShipmentsEnabled` | checkbox | `yes` / `no` |
| `importShipmentsCronTaskInterval` | number | Seconds between import cycles |
| `importShipmentsMaxLimit` | number | Max imports per batch |

---

### Multi-Vendor

| Field ID | Type | Accepted Values |
|----------|------|-----------------|
| `useVendorSettings` | checkbox | `yes` / `no` |
| `useSellerAddress` | checkbox | `yes` / `no` (PRO) |

---

### Advanced / General

| Field ID | Type | Accepted Values |
|----------|------|-----------------|
| `debug` | checkbox | `yes` / `no` |
| `displayAdvancedSettings` | checkbox | `yes` / `no` |
| `cache` | checkbox | `yes` / `no` |
| `cacheExpirationInSecs` | number | Cache TTL in seconds |
| `timeout` | number | API request timeout in seconds |
| `store` | select | Store ID (carrier-specific) |
| `warehouse` | select | Warehouse ID (carrier-specific) |

---

### Manufacturer Information

Available when the carrier supports customs declarations.

| Field ID | Type | Accepted Values |
|----------|------|-----------------|
| `defaultManufacturerName` | text | Manufacturer name |
| `defaultManufacturerAddress1` | text | Street address line 1 |
| `defaultManufacturerAddress2` | text | Street address line 2 |
| `defaultManufacturerCity` | text | City |
| `defaultManufacturerProvinceCode` | text | State/province code |
| `defaultManufacturerPostalCode` | text | Postal/ZIP code |
| `defaultManufacturerCountryCode` | select | Two-letter country code |

---

### Conditional Shipping Options

These fields are available when the carrier supports the corresponding feature.

| Field ID | Type | Accepted Values |
|----------|------|-----------------|
| `mediaMail` | select | `exclude`, `only`, `include` |
| `mediaMailCondition` | multiselect | Product category/tag conditions |
| `alcoholCondition` | multiselect | Product category/tag conditions (PRO) |
| `dryIceCondition` | multiselect | Product category/tag conditions (PRO) |

---

### Complex Fields: Boxes and Services

The `boxes` and `services` settings are arrays of objects. They can be overridden via `_SETTINGS` but require the exact structure the plugin expects.

#### Boxes

```php
'boxes' => [
    [
        'enabled' => 'yes',
        'boxName' => 'Small Box',
        'maxItems' => 5,
        'maxweight' => 10,
        'length' => 12,
        'width' => 8,
        'height' => 6,
        'weight' => 0.5,
    ],
    [
        'enabled' => 'yes',
        'boxName' => 'Large Box',
        'maxItems' => 20,
        'maxweight' => 50,
        'length' => 24,
        'width' => 18,
        'height' => 12,
        'weight' => 1.0,
    ],
]
```

#### Services

```php
'services' => [
    'usps_priority' => [
        'enabled' => 'yes',
        'title' => 'Priority Mail',
        'priceAdjustment' => 0,
        'priceAdjustmentPercent' => 0,
    ],
    'usps_first_class' => [
        'enabled' => 'yes',
        'title' => 'First Class',
        'priceAdjustment' => -1.00,
        'priceAdjustmentPercent' => 0,
    ],
]
```

> **Note:** Service codes are carrier-specific. Check your plugin's Services section in the admin UI for the available codes, or read them from the database option.

---

### Checkbox Values

All checkbox fields store `'yes'` (checked) or `'no'` (unchecked) as strings — not boolean `true`/`false`:

```php
// Correct
'sandbox' => 'no',
'insurance' => 'yes',

// Wrong — will not work as expected
'sandbox' => false,
'insurance' => true,
```

---

## Shipping Zones

External settings apply to global (instance 0) settings. When shipping zones are enabled, external values also override zone-level instance settings — external always wins. Zone instance forms show the same disabled/hidden fields as the global settings page.

---

## Troubleshooting

### `_SETTINGS` not taking effect

If the `_SETTINGS` array has a PHP syntax error (missing comma, unclosed bracket, etc.), PHP will fail to load `wp-config.php` entirely — you'll see a white screen or a PHP fatal error. Check your PHP error log and fix the syntax.

If the constant is defined but the plugin ignores it, make sure the value is a PHP array — not a string or other type.

### Constants not taking effect

1. Make sure the `define()` is **before** the line `/* That's all, stop editing! */` in `wp-config.php`
2. Clear any persistent object cache (Redis, Memcached) after making changes
3. Verify the constant name matches exactly — the prefix is case-sensitive

### Finding the right field ID

The easiest way to find a field ID is to inspect the admin settings form in your browser:

1. Open the plugin settings page in WooCommerce
2. Right-click on the field you want to override and select "Inspect"
3. The field's `id` attribute follows the pattern `woocommerce_{plugin_id}_{fieldKey}`
4. The `{fieldKey}` part is what you use in `_SETTINGS`

Alternatively, read the current settings from the database:

```bash
wp option get woocommerce_wc-shippo-shipping_settings --format=json | python3 -m json.tool
```

---

## Related Documentation

- [Hooks API Reference](hooks-api.md) — 22 public hooks for customizing shipping behavior
- [Plugin Hook Prefixes](hooks-by-plugin.md) — Hook prefix mapping for all supported plugins

---

## Related Plugins

Server-level configuration via `wp-config.php` is a **PRO-only feature** available in all 1TeamSoftware WooCommerce shipping plugins:

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

*Server-level configuration via `wp-config.php` is a PRO-only feature available across all 1TeamSoftware WooCommerce shipping plugins. [Upgrade to PRO](https://1teamsoftware.com/product-category/woocommerce-plugins/) to manage credentials, lock settings, and deploy at scale.*

[🏠 Home](../README.md) | [◀ Plugin Hook Prefixes](hooks-by-plugin.md)
