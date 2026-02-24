# Common Issues & Troubleshooting

Here are the most common connection and syncing issues and how to fix them.

## "Connection Failed: Access Denied"

If you see an "Access Denied" or "Invalid Credentials" error when trying to save a new cloud account:

*   **Amazon S3:** Ensure your IAM user has the correct permissions (specifically `s3:PutObject` and `s3:GetObject`). Double-check that your Access Key and Secret Key were copied without any leading or trailing spaces.
*   **Cloudflare R2:** Verify that the API Token you generated has **Object Read & Write** permissions for the specific bucket you are trying to connect. A common mistake is generating a token with "Read-only" permissions.
*   **Google Cloud:** If using a JSON key, ensure you copied the entire contents of the `.json` file, including the opening and closing curly braces `{ }`.

## The Upload Queue is Stuck

If you initiated a massive bulk upload via the Media Scanner and the queue seems to have frozen:

1. Go to **CloudSync Master > Queue**.
2. If the **Pending Uploads** count is stuck and not decreasing when you click **Process Upload Queue**:
3. **Reason:** Often, cheap shared hosting accounts aggressively kill long-running PHP processes. The plugin attempts to handle this gracefully via async AJAX calls, but a stringent server configuration might force a timeout.
4. **Fix:** Lower your **Upload Concurrency** slider in the **Settings** tab. If it's set to 20, drop it to 5 or even 1. Then, return to the Queue tab and click **Process Upload Queue** again to resume.

---
*[CloudSync Master PRO](https://1teamsoftware.com/product/wp-cloudsync-master-pro/) includes priority support for connection and sync issues.*

[🏠 Home](../README.md) | [◀ Previous](../05-integrations/image-optimizers.md) | [Next ▶](faq.md)