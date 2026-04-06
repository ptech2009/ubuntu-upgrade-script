# Ubuntu LTS Upgrade Skript

Ein Bash-Skript für vollautomatisierte, sichere Ubuntu-LTS-Release-Upgrades ohne manuelle Eingriffe.

---

## Beschreibung

Dieses Skript richtet sich an Administratoren, die ihren Upgrade-Prozess auf Ubuntu-Servern standardisieren und vereinfachen möchten. Es setzt auf `do-release-upgrade`, beantwortet Standard-Prompts automatisch und bricht bei bekannten Risikofällen bewusst ab – ohne blindes `yes | ...`.

### Wesentliche Funktionen

- **Sprachauswahl:** Deutsch und Englisch werden unterstützt; bei nicht-interaktivem Betrieb wird automatisch Deutsch gewählt.
- **Preflight-Checks:** Prüfung des Betriebssystems (nur Ubuntu LTS), der aktuellen Version, des freien Speicherplatzes (Standard: 6 GB auf `/`, 200 MB auf `/boot`), der APT-Quellen auf Drittanbieter-Repositories sowie bekannter Sonderfälle (RabbitMQ, Dovecot, Postfix).
- **Umfangreiches Backup:** Sicherung von `/etc`, APT-Quellen, SSH-Konfiguration, Systeminformationen und DNS-Zustand vor dem Upgrade.
- **Unattended-Modus:** Automatische Konfiguration von `dpkg`, `needrestart`, `debconf` und `apt` für vollständig unterbrechungsfreie Upgrades.
- **System-Updates:** Vollständige Paketaktualisierung (`update`, `upgrade`, `dist-upgrade`, `autoremove`) vor dem Release-Upgrade.
- **Release-Prüfung:** Verifikation, ob das Zielrelease über `do-release-upgrade` offiziell angeboten wird – kein blindes Upgrade auf nicht freigegebene Versionen.
- **Release-Upgrade:** Ausführung von `do-release-upgrade` mit Session-Transcript-Logging.
- **Post-Upgrade-Cleanup:** Bereinigung von Paketresten und Erfassung des Systemzustands nach dem Upgrade.
- **Neustart-Kontrolle:** Konfigurierbares automatisches oder interaktives Neustartverhalten.

### Optional: AdGuard Home / DNS

Wenn `MANAGE_ADGUARD_DNS=yes` gesetzt ist und ein Docker-Container mit dem konfigurierten Namen vorhanden ist, sorgt das Skript dafür, dass:

- `systemd-resolved` deaktiviert wird, damit AdGuard Port 53 übernehmen kann,
- `/etc/resolv.conf` statisch auf sichere DNS-Server gesetzt wird,
- der AdGuard-Container nach dem Upgrade neu gestartet wird.

Andere Docker-Container (z. B. Nextcloud, Vaultwarden, Portainer) werden nicht verändert.

### Optional: 2FA mit Google Authenticator

Ist `libpam-google-authenticator` installiert, prüft das Skript **nicht-destruktiv**, ob der PAM-Eintrag für SSH korrekt vorhanden ist, und gibt im Fehlerfall eine Warnung aus.

---

## Voraussetzungen

- Ubuntu LTS-Server ab Version **24.04**, der auf die nächste LTS-Version (Standard: 26.04) aktualisiert werden soll.
- `do-release-upgrade` muss verfügbar sein (Paket: `ubuntu-release-upgrader-core`); fehlende Hilfspakete werden automatisch nachinstalliert.
- Das Skript muss als **root** ausgeführt werden.

---

## Konfiguration

Das Skript ist vollständig über Umgebungsvariablen steuerbar – keine Änderungen im Skript selbst notwendig:

| Variable | Standard | Beschreibung |
|---|---|---|
| `TARGET_LTS` | `26.04` | Ziel-LTS-Version |
| `PROMPT_MODE` | `lts` | Release-Upgrade-Modus (`lts` oder `normal`) |
| `ALLOW_DEVEL_UPGRADE` | `no` | Development-Release erlauben (`yes`/`no`) |
| `UNATTENDED_MODE` | `yes` | Vollautomatischer Modus (`yes`/`no`) |
| `AUTO_CONFIRM` | `yes` | Bestätigungen automatisch annehmen (`yes`/`no`) |
| `AUTO_REBOOT` | `no` | Automatischer Neustart nach Upgrade (`yes`/`no`) |
| `ASK_REBOOT` | `no` | Neustart interaktiv erfragen (`yes`/`no`) |
| `ALLOW_THIRD_PARTY` | `no` | Drittanbieter-Repos erlauben (`yes`/`no`) |
| `ALLOW_SPECIAL_CASES` | `no` | Bekannte Sonderfälle (RabbitMQ, Dovecot) erlauben (`yes`/`no`) |
| `MANAGE_ADGUARD_DNS` | `no` | AdGuard-/DNS-Verwaltung aktivieren (`yes`/`no`) |
| `ADGUARD_CONTAINER_NAME` | `adguardhome` | Name des AdGuard-Docker-Containers |
| `MIN_ROOT_FREE_GB` | `6` | Mindest-Freispeicher auf `/` in GB |
| `MIN_BOOT_FREE_MB` | `200` | Mindest-Freispeicher auf `/boot` in MB |
| `LOGFILE` | `/var/log/ubuntu-lts-upgrade.log` | Pfad zur Logdatei |
| `BACKUP_DIR` | `/root/ubuntu_upgrade_backup_DATUM` | Pfad zum Backup-Verzeichnis |
| `LANGUAGE` | `de` | Standardsprache (`de`/`en`) |
| `LANGUAGE_CHOICE` | *(leer)* | Sprache überschreiben ohne interaktive Abfrage |

