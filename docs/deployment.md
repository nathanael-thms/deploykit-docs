---
icon: lucide/cloud-upload
---

# Deployment

!!! info
    php-deploykit as a command means the run.sh script from its installation location. If you created a symlink into PATH, php-deploykit (or the name you chose). Otherwise the script using the full path to the run.sh file.

!!! warning
    Ensure .env is properly configured, while configuring, **do not expose the site** until fully configured and you have tested a deployment

You deploy the application by running `php-deploykit --deploy`, for information about deployment types, available pre-flight checks and persistent files/directories(for symlink) see below

## Deployment types

You specify this via the `SYMLINK_DEPLOYMENT` variable in .env, if true, it will use symlink, if false, it will use classical

!!! warning
    Do not set this to true unless symlink deployment is actually set up (that is, current and releases directories exist). You will most likely also want a shared directory.

### Classical deployment

In this method, the deployment script will change to the application directory, specified with `APP_DIR` in .env, run git pull(if enabled) and all the other necessary steps to update the app. This method is not recommended because it can cause downtime during deployment. Also, if the deployment fails, the app will be left in a broken state until the next deployment.

This really only exists so you can drop your code into the app directory, run your first deployment, then use the auto migration feature to update the app without having to worry about the git repository. It is not recommended for regular use.

### Symlink deployment (recommended)

In this method, a new directory is made at `APP_DIR`/releases/timestamp, then git will clone the repository specified with `SYMLINK_DEPLOYMENT_GIT_PATH` in .env to a depth of 1 into it, then change into the new directory and run all the necessary steps to prepare the app for deployment. If everything goes well, the `APP_DIR`/current symlink will be updated to point to the new release. This method is recommended because it allows for zero downtime deployments and easy rollbacks in case of failure.

#### Persistent files/directories

By default, only the files in the repository are deployed, but you can make other files or directories persistent across deployments by moving them to `APP_DIR`/shared and creating a symlink to them in the deployment script. For example, if you want to make the `storage` directory persistent, you would move it to `APP_DIR`/shared/storage and redeploy the application to symlink it automatically.

### Pre-flight checks

php-deploykit can execute pre flight checks to make scripts 'fail-fast'. These generally take less than 2 seconds to execute and are executed on every deployment if `PRE_FLIGHT_CHECKS` is set to true in .env. They check the following

- Ensures necessary commands are present on the system
- Checks you have write permission to `APP_DIR`
- Checks you have the space specified by `MIN_STORAGE_GB` in .env

It is strongly recommended to enable these in production environments.