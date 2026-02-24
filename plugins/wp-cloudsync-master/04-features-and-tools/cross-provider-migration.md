# Cross-Provider Migration

Switching cloud providers normally means downloading gigabytes of data to your local server, then re-uploading everything to the new provider.

**CloudSync Master PRO's Cross-Provider Migration skips that.** It streams files directly between cloud providers without touching your local disk.

## How to Stream Files Between Clouds

The Cross-Provider Migration tool allows you to stream files directly from your old cloud provider to your new one—without ever hitting your local hard drive.

### The Migration Process

1. Go to **CloudSync Master > Accounts** and add your *new* cloud provider (e.g., Cloudflare R2).
2. Go back to **Accounts** and ensure your *old* provider (e.g., Amazon S3) is also connected.
3. Click on the **Tools** tab.
4. Locate the **Cross-Provider Migration** section.
5. Set the **Source** account (Amazon S3).
6. Set the **Destination** account (Cloudflare R2).
7. Start the migration.

### What Happens Under the Hood?

*   The plugin creates tasks in the [Background Sync Queue](../03-core-configuration/background-sync.md).
*   It grabs a temporary copy of the file from the Source right into memory on your server.
*   It immediately streams that file into the Destination bucket.
*   It automatically updates all the URLs in your WordPress database to point to the new destination.
*   It deletes the file from the Source bucket (optional but recommended).

This bypasses your web host's disk entirely, which saves time and avoids storage overflow.

---
*[CloudSync Master PRO](https://1teamsoftware.com/product/wp-cloudsync-master-pro/) includes cross-provider migration and competitor plugin import.*

[🏠 Home](../README.md) | [◀ Previous](offloading-media.md) | [Next ▶](../05-integrations/ecommerce.md)