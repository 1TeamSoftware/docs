# Action and Filter Hooks

WP CloudSync Master provides actions and filters for modifying its behavior, integrating with other plugins, and customizing the sync process.

All hooks are prefixed with `wp-cloudsync-master_` to avoid conflicts with other plugins.

## Filter Hooks

### Storage & URLs

*   `wp-cloudsync-master_cloud_url`
    *   **Arguments**: `string $cloudUrl`, `string $localUrl`, `string $filePath`
    *   **Description**: Filters the generated cloud URL before it's returned to the requesting process. Crucial for custom Signed URL implementations or custom domains.
    *   **Example**:
        ```php
        add_filter('wp-cloudsync-master_cloud_url', function(string $cloudUrl, string $localUrl, string $filePath): string {
            // Replace the cloud URL with a custom proxy URL
            return str_replace('storage.googleapis.com/my-bucket', 'cdn.my-domain.com', $cloudUrl);
        }, 10, 3);
        ```

*   `wp-cloudsync-master_upload_acl`
    *   **Arguments**: `string $aclType`, `string $filePath`
    *   **Description**: Filters the ACL type (e.g., 'public-read', 'private') applied to an object during upload.
    *   **Example**:
        ```php
        add_filter('wp-cloudsync-master_upload_acl', function(string $aclType, string $filePath): string {
            // Force PDF files to be private
            if (pathinfo($filePath, PATHINFO_EXTENSION) === 'pdf') {
                return 'private';
            }
            return $aclType;
        }, 10, 2);
        ```

*   `wp-cloudsync-master_upload_bucket`
    *   **Arguments**: `string $bucketName`, `string $filePath`, `string $aclType`
    *   **Description**: Filters the target bucket name for file uploads. Allows routing files to different buckets based on ACL or path.

### URL Transformation

*   `wp-cloudsync-master_should_rewrite_url`
    *   **Arguments**: `bool $shouldRewrite`, `string $url`
    *   **Description**: Control whether a specific URL should be intercepted and rewritten by the plugin.
    *   **Example**:
        ```php
        add_filter('wp-cloudsync-master_should_rewrite_url', function(bool $shouldRewrite, string $url): bool {
            // Do not rewrite URLs for a specific domain or path
            if (strpos($url, 'ignore-this-folder') !== false) {
                return false;
            }
            return $shouldRewrite;
        }, 10, 2);
        ```

*   `wp-cloudsync-master_before_url_replace`
    *   **Arguments**: `string $content`
    *   **Description**: Filters the full HTML content before the URL replacement regex engine runs.

*   `wp-cloudsync-master_after_url_replace`
    *   **Arguments**: `string $content`, `string $originalContent`
    *   **Description**: Filters the full HTML content after the URL replacement regex engine has run.

### Developer API & Configuration

*   `wp-cloudsync-master_storage_config`
    *   **Arguments**: `array $config`, `string $providerName`
    *   **Description**: Filters the storage manager configuration array before the cloud provider SDK connection is initialized.

*   `wp-cloudsync-master_object_data`
    *   **Arguments**: `array $objectData`, `string $filePath`
    *   **Description**: Modify the object data mapping payload before it is saved to the local Object Link Repository.

## Action Hooks

### File Operations

*   `wp-cloudsync-master_before_upload`
    *   **Arguments**: `string $filePath`, `string $objectName`, `string $bucketName`
    *   **Description**: Fires immediately before a file is uploaded to cloud storage.

*   `wp-cloudsync-master_after_upload`
    *   **Arguments**: `string $filePath`, `string $objectName`, `string $bucketName`, `string $publicUrl`
    *   **Description**: Fires immediately after a file has been successfully uploaded to cloud storage.
    *   **Example**:
        ```php
        add_action('wp-cloudsync-master_after_upload', function(string $filePath, string $objectName, string $bucketName, string $publicUrl): void {
            // Log to an external service or trigger a custom workflow
            error_log("Successfully offloaded: " . $filePath . " to " . $bucketName);
        }, 10, 4);
        ```

*   `wp-cloudsync-master_upload_failed`
    *   **Arguments**: `string $filePath`, `string $objectName`, `string $bucketName`, `string $errorMessage`
    *   **Description**: Fires when a background or synchronous file upload fails.

