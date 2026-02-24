# Frequently Asked Questions

### Which cloud storage provider should I choose?

It really depends on your budget and needs:
*   **Cloudflare R2:** Best for high-traffic sites because it charges **zero egress fees** (meaning no bandwidth costs).
*   **Wasabi:** Also great for predictable pricing with no egress fees.
*   **Amazon S3:** The absolute industry standard. Best for enterprise reliability and the largest ecosystem.
*   **Google Cloud:** Best if your organization is already heavily invested in Google Workspace or GCP. 
*   **DigitalOcean Spaces:** A very simple, affordable starting point.

### Will CloudSync Master slow down my WordPress admin?

No. Offloading happens via the Background Sync Queue, which processes files incrementally in the background. Your site stays responsive during the process.

### What happens to my existing media files?

When you install the plugin, your existing files stay right where they are on your server. 
Using the **Media Scanner (PRO)**, the plugin will copy them to the cloud. Only if you enable the "Delete from Local Server" setting will the plugin remove the local copies *after* verifying a successful cloud backup.

### Can I switch cloud providers later?

Yes, easily. Your media library stays intact. You can add multiple provider accounts and use the [Cross-Provider Migration](../04-features-and-tools/cross-provider-migration.md) tool to stream files from your old provider directly into your new one asynchronously.

### Does this work with page builders like Elementor?

Yes. The Free version handles standard WordPress content. **CloudSync Master PRO** adds specific integrations for **Elementor** and **Advanced Custom Fields (ACF)** that rewrite background images and custom field URLs to serve from your cloud bucket.

---
*[CloudSync Master PRO](https://1teamsoftware.com/product/wp-cloudsync-master-pro/) includes unlimited offloading, 10 providers, and no per-item limits.*

[🏠 Home](../README.md) | [◀ Previous](common-issues.md)