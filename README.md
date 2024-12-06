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



# Ubuntu LTS Upgrade Skript

Dieses Skript führt ein vollautomatisiertes Upgrade eines Ubuntu-Servers auf die nächste LTS-Version durch, ohne dass manuelle Eingriffe erforderlich sind. Es richtet sich an Administratoren, die ihren Upgrade-Prozess vereinfachen und standardisieren möchten. Wesentliche Funktionen sind:

- Erstellen eines vollständigen Backups des `/etc`-Verzeichnisses.
- Konfiguration von `dpkg` für das automatische Zusammenführen von Konfigurationsdateien.
- Ausführen umfangreicher System-Updates (`apt-get update`, `upgrade`, `dist-upgrade`, `autoremove`).
- Durchführung des Release-Upgrades auf die nächste Ubuntu LTS-Version mittels `do-release-upgrade`.
- Bereinigung temporärer Dateien nach dem Upgrade.
- Automatischer Neustart des Systems nach Abschluss.

Das Skript erkennt optional vorhandene Komponenten und passt das Vorgehen entsprechend an:

- **AdGuard Home auf Port 53:**  
  Ist ein Docker-Container mit dem Namen `adguardhome` vorhanden, sorgt das Skript dafür, dass Port 53 freigegeben, `systemd-resolved` deaktiviert und AdGuard Home nach dem Upgrade neu gestartet wird.

- **2FA mit Google Authenticator:**  
  Ist das Paket `libpam-google-authenticator` installiert, stellt das Skript sicher, dass die Zwei-Faktor-Authentifizierung für SSH korrekt eingerichtet ist.

Andere Docker-Container (z. B. Nextcloud, Vaultwarden, Portainer) werden nicht verändert und bleiben von den Aktionen des Skripts unberührt.

## Voraussetzungen

- Ein Ubuntu LTS-Server (z. B. Ubuntu 20.04), der auf die nächste LTS-Version (z. B. 22.04) aktualisiert werden kann.
- `do-release-upgrade` sollte verfügbar sein (in der Regel im Paket `ubuntu-release-upgrader-core` enthalten).
- Das Skript muss als `root` ausgeführt werden.

## Verwendung

1. Skript herunterladen oder lokal erstellen:
   ```bash
   wget https://raw.githubusercontent.com/dein-benutzername/dein-repo/main/upgrade-script.sh
   chmod +x upgrade-script.sh

    Als root ausführen:

    sudo ./upgrade-script.sh

    Das Skript führt alle erforderlichen Schritte automatisch durch. Kein weiterer Eingriff ist nötig.

Logging & Fehlersuche

    Alle Aktionen werden in /var/log/ubuntu-lts-upgrade.log protokolliert.
    Im Fehlerfall oder bei Problemen kann diese Logdatei zur Fehlersuche herangezogen werden.

Anpassungen

Das Skript ist modular aufgebaut und kann leicht angepasst werden. Wenn Du z. B. systemd-resolved nicht deaktivieren oder keine 2FA einrichten möchtest, kannst Du die entsprechenden Abschnitte auskommentieren oder entfernen. Überprüfe die Funktionsblöcke, um das Skript an Deine Umgebung anzupassen.
Haftungsausschluss

Die Verwendung des Skripts erfolgt auf eigene Gefahr. Es wird dringend empfohlen, vor dem Upgrade ein vollständiges Backup des Systems anzufertigen.

✉️ Kontakt

Bei Fragen oder Anregungen kannst Du gerne ein GitHub Issue eröffnen oder eine E-Mail an ptech09@schumacher.or.at senden.

Lizenz

Dieses Projekt steht unter der MIT-Lizenz. Weitere Informationen findest Du in der LICENSE-Datei.
