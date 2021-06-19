# Hosted Instances
Sie unterstützen das junge Start-Up-Unternehmen ACME mit Ihren IT- und Cloud-Kenntnissen.
Das Unternehmen hat eine fertige Web-Seite erworben und möchte mit dieser nun live gehen.
Zurzeit mietet das Unternehmen lediglich einen Co-Working-Space für seine Mitarbeiter.
Eigene Server-Hardware ist nicht vorhanden.

## Kurzbeschreibung
Ziel dieser Übung ist es erste Erfahrungen im Umgang mit den sog. Hosted Instanzes zu machen.
Diese sind virtuelle Computer die auf Abruf von einem Cloud Provider bereitgestellt werden können.

## Anleitung

### Vorbereitungen
1. Schalte die AWS Management Console auf Englisch um.

### Starten der Instanz
1. Öffnen Sie den EC2-Service.
2. Vergewissern Sie sich, dass sie mit der Region Frankfurt verbunden sind.
3. Navigieren Sie zum Abschnitt `Instances`.
4. Clicken Sie auf `Launch instances`.
5. Wählen Sie als Image `Ubuntu Server 20.04 LTS` mit Chip-Architektur `64-bit (x86)` aus.
6. Wählen Sie als Instance Type `t2.micro`. Dies ist in der Regel vorausgewählt.
7. Erstellen Sie für die Instanz einen Tag `Name=<Ihr-Kürzel>`.
8. Weiter zum Reiter `Configure Security Group`.
9. Wählen Sie `Select an existing security group` und wählen Sie dann die Gruppe `AllowWebAndSSH` aus.
10. Starten Sie die Instanz.
    Wählen Sie im folgenden Dialog `Create a new key pair` im Dropdown.
    Geben Sie der Datei einen sinnvollen Namen und laden Sie die Schlüsseldatei runter.

### Verbindung zur Instanz herstellen
10. Ändern Sie den Datei-Modus der Schlüsseldatei auf `400`, sodass nur der Inhaber der Datei sie lesen kann.
    `chmod 400 <keyfile>`
11. Warten Sie bis die Instanz in den `Running`-State wechselt und verbinden Sie sich mit der Instanz über SSH.
    `ssh -i <keyfile> ubuntu@<instance-public-ip>`

Sie sollten nun mit der Instanz verbunden sein.

### Installation des Web-Servers
1. Installieren Sie den Web-Server und Reverse-Proxy `nginx`.
    `sudo apt update`
    `sudo apt install nginx`
2. Kontrollieren Sie, ob `nginx` korrekt ausgeführt wird.
    `systemctl status nginx`

### Konfiguration des Web-Servers
1. Innerhalb dieses Projekts, im Ordner `website` finden Sie die
    Webseite die das Unternehmen ACME bereitstellen möchte.
2. Öffnen Sie die `index.html` und ersetzen Sie den Titel und Untertitel durch etwas sinnvolles.
    Bspw.: Ihren Namen und/oder Kürzel
3. Kopieren Sie den Inhalt aus dem Ordner `website` zum Web-Server
    `scp -i <keyfile> website/* ubuntu@<instance-public-ip>:/home/ubuntu`
    `ssh -i <keyfile> ubuntu@<instance-public-ip> 'sudo cp ./* /var/www/html'`

### Konfiguration der DNS-Records
1. Öffnen Sie den Service `Route 53`.
2. Navigieren  Sie zu den `Hosted Zones` und klicken sie auf `jsr-acme.de`
3. Clicken Sie auf `Create record`
4. Setzen Sie Ihr Kürzel als Subdomäne und als Wert setzen Sie die public IP des eben erstellten Web-Servers.
5. Clicken Sie auf `Create record`
6. Versuchen Sie im Browser die Webseite aufzurufen.
    [](http://<kürzel>.jsr-acme.de/)

Sie sollten nun Ihre Seite sehen. Der Web-Server verfügt über kein Zertifikat und wird vom Browser
als nicht sicher angezeigt. Beheben wir dies noch.

### Absichern des Web-Servers mit TLS-Zertifikaten
1. Verbinden Sie sich mit dem Web-Server
2. Installieren Sie die Pakete `certbot` und `python3-certbot-nginx`.
3. Erzeugen Sie die letsencrypt Zertifikate indem Sie den Certbot starten
    `sudo certbot -d <kürzel>.jsr-acme.de --nginx --register-unsafely-without-email`
4. Testen Sie die neue Konfiguration: https://<kürzel.jsr-acme.de/>