*   `wp-cloudsync-master_before_delete_object`
    *   **Arguments**: `string $objectName`, `string $bucketName`
    *   **Description**: Fires immediately before an object is targeted for deletion from cloud storage.

*   `wp-cloudsync-master_after_delete_object`
    *   **Arguments**: `string $objectName`, `string $bucketName`
    *   **Description**: Fires immediately after a file has been successfully deleted from cloud storage.

*   `wp-cloudsync-master_local_file_deleted`
    *   **Arguments**: `string $filePath`
    *   **Description**: Fires when a local file is deleted by the plugin (e.g., after a successful offload).

### Object Link Repository

*   `wp-cloudsync-master_before_object_link_update`
    *   **Arguments**: `string $filePath`, `array $data`
    *   **Description**: Fires before updating an object link tracking record in the repository.

*   `wp-cloudsync-master_after_object_link_update`
    *   **Arguments**: `string $filePath`, `array $data`
    *   **Description**: Fires after an object link tracking record is updated in the repository.

*   `wp-cloudsync-master_object_link_deleted`
    *   **Arguments**: `string $filePath`
    *   **Description**: Fires when an object link tracking record is deleted from the repository.

### Extra File Sources (PRO)

These hooks allow integrations to provide additional non-attachment files for the Fill Upload Queue process. Used by the BuddyBoss/BuddyPress integration to discover avatars and cover images.

*   `csm_pro_extra_file_sources`
    *   **Type**: Filter
    *   **Arguments**: `array $sources`
    *   **Description**: Register additional file sources for the Fill Upload Queue. Each source is a callable that receives pagination parameters and returns a batch of file paths. The Fill Upload Queue task iterates through all registered sources after processing standard WordPress attachments.
    *   **Example**:
        ```php
        add_filter('csm_pro_extra_file_sources', function(array $sources): array {
            $sources['my_plugin'] = function(int $offset, int $batchSize): array {
                // Discover files in batches using directory-level pagination
                $directories = glob('/path/to/custom/uploads/[0-9]*', GLOB_ONLYDIR);
                sort($directories);
                $batch = array_slice($directories, $offset, $batchSize);

                $files = [];
                foreach ($batch as $dir) {
                    foreach (glob($dir . '/*.{jpg,png,gif,webp}', GLOB_BRACE) as $file) {
                        $files[] = $file;
                    }
                }

                return [
                    'files' => $files,     // Absolute file paths
                    'total' => count($directories),  // Total directories (for pagination)
                ];
            };
            return $sources;
        });
        ```
    *   **Notes**:
        - The `$offset` and `$batchSize` parameters refer to **directories**, not individual files. This enables efficient resume for large sites.
        - Return absolute file paths in the `files` array.
        - The `total` count should reflect the total number of directories, not files.
        - Once all directories are processed, the source is marked complete and won't re-scan until invalidated.

*   `csm_pro_before_extra_file_sources`
    *   **Type**: Action
    *   **Arguments**: `string $cronTaskId`
    *   **Description**: Fires before the Fill Upload Queue checks extra file source completion status. Integrations use this to consume invalidation flags and reset per-source progress, forcing a re-scan when new files have been added.
    *   **Example**:
        ```php
        add_action('csm_pro_before_extra_file_sources', function(string $cronTaskId): void {
            if (!get_option('my_plugin_scan_invalidated', false)) {
                return;
            }
            // Reset progress so the next Fill Queue cycle re-scans
            delete_option($cronTaskId . '_extra_my_plugin_complete');
            delete_option($cronTaskId . '_extra_my_plugin_offset');
            delete_option('my_plugin_scan_invalidated');
        });
        ```

### WP-CLI Events

*   `wp-cloudsync-master_cli_before_sync`
    *   **Arguments**: `array $accounts`, `int $batchSize`
    *   **Description**: Fires before the `wp cloudsync sync` command starts scanning the media library. `$accounts` is an array of provider account objects.

*   `wp-cloudsync-master_cli_after_sync`
    *   **Arguments**: `array $accounts`, `int $totalQueued`
    *   **Description**: Fires after the `wp cloudsync sync` command completes scanning the media library. `$accounts` is an array of provider account objects.
