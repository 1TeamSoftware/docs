# E-Commerce & Secure Downloads

If you sell digital products via **WooCommerce** or **Easy Digital Downloads (EDD)**, offloading to the cloud creates a problem: anyone with the direct link to your file can share it, bypassing your checkout entirely.

**WP CloudSync Master PRO** solves this with **Secure Signed URLs**.

## How Secure Signed URLs Work

The solution to protecting digital goods stored in the cloud is "Signed URLs." 

A Signed URL is a unique, mathematically generated link to your cloud file that contains a built-in expiration timer. Instead of handing the customer a permanent, public link to your cloud storage bucket, CloudSync Master acts as a secure middleman.

Here is the exact flow of how this protects your store:
1. **Customer Buys a Product:** A user purchases a digital download on your WooCommerce store.
2. **User Clicks Download:** They go to their "My Account" page and click the download button provided by WooCommerce.
3. **Dynamic Generation:** Native WooCommerce securely intercepts the request. Instead of serving a local file, WP CloudSync Master hooks into the process and instantly contacts your cloud provider (e.g., Amazon S3, Google Cloud, Cloudflare R2).
4. **The Hand-off:** WP CloudSync Master generates a temporary, cryptographically signed URL valid *only* for a specific window of time.
5. **Secure Delivery:** The user's browser is transparently redirected to this temporary cloud link to download the file directly from the high-speed cloud edge.

Once that URL expires (e.g., 60 minutes after being generated), the link breaks. Even if the customer posts the link on a forum or shares it with friends, attempting to use the expired link results in a `403 Forbidden` error from the cloud provider. 

## Setting Up Secure Downloads

1. Ensure your WooCommerce products are set up correctly as **Virtual** and **Downloadable** files. 
2. Set the WooCommerce download method to **Redirect only** (found in WooCommerce > Settings > Products > Downloadable products). This is highly recommended so the bandwidth load is handled entirely by your cloud provider, rather than your local web server.
3. In **WP CloudSync Master**, go to the **Accounts** tab and ensure your provider is configured.
4. Go to **CloudSync Master > Integrations**.

![Integrations Dashboard](../assets/images/integrations-tab.png)

5. Toggle **ON** the integration for WooCommerce (or Easy Digital Downloads).
6. Set your preferred **Signed URL Expiration Time**. The default is usually 1 hour (3600 seconds), giving users plenty of time to download their purchase, but short enough to prevent long-term piracy.

### How the Plugin Handles Private Files

You **do not** need to make your main media bucket private (doing so would break your public blog images!). WP CloudSync Master handles file security intelligently based on your cloud provider's capabilities:

*   **Providers with Per-File Permissions (Amazon S3, Google Cloud):** 
    The plugin detects when you upload a file via WooCommerce. It automatically applies a strict "Private" Object Access Control List (ACL) to that specific file during upload, fully protecting it even if the rest of your bucket is public.
*   **Providers without Per-File Permissions (Cloudflare R2):**
    For providers like Cloudflare R2 that only support bucket-level permissions, CloudSync Master provides a dedicated **"Private Bucket"** setting under the *Accounts* tab. Create a separate, explicitly private bucket in Cloudflare, and enter its name in the plugin settings. The plugin will automatically route sensitive WooCommerce uploads exclusively to this private bucket.

> [!TIP]
> You can easily verify if a file is protected by going to the WordPress Media Library (List View). The "Access Type" column will clearly show whether a file is `public` or `private`.

---
*[CloudSync Master PRO](https://1teamsoftware.com/product/wp-cloudsync-master-pro/) includes WooCommerce and EDD signed URL support.*

[🏠 Home](../README.md) | [◀ Previous](../04-features-and-tools/cross-provider-migration.md) | [Next ▶](image-optimizers.md)