### Beispiel mit überschriebenen Variablen

```bash
TARGET_LTS=26.04 AUTO_REBOOT=yes MANAGE_ADGUARD_DNS=yes sudo ./upgrade-script.sh
```

---

## Verwendung

1. Skript herunterladen:
   ```bash
   wget https://raw.githubusercontent.com/dein-benutzername/dein-repo/main/upgrade-script.sh
   chmod +x upgrade-script.sh
   ```

2. Als root ausführen:
   ```bash
   sudo ./upgrade-script.sh
   ```

Das Skript führt alle Schritte automatisch durch. Bei `UNATTENDED_MODE=yes` (Standard) ist kein weiterer Eingriff erforderlich.

---

## Ablauf

1. Sprachauswahl (interaktiv oder automatisch)
2. Root-Prüfung
3. Logging initialisieren
4. **Preflight-Checks:**
   - OS- und LTS-Prüfung
   - Versionsbereich prüfen (≥ 24.04, < Zielversion)
   - Benötigte Pakete installieren
   - Backup anlegen (`/etc`, APT, SSH, Systemzustand, DNS)
   - Unattended-Modus konfigurieren
   - Freien Speicher prüfen
   - Release-Prompt setzen
   - Drittanbieter-Repos inventarisieren und Policy durchsetzen
   - Sonderfälle prüfen
   - SSH- und 2FA-Konfiguration prüfen (nicht-destruktiv)
   - Zusammenfassung ausgeben
5. Bestätigung einholen (oder automatisch fortfahren)
6. Alle ausstehenden Paketupdates installieren
7. Release-Angebot über `do-release-upgrade -c` verifizieren
8. Release-Upgrade durchführen (mit Session-Transcript)
9. AdGuard-/DNS-Anpassung (optional)
10. Post-Upgrade-Cleanup und Statussicherung
11. Neustart (automatisch, interaktiv oder manuell)

---

## Logging & Fehlersuche

- Alle Aktionen werden in `/var/log/ubuntu-lts-upgrade.log` protokolliert.
- Der vollständige `do-release-upgrade`-Sitzungsverlauf wird als `do-release-upgrade.session.log` im Backup-Verzeichnis gespeichert.
- Zusätzliche Upgrade-Logs von Ubuntu liegen typischerweise unter `/var/log/dist-upgrade/`.
- Das Backup-Verzeichnis enthält Vorher-/Nachher-Snapshots von `dpkg -l`, `lsb_release` und weiteren Systemzuständen.

---

## Sicherheit & Fehlschutz

- Das Skript verwendet `set -Eeuo pipefail` und `umask 077`.
- Fehler werden mit Zeilennummer und Befehl geloggt; temporäre Konfigurationsdateien werden beim Beenden automatisch entfernt.
- Drittanbieter-Repositories führen standardmäßig zum Abbruch (`ALLOW_THIRD_PARTY=no`).
- Bekannte Sonderfälle (RabbitMQ, Dovecot) führen standardmäßig zum Abbruch (`ALLOW_SPECIAL_CASES=no`).
- Reboot-Anforderungen nach regulären Updates werden erkannt und führen zu einem kontrollierten Abbruch mit Hinweis.
- Der Upgrade-Befehl selbst wird nie mit `yes |` aufgerufen.

---

## Anpassungen

Das Skript ist modular aufgebaut. Einzelne Funktionsblöcke können bei Bedarf auskommentiert oder entfernt werden:

- `manage_adguard_dns_after_upgrade` – AdGuard-/DNS-Anpassung
- `verify_ssh_and_2fa_non_destructive` – SSH- und 2FA-Prüfung
- `check_special_cases` – Erkennung von RabbitMQ, Dovecot, Postfix
- `enforce_repo_policy` – Drittanbieter-Repository-Policy

---

## Haftungsausschluss

Die Verwendung des Skripts erfolgt auf eigene Gefahr. Es wird dringend empfohlen, vor dem Upgrade ein vollständiges System-Backup anzufertigen. Das Skript selbst legt ein Konfigurations-Backup an, ersetzt aber kein vollständiges Systembackup.

---

## Lizenz

Dieses Projekt steht unter der **MIT-Lizenz**. Weitere Informationen findest Du in der `LICENSE`-Datei.

---

## Kontakt

Bei Fragen oder Anregungen kannst Du gerne ein [GitHub Issue](../../issues) eröffnen oder eine E-Mail an [ptech09@schumacher.or.at](mailto:ptech09@schumacher.or.at) senden.
