---
icon: lucide/terminal
---

# Usage

!!! info
    php-deploykit as a command means the run.sh script from its installation location. If you created a symlink into PATH, php-deploykit (or the name you chose). Otherwise the script using the full path to the run.sh file.

## Running with flags

Available flags for the `php-deploykit` command:

| Flag | Function |
|---|---|
| `deploy` | Same as running without flags and selecting option 1. Performs a classical or symlink deployment according to `.env`. Does not require user interaction. |
| `migrate` | Same as selecting option 2. Starts migration to symlink deployment as described in [Migration to symlink deployment](#migration-to-symlink-deployment). Requires user interaction. |
| `revert` | Same as selecting option 3 (symlink only). Reverts to a previous deployment as described in [Reverting to a previous deployment](#reverting-to-a-previous-deployment). Requires user interaction. |
| `first` | Same as selecting option 4. Use this for the initial deployment; it prevents the app-down command from being run in classical mode. Does not require user interaction but oversight is recommended. |
| `cleanup` | Same as selecting option 5 (symlink only). Cleans up old releases as described in [Cleaning up old releases](#cleaning-up-old-releases). Can be followed by `-<n>` (for example `--cleanup-10`) to keep the latest `n` releases. Requires user interaction if not followed by an integer. |
| `logs` | Same as selecting option 6. Displays all the deployments, red if failed and green if successful. Then prompts for you to select a deployment to view logs for. If all you wanted to do is check if it was successful, press Ctrl+C after viewing the logs. |
| `webhook-listener` | Starts the webhook listener. Meant to be executed by a systemd service. Starts the webhook listener, which listens for incoming webhook requests and triggers deployments when a request is received. It calls `webhook_listener/webhook_listener.sh`. |
| `webhook-service-install` | Installs the webhook listener as a systemd service. |
| `webhook-service-uninstall` | Uninstalls the webhook listener systemd service. It calls `utilities/webhook_listener_service_uninstall.sh`, which stops the service, disables it from starting on boot, and removes the service file. |
| `help` | Prints the available flags. |

Only one option may be specified at a time. Example:

`php-deploykit --deploy`

## Running without flags

!!! note
    Some options can only be run using flags, to prevent accidental execution. Those options include the detailed info in the table, the rest have the detailed info here.

    webhook-listener cannot be run without flags because once run, does not ask for further human intervention and can cause port conflicts. webhook-service-uninstall because it can be mistaken for service install.

Running without flags presents a menu where you can select 1, 2, 3, 4, 5, 6 or 7. More detailed information on each option follows.

### Option 1/deploy

This calls `deploy-logic/deploy.sh`. That script checks `.env` to determine whether to use symlink or classical deployment, then runs the framework-specific deployment script. Currently only Laravel is supported; for Laravel with symlink enabled it runs `deploy-logic/laravel/deploy_symlink.sh`.

### Option 2/migrate

This starts the migration to symlink deployment as described in [Migration to symlink deployment](#migration-to-symlink-deployment). It calls `utilities/migrate_to_symlink.sh`.

### Option 3/revert

This reverts to a previous deployment when using symlink deployment, as described in [Reverting to a previous deployment](#reverting-to-a-previous-deployment). It calls `utilities/revert_to_previous_deployment.sh`.

### Option 4/first

Performs the same actions as option 1 but ensures that the app-down command (for example `php artisan down`) is not run in classical deployment.

### Option 5/cleanup

Cleans up old releases when using symlink deployment, as described in [Cleaning up old releases](#cleaning-up-old-releases). It calls `utilities/clean_up_releases.sh`.

### Option 6/logs.

Displays all deployments, red if failed and green if successful. Then prompts for you to select a deployment to view logs for. Only works if logging is enabled. If all you wanted to do is check if it was successful, press Ctrl+C after viewing the logs. It calls `utilities/view_logs.sh`.

### Option 7/webhook-service-install

This installs the webhook listener as a systemd service. It calls `utilities/webhook_listener_service_install.sh`, which creates a systemd service file for the webhook listener and starts the service. The service is configured to start on boot and restart automatically if it fails.