---
icon: lucide/receipt-text
---

# Logging

## General logging

php-deploykit can log deployment output to a file specified by `LOG_FILE` in `.env`. If `LOG="true"`, the script stores output in the log file; if `LOG="false"`, it does not. Ensure you have permissions to write to the specified file; if the file does not exist, the script will try to create it and may fail if you lack permissions for the parent directory.

Every time you run a command via run.sh, it creates a new log entry with the date and time. You can view logs for a specific deployment by running `php-deploykit --logs` or selecting option 6 from the menu.

## Webhook logging

This is a much less common use case, but php-deploykit can also log incoming webhook requests if `LOG_WEBHOOK="true"` in `.env`. If enabled, logs are stored in the file specified by `WEBHOOK_LOG_FILE`. This is meant for debugging; unless you are debugging, it is recommended to set `LOG_WEBHOOK="false"` in production.

There is no polished interface for viewing webhook logs; you can view them by opening the log file directly.

!!! note
    You should still view logs to make sure the deploykit runs are executing successfully when using the webhook listener, even if you do not log webhook requests. A good way to do a first test is by checking how many runs are available with `php-deploykit --logs` before sending a test webhook, then sending the test webhook and checking if a new run appears in the logs, it should be in green. If it appears, but read, you can check the error there, if you still cannot figure it out, or no run appears, enable webhook logging and trigger another test webhook, then check the webhook log file for errors.

!!! info
    You must use systemctl to restart the webhook listener if you change the .env variables related to the webhook listener, since the listener only reads those variables on startup.