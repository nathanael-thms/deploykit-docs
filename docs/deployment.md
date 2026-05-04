---
icon: lucide/cloud-upload
---

# Deployment

!!! info
    php-deploykit as a command means the run.sh script from its installation location. If you created a symlink into PATH, php-deploykit (or the name you chose). Otherwise the script using the full path to the run.sh file.

!!! warning
    Ensure .env is properly configured, while configuring, **do not expose the site** until fully configured and you have tested a deployment

You deploy the application by running `php-deploykit --deploy`, for information about deployment types, available pre-flight checks and persistent files/directories(for symlink) see below


Below are the available deployment types, you specify which one to use via the `SYMLINK_DEPLOYMENT` variable in .env. Symlink deployment is recommended over classical deployment for its zero downtime and easy rollbacks. Also, the deployment steps run are listed under each type

## Classical deployment

In this method, the deployment script will change to the application directory, specified with `APP_DIR` in .env, run git pull(if enabled) and all the other necessary steps to update the app. This method is not recommended because it can cause downtime during deployment. Also, if the deployment fails, the app will be left in a broken state until the next deployment.

This really only exists so you can drop your code into the app directory, run your first deployment, then use the auto migration feature to update the app without having to worry about the git repository. It is not recommended for regular use.

### Steps run

#### Laravel

