# Offloading Your Existing Media Library

If you've just installed WP CloudSync Master on an established website, your current images won't automatically jump to the cloud. You need to initiate a bulk upload. 

That's where the **Media Scanner** comes in.

## Using the Media Scanner (PRO)

The Media Scanner crawls your entire WordPress Media Library and pushes every un-synced file to your active cloud bucket.

1. Navigate to **CloudSync Master**.
2. Assuming you've already connected an account (like S3 or R2), go to the **Tools** tab.
3. Locate the **Scanner** tool.
4. Click **Start Scan and Upload**.

### The Process

The plugin will begin indexing your media. It will push items into the [Background Sync Queue](../03-core-configuration/background-sync.md). You can navigate to the **Queue** tab at any time to watch the progress in real-time.

> [!IMPORTANT]
> The time it takes to bulk-upload your library depends entirely on the size of your library, the size of the files, your server's upload speed, and the **Concurrency** setting under General Settings. 

### Freeing Up Local Space

If your goal in moving to the cloud is to save disk space on your web host:

1. Go to **Settings**.
2. Ensure **Delete from Local Server** is enabled.
3. Once the Media Scanner completes and the Queue shows 100% success, the plugin will have deleted the local copies from `wp-content/uploads`, freeing up disk space.

### Dealing with Failed Uploads

If items fail during a bulk scan (often caused by temporary server timeouts on shared hosting), the plugin automatically retries failed items.

If you notice the queue has completely stalled or items are repeatedly failing without success:

1. Go to the **Settings** tab.
2. Lower your **Upload Concurrency** slider (e.g., from 20 down to 5 or 1) to reduce server load.
3. Return to the **Queue** tab and click **Process Upload Queue** to resume picking up right where it left off.

---
*[CloudSync Master PRO](https://1teamsoftware.com/product/wp-cloudsync-master-pro/) includes the Media Scanner for bulk-uploading your existing library.*

[🏠 Home](../README.md) | [◀ Previous](../03-core-configuration/objects-browser.md) | [Next ▶](cross-provider-migration.md)