# WP CloudSync Master Documentation

**Stop letting media files slow down your WordPress site.** Large media libraries consume expensive server space, slow down your page load times, and hurt your SEO rankings.

**WP CloudSync Master** automatically offloads your WordPress media to cloud storage (Amazon S3, Cloudflare R2, Google Cloud, and more), serving your images and videos through a CDN.

Lower your hosting costs, improve Core Web Vitals, and stop worrying about disk space.

## Why Choose CloudSync Master?

* **Zero egress fees with Cloudflare R2:** Our R2 integration means you pay nothing for bandwidth when visitors load your images.
* **Free bulk migration:** The free version includes tools to migrate your existing media library to the cloud, not just new uploads.
* **Built for WooCommerce:** Offload product images and protect digital downloads with time-limited Signed URLs.
* **One-click setup:** Connect to 10+ cloud providers in seconds without touching `wp-config.php`.
## Free vs. PRO

The Free version gives you everything you need to offload new media to any of our 10 supported cloud providers, including background sync and URL rewriting.

**CloudSync Master PRO** adds tools for businesses and power users:

*   **The Media Scanner:** Bulk-upload your existing media library to the cloud with a single click.
*   **One-Click Cloud Setup:** Skip the confusing access keys. Connect to Cloudflare R2 and Google Cloud instantly using secure OAuth.
*   **High-Speed Concurrency:** Up to 20 parallel background uploads, which cuts bulk migration times significantly.
*   **Secure Digital Downloads:** Protect your WooCommerce or Easy Digital Downloads (EDD) products with time-limited Signed URLs.
*   **Static Asset Offloading:** Speed up your entire site by moving your theme's CSS and JS files to the cloud.
*   **Integrations:** Works with image optimizers (ShortPixel, Imagify), cache plugins (WP Rocket), and page builders (Elementor).

## How to Use These Docs

We've organized the guides to follow your natural workflow:

1. **[Getting Started](01-getting-started/overview.md):** Learn what the plugin does and run through our simple setup wizard.
2. **[Cloud Providers](02-cloud-providers/amazon-s3.md):** Detailed connection guides for our 10 supported cloud platforms (Amazon S3, Cloudflare R2, Google Cloud, and more).
3. **[Core Configuration](03-core-configuration/general-settings.md):** Tweak your sync logic, upload concurrency, and manage your cloud files directly from WordPress.
4. **[Features & Tools](04-features-and-tools/offloading-media.md):** Discover how to offload existing media, migrate between cloud providers, or host your CSS/JS assets in the cloud.
5. **[Integrations](05-integrations/ecommerce.md):** Connect CloudSync Master with WooCommerce, image optimizers, cache plugins, and page builders.
6. **[Troubleshooting](06-troubleshooting/common-issues.md):** Fast solutions to common questions and connection issues.
7. **[Developer Guide](07-developer-guide/README.md):** Reference documentation for WP-CLI commands and action/filter hooks.

---

**Need Help?**
If you run into any trouble that isn't covered in these docs, reach out to our support team at [1TeamSoftware Support](https://1teamsoftware.com/contact-us/).
