---
icon: lucide/wrench
---

# Utilities

!!! info
    php-deploykit as a command means the run.sh script from its installation location. If you created a symlink into PATH, php-deploykit (or the name you chose). Otherwise the script using the full path to the run.sh file.

php-deploykit comes with utility scripts to assist with various tasks related to deployment, such as migrating to symlink deployment, reverting to a previous deployment, and cleaning up old releases. Below is a list of the utilities and how to run them. Some have already been explained, such as [installing the webhook listener as a service](webhook.md#install-as-a-servicerecommended) and [starting the webhook listener manually](webhook.md#manual-start), which are also utilities, but are explained in more detail in the webhook listener documentation.

## Migrate to symlink deployment

Migration to symlink deployment can be complicated. This project includes a script to automate the process; it performs all steps except clearing/recaching application caches and updating your web server configuration. To run the script, execute php-deploykit, choose option 2, and follow the prompts. Afterwards clear and recache the application as instructed and update your web server to point to the new current directory.
!!! danger
    Read the script prompts carefully; failure to follow instructions may pose a security risk.

To make other files persistent across deployments, move them to APP_DIR/shared and redeploy, as described in the [persistent files/directories](deployment.md#persistent-filesdirectories) section of the introduction documentation.

## Revert to a previous deployment

A key advantage of symlink deployment is the ability to revert to a previous deployment if it is needed for whatever reason. The script remaps the current symlink to an older release directory. It lists releases in newest-to-oldest order and prompts you to select a number. Although named "revert", the script can also be used to move forward again if needed. To run it, execute php-deploykit, choose option 3, and follow the prompts.

## Clean up old releases

Over time your server may accumulate many releases. To clean them up, php-deploykit includes a cleanup script. When run, if there is more than one release, it prompts for how many releases to keep (for example, entering 10 keeps the 10 newest releases and deletes the rest). To run the script, execute php-deploykit, choose option 5, and follow the prompts. If you enter a number greater than or equal to the current number of releases, the script does nothing and exits with code 0, which makes automation easier.

!!! note
    Remember you can configure automatic cleanup in .env by setting `AUTO_CLEANUP="true"` and specifying how many releases to keep with `KEEP_RELEASES=<n>`. This way you can ensure that old releases are regularly cleaned up without needing to remember to run the cleanup script.

!!! danger
    If you have reverted or manually changed the current symlink to an older release, cleaning up by keeping the latest n releases may remove the directory pointed to by current. This will cause the web server to stop working.

## Log Viewer

Already briefly mentioned in the usage documentation, the log viewer displays all deployments with a color-coded status and allows you to select a deployment to view logs for. To run it, execute php-deploykit, choose option 6, and follow the prompts. Only works for [general logging](logging.md#general-logging) If all you wanted to do is check if it was successful, press Ctrl+C after viewing the logs.