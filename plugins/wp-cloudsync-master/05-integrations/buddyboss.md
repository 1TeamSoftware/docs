# BuddyBoss / BuddyPress Integration

WP CloudSync Master PRO integrates with **BuddyBoss Platform** and **BuddyPress** to offload community media — activity feed photos, user avatars, group avatars, and cover images — to your cloud storage provider.

This eliminates expensive local storage and bandwidth, serving all community media from your cloud CDN (e.g., Cloudflare R2, Amazon S3, Google Cloud Storage).

## What Gets Offloaded

| Media Type | How It's Offloaded |
|------------|-------------------|
| **Activity feed photos** | Automatically on upload or via Fill Upload Queue |
| **Activity feed videos** | Automatically on upload or via Fill Upload Queue |
| **Activity feed documents** | Automatically on upload or via Fill Upload Queue |
| **User avatars** | Immediately when cropped + background scan for existing files |
| **Group avatars** | Immediately when cropped + background scan for existing files |
| **User cover images** | Immediately on upload + background scan for existing files |
| **Group cover images** | Immediately on upload + background scan for existing files |

> **Note:** Activity feed media (photos, videos, documents) is a BuddyBoss-only feature. Standard BuddyPress does not include these — only avatars and cover images are applicable for BuddyPress sites.

## How It Works

### Activity Feed Media (BuddyBoss only)

BuddyBoss stores activity media as standard WordPress attachments. CloudSync Master handles these exactly like any other Media Library file:

1. **On upload:** If **"Offload on Upload"** is enabled (in **CloudSync Master → Settings**), the file is uploaded to cloud storage immediately.
2. **Via Fill Queue:** If "Offload on Upload" is disabled (or the file was uploaded before CloudSync was installed), clicking **Fill Upload Queue** discovers and offloads these files.

BuddyBoss normally serves activity media through a symlink system for privacy. CloudSync automatically bypasses this and serves files directly from your cloud CDN.

**Before CloudSync:**
```
https://yoursite.com/wp-content/uploads/bb-platform-previews/b9014e77.../9b5b2b18...jpg
```

**After CloudSync:**
```
https://media.yoursite.com/wp-content/uploads/buddypress/bb_medias/photo.jpg
```

### Avatars & Cover Images

Avatars and cover images are not standard WordPress attachments — they're stored as plain files without Media Library records. CloudSync Master handles these automatically:

1. **Immediate upload:** When a user uploads or crops an avatar (or uploads a cover image), the file is uploaded to cloud storage right away — the cloud URL is available on the very next page load.
2. **Background scan:** For pre-existing files (uploaded before CloudSync was installed), **Fill Upload Queue** discovers avatar and cover image files and queues them for offload.
3. **Cloud fallback:** When local files are removed after offloading (e.g., if you enable "Delete Local File After Upload"), CloudSync serves the cloud URL automatically. Users see no difference — avatars and covers load seamlessly from the cloud.

## Setup

### Prerequisites

- WP CloudSync Master PRO active with a configured cloud storage account
- BuddyBoss Platform **or** BuddyPress plugin active

### Enabling the Integration

The integration **auto-enables** when BuddyBoss Platform or BuddyPress is detected. No manual configuration needed.

To verify or toggle the integration:

1. Go to **CloudSync Master → Integrations** tab
2. Scroll to the **Community** section
3. The **BuddyBoss / BuddyPress** card shows which plugin is detected and a toggle switch

### Offloading Existing Media

If you're installing CloudSync Master on a site that already has BuddyBoss or BuddyPress content:

1. Go to **CloudSync Master → [Your Provider] tab** (e.g., "Cloudflare R2")
2. Click **Fill Upload Queue**
3. Click **Continue** to confirm

The fill process automatically discovers all community media including activity feed files, avatars, and cover images.

Once discovered, click **Process Upload Queue** to upload all files to cloud storage.

### Verifying Cloud URLs

After offloading, verify that community media is served from your cloud CDN:

1. Visit a user's profile page
2. Right-click their avatar → **Inspect Element**
3. The `<img>` `src` attribute should show your cloud domain (e.g., `https://media.yoursite.com/...`)

## Compatibility

- **BuddyBoss Platform** — fully supported
- **BuddyPress** — fully supported (avatars, cover images, background scan, immediate upload)
- Works with all CloudSync Master supported providers: Amazon S3, Google Cloud Storage, Cloudflare R2, DigitalOcean Spaces, Backblaze B2, Wasabi, Vultr, Linode
- Compatible with BuddyBoss's privacy settings — files are offloaded regardless of privacy level. If you need privacy-aware access control, use a private bucket with signed URLs.

## Troubleshooting

### Avatars still showing local URLs after offloading

1. Check that the integration is enabled in **CloudSync Master → Integrations**
2. Verify files were offloaded: Go to **CloudSync Master → Objects** and search for "avatars"
3. If "Offload on Upload" is disabled, run **Fill Upload Queue** to discover existing avatar files

### Activity media not offloading (BuddyBoss only)

Activity feed photos, videos, and documents are standard WordPress attachments. If they're not offloading:

1. Check that the attachment exists in **Media → Library**
2. Run **Fill Upload Queue** and wait for processing to complete

### Fill Upload Queue appears stuck

1. This is normal for large sites — the process runs in background batches via WP-Cron
2. Check progress by refreshing the queue page
3. If truly stuck, check your server's WP-Cron configuration — some hosts require a real cron job instead of WordPress's built-in cron

---
*[CloudSync Master PRO](https://1teamsoftware.com/product/wp-cloudsync-master-pro/) includes the BuddyBoss/BuddyPress integration along with support for Elementor, ACF, WooCommerce, EDD, WPML, and more.*

[🏠 Home](../README.md) | [⬅ Integrations](README.md)
