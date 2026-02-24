# WP-CLI Commands

WP CloudSync Master Pro includes WP-CLI support for managing accounts, settings, object links, and the sync queue from the command line.

## AI Agent Integration (Skills)

WP CloudSync Master provides a "skill" for AI Agents (like Cursor, GitHub Copilot, etc.) to interact with its CLI commands.

You can install the official WP CloudSync Master skill directly using `npx` from the [1TeamSoftware Skills Repository](https://github.com/1TeamSoftware/skills):

```bash
npx skills add https://github.com/1TeamSoftware/skills --skill 1teamsoftware/wp-cloudsync-master
```

Or you can install it using [skills.sh](https://skills.sh):

```bash
skills add 1teamsoftware/wp-cloudsync-master
```

This will provide your AI assistant with the necessary context and commands to automate syncs, manage accounts, and script complex migration tasks based on your prompts.

## Base Commands

### Check Sync Status
Shows the overall sync status across all configured accounts, including queued items.

```bash
wp cloudsync status
wp cloudsync status --format=json
```

### Trigger Synchronization
Triggers a scan of the media library to find missing files and queue them for background offloading.

```bash
wp cloudsync sync                    # Sync all active accounts
wp cloudsync sync --account=<id>     # Sync a specific account by ID
wp cloudsync sync --batch-size=100   # Set items to process per batch
wp cloudsync sync --account=1 --batch-size=500  # Combine arguments
```

## Queue Operations (PRO Only)

Manage the background upload queue to oversee large migrations.

```bash
# View current background queue status
wp cloudsync queue status

# List items currently pending in the queue
wp cloudsync queue list

# Process pending queue items immediately
wp cloudsync queue process

# Clear all pending items from the queue
wp cloudsync queue clear
```

## Account Management

Manage cloud storage provider accounts directly from code pipelines or scripts.

```bash
# List all configured accounts
wp cloudsync accounts list

# Add a newly provisioned S3 compatible account
wp cloudsync accounts add --provider=s3 --bucket=my-bucket --access-key-id=KEY --secret-access-key=SECRET

# Add a Cloudflare R2 account
wp cloudsync accounts add --provider=cloudflare-r2 --account-id=xxx --access-key-id=xxx --secret-access-key=xxx

# Update an existing account settings
wp cloudsync accounts update --account=<id> --name="Production Bucket" --region="us-east-1"

# Set an existing account as the active target
wp cloudsync accounts activate --account=<id>

# Run a connection validation test for an account
wp cloudsync accounts test --account=<id>

# Show detailed information and statistics for a specific account
wp cloudsync accounts info --account=<id>

# Delete an account (and optionally remove its tracked local object link records)
wp cloudsync accounts delete --account=<id>
wp cloudsync accounts delete --account=<id> --delete-objects
```

## Object Management

Manage individual file object link records mapped in the local WordPress database.

```bash
# List synced object link records for the active account
wp cloudsync objects list [--search=<term>] [--page=1] [--per-page=20]

# List synced object link records for a specific account
wp cloudsync objects list --account=<id>

# Add a specific single file path back to the upload queue to re-upload it
wp cloudsync objects reupload --file-path=wp-content/uploads/2023/image.jpg

# Request a re-upload of all recorded objects for a given account
wp cloudsync objects reupload --account=<id> --all

# Wipe out the tracking record of a specific local file
wp cloudsync objects delete --file-path=wp-content/uploads/2023/image.jpg

# Wipe out the tracking record from a specific account
wp cloudsync objects delete --file-path=wp-content/uploads/2023/image.jpg --account=<id>

# Wipe out all object tracking records associated with an account
wp cloudsync objects delete --account=<id> --all --yes
```

## Settings & Configuration

View and modify base plugin settings as well as hardcoded external configurations.

```bash
# List all settings and their currently evaluated values
wp cloudsync settings list

# Inspect a specific setting's value
wp cloudsync settings get uploadBatchSize

# Update a setting's value programmatically
wp cloudsync settings set uploadConcurrency 5
wp cloudsync settings set deleteFromLocalServer yes

# Reset a setting to its default out-of-the-box value
wp cloudsync settings reset uploadBatchSize

# Debug mode: View the source of all configurations (Database vs wp-config.php)
wp cloudsync config show

# Hardcode security: Generate a PHP snippet format of the active account data for wp-config.php insertion
wp cloudsync config generate [--account=<id>]
```

## Integrations Management

Enable or configure third-party plugin integrations via CLI.

```bash
# Display all available integrations and their current status
wp cloudsync integrations list

# Enable a specific integration
wp cloudsync integrations enable woocommerce

# Disable a specific integration
wp cloudsync integrations disable woocommerce

# Fine-tune a specific integration option setting
wp cloudsync integrations set woocommerce enable_signed_urls yes
```

## License Management

Automate the activation of your PRO license key for deployment scripts.

```bash
# Show current license activation status
wp cloudsync license status

# Register and activate a new license key on the installation
wp cloudsync license activate <YOUR-LICENSE-KEY>

# Clear and deactivate the current license key
wp cloudsync license deactivate
```

## Competitor Plugin Migration

Import setups smoothly from legacy competitor offload plugins.

```bash
# Scan system for installed and supported competitor offload plugins
wp cloudsync migrate detect

# Import settings and create a new managed account pulled straight from a competitor plugin
wp cloudsync migrate import --from=offload-media
```
