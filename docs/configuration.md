---
icon: lucide/settings
---

# Configuration

The configuration of php-deploykit is done through the .env file. The .env file is located in the root directory of the project. You can copy the .env.example file to .env and then edit the values as needed.

```bash
cp .env.example .env
```

## Variables

- `APP_DIR`: This variable tells php-deploykit where your app is located
!!! note
    If you are using symlink deployment, this should not include current. For example, use /var/www/app rather than /var/www/app/current.
- `SYMLINK_DEPLOYMENT`: Set to `true` or `false`. When `true`, the script uses symlink deployment; when `false`, it uses classical deployment.
!!! warning
    Do not set this to `true` unless symlink deployment is actually set up (that is, `current` and `releases` directories exist). You will most likely also want a `shared` directory.

- `GIT_PULL`: A `true`/`false` variable relevant only for classical deployment; it is ignored for symlink deployments. It will usually be `true`, since there is currently no other automated method to retrieve code.

- `GIT_BRANCH`: This variable is relevant whether you are using classical or symlink. In classical, it specifies which branch to pull from, in symlink, it specifies which branch to clone

- `FRAMEWORK`: Specifies which PHP framework the app uses. Currently only Laravel is supported; Symfony support is planned.

- `MIGRATE`: A `true`/`false` variable relevant for both classical and symlink deployments. When `true`, the deployment script runs the migration command (for example, `php artisan migrate`).

- `OPTIMIZE`: A `true`/`false` variable relevant for both deployment types. When `true`, the deployment script runs the optimization command (for example, `php artisan optimize`).

- `RUN_NPM`: A `true`/`false` variable relevant for both deployment types. When `true`, the deployment script runs an npm command specified by `NPM_COMMAND`.

- `NPM_COMMAND`: Specifies the npm command to run. This can be omitted if `RUN_NPM="false"`; if present but `RUN_NPM="false"`, it will be ignored. For example, setting `build` runs `npm run build`.

- `LOG`: A `true`/`false` variable relevant for both deployment types. When `true`, the deployment script stores output in a log file specified by `LOG_FILE`.

- `LOG_FILE`: Specifies where to store logs. This can be omitted if `LOG="false"`; if present but `LOG="false"`, it will be ignored. Ensure you have permissions to write to the specified file; if the file does not exist, the script will try to create it and may fail if you lack permissions for the parent directory.

- `DOWN_APP`: A `true`/`false` variable relevant only for classical deployment. When `true`, the deployment script runs the command to put the app down before deployment and bring it back up on success (for example, `php artisan down` && `php artisan up`). symlink deployment is zero-downtime, so this is not used there.
!!! warning
    If any part of the script fails while the app is down, it will remain down. You must manually bring it back up unless `BRING_APP_UP_ON_FAILURE="true"`.

- `BRING_APP_UP_ON_FAILURE`: A `true`/`false` variable relevant only for classical deployment when `DOWN_APP="true"`. When `true`, the deployment script will attempt to bring the app back up if the main deployment process fails and the app had been put down (for example, `php artisan up`). This is not used for symlink deployments.
!!! Danger
    This is strongly discouraged: if bringing the app back up fails during a partially completed deployment, it could expose a broken application to the public and create security risks This is one reason symlink deployments are preferable: a failed deployment will not be made public.

- `SYMLINK_DEPLOYMENT_GIT_PATH`: Relevant only for symlink deployment; this tells the script where to clone the repository. For GitHub, using SSH is recommended, for example: `SYMLINK_DEPLOYMENT_GIT_PATH="git@github.com:user/app.git"`. The script will run a command similar to: `git clone --branch "<GIT_BRANCH>" --depth 1 git@github.com:user/app.git "<APP_DIR>/releases/<timestamp>"`

- `PRE_FLIGHT_CHECKS`:  A `true`/`false` variable relevant only for both deployment types. When `true`, the script automatically performs pre flight checks as described in [pre-flight checks](index.md#pre-flight-checks)

- `MIN_STORAGE_GB`: Specifies the npm command to run. This can be omitted if `PRE_FLIGHT_CHECKS="false"`; if present but `PRE_FLIGHT_CHECKS="false"`, it will be ignored. Specifies how much storage to check for in pre flight checks as described in [pre-flight checks](index.md#pre-flight-checks)

- `AUTO_CLEANUP`: A `true`/`false` variable relevant only for symlink deployment. When `true`, the script automatically cleans up old releases, keeping the latest `KEEP_RELEASES` releases.

- `KEEP_RELEASES`: This variable, only relevant for symlink deployment and when `AUTO_CLEANUP="true"` tells the auto cleanup script to keep the latest `KEEP_RELEASES` releases

- `WEBHOOK_PORT`: This variable is used to specify the port the webhook listener uses to listen on.

- `WEBHOOK_SECRET`: This variable is used to specify the secret the webhook listener uses to verify incoming requests.

- `WEBHOOK_PROVIDER`: This variable is used to specify the provider the webhook listener is expecting requests from. Supported values are `github`, `gitlab` and `bitbucket`.

- `LOG_WEBHOOK`: A `true`/`false` variable that specifies whether to log incoming webhook requests. If `true`, logs are stored in the file specified by `WEBHOOK_LOG_FILE`. Unless you are debugging, there is no need to log webhook requests, so it is recommended to set this to `false` in production.

- `WEBHOOK_LOG_FILE`: Specifies where to store logs of incoming webhook requests. This can be omitted if `LOG_WEBHOOK="false"`; if present but `LOG_WEBHOOK="false"`, it will be ignored. Ensure you have permissions to write to the specified file; ensure the file exists.

- `GITHUB_REPORTING`: A `true`/`false` variable that specifies whether to report deployment status back to GitHub when using the webhook listener with GitHub. If `true`, the script reports deployment status back to GitHub, which can be viewed by checking the repository's commits, described in [reporting deployment status back to GitHub](webhook.md#reporting-deployment-status-back-to-github). This is not used for other providers.

- `GITHUB_TOKEN`: If `GITHUB_REPORTING="true"`, this variable is used to specify a GitHub token with permissions to create deployment statuses in the repository. The script uses this token to authenticate with the GitHub API when reporting deployment status. Also described in [reporting deployment status back to GitHub](webhook.md#reporting-deployment-status-back-to-github).

- `GITHUB_REPO_OWNER` and `GITHUB_REPO_NAME`: If `GITHUB_REPORTING="true"`, these variables are used to specify the owner and name of the GitHub repository, respectively. The script uses these variables to identify which repository to report deployment status for when using the GitHub API. Also described in [reporting deployment status back to GitHub](webhook.md#reporting-deployment-status-back-to-github)

Values must be surrounded by double quotes (`""`) so the scripts parse them correctly.

Variables that are irrelevant to the chosen deployment mode will be ignored.