1. cd to `APP_DIR`
2. If `DOWN_APP` is enabled, run `php artisan down`
3. Run [before changes custom script](#before-changes) and cd back to `APP_DIR`
4. If `GIT_PULL` is enabled, run `git fetch --all --prune`, `git checkout -B "$TARGET_BRANCH" "origin/$TARGET_BRANCH"` and `git clean -fdelse`
5. If `RUN_NPM` is enabled, run `npm install` and `npm run $NPM_COMMAND`
6. Run `composer install --no-dev --optimize-autoloader --no-interaction`
7. If `MIGRATE` is enabled, run `php artisan migrate --force`
8. If `OPTIMIZE` is enabled, run `php artisan optimize`
9. Run [after changes custom script](#after-changes) and cd back to `APP_DIR`
10. If the app was put down in step 2, run `php artisan up`

!!! note
    If `BRING_APP_UP_ON_FAILURE` is enabled, `php artisan up` will be run whenever a command fails

#### Symfony

1. cd to `APP_DIR`
2. Run [before changes custom script](#before-changes) and cd back to `APP_DIR`
3. If `GIT_PULL` is enabled, run `git fetch --all --prune`, `git checkout -B "$TARGET_BRANCH" "origin/$TARGET_BRANCH"` and `git clean -fdelse`
4. If `RUN_NPM` is enabled, run `npm install` and `npm run $NPM_COMMAND`
5. Run `composer install --no-dev --optimize-autoloader --no-interaction`
6. If `MIGRATE` is enabled, run `php bin/console doctrine:migrations:migrate --no-interaction --allow-no-migration`
7. If `OPTIMIZE` is enabled, run `php bin/console cache:clear --no-warmup --quiet`, `php bin/console cache:warm --quiet` and `php bin/console cache:warmup --env=prod`
8. Run [after changes custom script](#after-changes) and cd back to `APP_DIR`

## Symlink deployment (recommended)

In this method, a new directory is made at `APP_DIR`/releases/timestamp, then git will clone the repository specified with `SYMLINK_DEPLOYMENT_GIT_PATH` in .env to a depth of 1 into it, then change into the new directory and run all the necessary steps to prepare the app for deployment. If everything goes well, the `APP_DIR`/current symlink will be updated to point to the new release. This method is recommended because it allows for zero downtime deployments and easy rollbacks in case of failure.

### Steps run

#### Laravel

1. Create a new release directory at `APP_DIR/releases/timestamp`
2. cd to the new release directory
3. Run the [before changes custom script](#before-changes)
4. Run `git clone --branch "$GIT_BRANCH" --depth 1 "$SYMLINK_DEPLOYMENT_GIT_PATH" "$NEW_RELEASE_DIR"`
5. Symlink all files/directories in shared to the new release folder, overwriting anything existing
6. cd to the new release directory
7. If `RUN_NPM` is enabled, run `npm install` and `npm run $NPM_COMMAND`
8. Run `composer install --no-dev --optimize-autoloader --no-interaction`
9. If `MIGRATE` is enabled, run `php artisan migrate --force`
10. If `OPTIMIZE` is enabled, run `php artisan optimize`
11. Run the [before linking custom script](#before-linking-symlink-mode-only)
12. Run `ln -sfn "$NEW_RELEASE_DIR" "$APP_DIR/current"`
13. cd to $APP_DIR/current
14. Run [after changes custom script](#after-changes)
15. If `AUTO_CLEANUP` is enabled, run `project_root/utilities/clean_up_releases.sh` and pass `KEEP_RELEASES`

#### Symfony

1. Create a new release directory at `APP_DIR/releases/timestamp`
2. cd to the new release directory
3. Run the [before changes custom script](#before-changes)
4. Run `git clone --branch "$GIT_BRANCH" --depth 1 "$SYMLINK_DEPLOYMENT_GIT_PATH" "$NEW_RELEASE_DIR"`
5. Symlink all files/directories in shared to the new release folder, overwriting anything existing
6. cd to the new release directory
7. If `RUN_NPM` is enabled, run `npm install` and `npm run $NPM_COMMAND`
8. Run `composer install --no-dev --optimize-autoloader --no-interaction`
9. If `MIGRATE` is enabled, run `php bin/console doctrine:migrations:migrate --no-interaction --allow-no-migration`
10. If `OPTIMIZE` is enabled, run `php bin/console cache:clear --no-warmup --quiet`, `php bin/console cache:warm --quiet` and `php bin/console cache:warmup --env=prod`
11. Run the [before linking custom script](#before-linking-symlink-mode-only)
12. Run `ln -sfn "$NEW_RELEASE_DIR" "$APP_DIR/current"`
13. cd to $APP_DIR/current
14. Run [after changes custom script](#after-changes)
15. If `AUTO_CLEANUP` is enabled, run `project_root/utilities/clean_up_releases.sh` and pass `KEEP_RELEASES`

### Setup

To set symlink deployment up, just create the `releases` directory in `APP_DIR`, and optionally a `shared` directory if you want to use persistent files/directories, then run a deployment with `SYMLINK_DEPLOYMENT` set to true.

### Persistent files/directories

By default, only the files in the repository are deployed, but you can make other files or directories persistent across deployments by moving them to `APP_DIR`/shared and creating a symlink to them in the deployment script. For example, if you want to make the `storage` directory persistent, you would move it to `APP_DIR`/shared/storage and redeploy the application to symlink it automatically.

## Pre-flight checks

php-deploykit can execute pre flight checks to make scripts 'fail-fast'. These generally take less than 2 seconds to execute and are executed on every deployment if `PRE_FLIGHT_CHECKS` is set to true in .env. They check the following

- Ensures necessary commands are present on the system
- Checks you have write permission to `APP_DIR`
- Checks you have the space specified by `MIN_STORAGE_GB` in .env

It is strongly recommended to enable these in production environments.

## Custom commands

php-deploykit supports custom commands, with options to run them at 3 points in deployments(or 2 only in classical mode), they are listed below

### Before changes

These steps, specified in `{project_root}/custom-before-changes.sh` are executed before pulling the git code, from the new releases or `APP_DIR` directory

For example, as symfony does not support putting the app down, you can put logic here(such as using touch to a lockfile), and put it back up in [after changes](#after-changes)

### Before linking (symlink mode only)

These steps, specified in `{project_root}/custom-before-linking.sh` are executed before linking the new release directory to the `current` directory.

For example, you can execute custom build commands other than NPM

### After changes

These steps, specified in `{project_root}/custom-after-changes.sh` are executed at the end of the deployment(in laravel, if enabled, before putting the app back up)

For example, as symfony does not support putting the app down, you can put logic here(such as using deleting a lockfile), and have it put down in [before changes](#before-changes)