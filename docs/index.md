---
icon: lucide/house
---

# Introduction

php-deploykit is a project currently in development that allows deployment of PHP apps in two ways: classical and symlink (recommended) on Linux servers.

!!! warning
    Use in production environments at your own risk, but it is not yet recommended. Breaking changes may occur before the v1.0.0 release.

## Classical deployment

In this method, the deployment script will change to the application directory, specified with `APP_DIR` in .env, run git pull(if enabled) and all the other necessary steps to update the app. This method is not recommended because it can cause downtime during deployment. Also, if the deployment fails, the app will be left in a broken state until the next deployment.

This really only exists so you can drop your code into the app directory, run your first deployment, then use the auto migration feature to update the app without having to worry about the git repository. It is not recommended for regular use.

## Symlink deployment (recommended)

In this method, a new directory is made at `APP_DIR`/releases/timestamp, then git will clone the repository specified with `SYMLINK_DEPLOYMENT_GIT_PATH` in .env to a depth of 1 into it, then change into the new directory and run all the necessary steps to prepare the app for deployment. If everything goes well, the `APP_DIR`/current symlink will be updated to point to the new release. This method is recommended because it allows for zero downtime deployments and easy rollbacks in case of failure.

## Features

- Zero downtime deployments (symlink method)
- Easy rollbacks (symlink method)
- Auto migration to symlink
- Automatic webhook support for GitHub, GitLab, and Bitbucket
- View if the deployment passed, failed or is in progress just by checking on the GitHub commit page(if using GitHub and enabled)
- Logging system
- Easy log viewing, see at a glance which deployments failed and which succeeded(color coded), and view the logs for each deployment without manually opening the log files
- Automatic cleanup of old releases, keeping the latest releases specified with `KEEP_RELEASES` in .env(if enabled)
- Easy configuration with .env file
- Open source and free to use