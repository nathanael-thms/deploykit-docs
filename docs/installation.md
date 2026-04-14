---
icon: lucide/rocket
---

# Install/Update

## Required packages

php-deploykit aims to use as few packages as possible beyond the coreutils included in most Linux distributions (for example cp, mv, and ls), but some packages are required:

- PHP(obviously)
- Composer
- git (if using the symlink deployment method or git pull in classical method)
- Node.js/npm (if using the npm related step in the deployment script)
- rsync (used for migration to symlink method)
- SSH (if you clone the repository using SSH, otherwise it is not required)
- python3 (currently the webhook server is written in python)

All scripts use /bin/bash and /usr/bin/env python3; make sure they are present

## Installation steps

### Automatic installation (recommended)

The recommended installation method is to use the install.sh script, which will automatically install php-deploykit and create a symlink to run.sh in /usr/local/bin. This allows you to run php-deploykit from anywhere without having to specify the full path to the run.sh script. To install, run the following command from the parent directory of where you want to install php-deploykit:
!!! note
    This installer is interactive and needs user input. Runs sudo scripts. May prompt for password. You may open install.sh in the GitHub repository to review it before running the installer if you have any concerns.
```bash
curl -sSL https://raw.githubusercontent.com/nathanael-thms/php-deploykit/refs/heads/main/install.sh | bash
```

### Manual installation(if preferred)

1. Install the required packages listed above
2. Clone the latest version of the repository using the following command:
```bash
git clone --branch v0.4.0 --depth 1 https://github.com/nathanael-thms/php-deploykit.git
```
3. Ensure run.sh is executable, in the directory where you cloned the repository, run:
```bash
chmod +x run.sh
```
4. If you want to install php-deploykit globally, run the following from the parent directory of php-deploykit (do not run this inside the php-deploykit directory). Replace the target path or name as desired.
```bash
# If you have more than one app, you may want to move it to something else, eg.
# sudo cp -r php-deploykit /opt/php-deploykit-app
# sudo ln -s /opt/php-deploykit-app/run.sh /usr/local/bin/php-deploykit-app

# move code
sudo cp -r php-deploykit /opt/php-deploykit

# create symlink of run.sh into PATH
sudo ln -s /opt/php-deploykit/run.sh /usr/local/bin/php-deploykit
```
From now on, when instructed to "run php-deploykit", execute the run.sh script from its installation location. If you created a symlink into PATH, you can run php-deploykit (or the name you chose). Otherwise run the script using the full path to the run.sh file.

## Update steps

### If installed with the installer

Re-run the installation script at [automatic installation](#automatic-installation-recommended)

### If installed manually

To update php-deploykit, simply pull the latest changes from the repository. If you installed it globally, make sure to pull the changes in the installation directory (eg. /opt/php-deploykit if you followed the example above). If you did not install it globally, pull the changes in the directory where you cloned the repository.

```bash
# If you installed globally, navigate to the installation directory, eg.
cd /opt/php-deploykit
# If you did not install globally, navigate to the directory where you cloned the repository, eg.
# cd ~/php-deploykit

git fetch --tags --depth=1
git checkout v0.4.0
```