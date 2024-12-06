# Ubuntu-upgrade-script
A script to streamline and automate the upgrade process of Ubuntu systems.

This script performs a fully automated upgrade of an Ubuntu server to the next LTS version without requiring manual interaction. It is designed for administrators who wish to streamline and simplify their upgrade process. Key features include:

- Creating a full backup of the `/etc` configuration directory.
- Configuring `dpkg` for automatic merging of configuration files.
- Running comprehensive system updates (`apt-get update`, `upgrade`, `dist-upgrade`, `autoremove`).
- Performing the release upgrade to the next Ubuntu LTS version using `do-release-upgrade`.
- Cleaning up temporary files after the upgrade.
- Automatically rebooting the system upon completion.

The script also detects and adjusts for optional components as needed:

- **AdGuard Home on Port 53:**  
  If a Docker container named `adguardhome` is present, the script ensures that port 53 is made available, `systemd-resolved` is disabled, and AdGuard Home is restarted after the upgrade.

- **2FA via Google Authenticator:**  
  If `libpam-google-authenticator` is installed, the script ensures that two-factor authentication for SSH is properly configured.

Other Docker containers (e.g., Nextcloud, Vaultwarden, Portainer) are not modified and remain unaffected by the script’s actions.

## Requirements

- An Ubuntu LTS server (e.g., Ubuntu 20.04) that can be upgraded to the next LTS version (e.g., 22.04).
- `do-release-upgrade` should be available (commonly included in the `ubuntu-release-upgrader-core` package).
- The script must be run as `root`.

## Usage

1. Download or create the script locally:
   ```bash
   wget https://raw.githubusercontent.com/your-username/your-repo/main/upgrade-script.sh
   chmod +x upgrade-script.sh

    Run the script as root:

    sudo ./upgrade-script.sh

    The script will perform all necessary steps automatically. No further input is required.

Logging & Troubleshooting

    All actions are logged to /var/log/ubuntu-lts-upgrade.log.
    In case of errors or issues, consult this log file to review the upgrade process and identify potential problems.

Customization

The script is modular and can be easily adjusted. If you prefer not to disable systemd-resolved or do not wish to configure 2FA, feel free to comment out or remove the relevant sections. Review the function blocks for tailoring the script to your specific environment.
Disclaimer

Use this script at your own risk. It is highly recommended to perform a full system backup before initiating the upgrade.

✉️ Contact

For questions or suggestions, feel free to open a GitHub Issue or send an email to ptech09@schumacher.or.at.

License

This project is licensed under the MIT License. Please refer to the LICENSE file for more details.
