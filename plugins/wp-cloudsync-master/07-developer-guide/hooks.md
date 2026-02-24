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

### WP-CLI Events

*   `wp-cloudsync-master_cli_before_sync`
    *   **Arguments**: `\OneTeamSoftware\WP\CloudSyncMaster\Provider\Account\ProviderAccountInterface[] $accounts`, `int $batchSize`
    *   **Description**: Fires before the `wp cloudsync sync` command starts scanning the media library.

*   `wp-cloudsync-master_cli_after_sync`
    *   **Arguments**: `\OneTeamSoftware\WP\CloudSyncMaster\Provider\Account\ProviderAccountInterface[] $accounts`, `int $totalQueued`
    *   **Description**: Fires after the `wp cloudsync sync` command completes scanning the media library.
