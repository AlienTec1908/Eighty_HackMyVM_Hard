# Eighty - HackMyVM (Hard)

![Eighty.png](Eighty.png)

## Übersicht

*   **VM:** Eighty
*   **Plattform:** [HackMyVM](https://hackmyvm.eu/machines/machine.php?vm=Eighty)
*   **Schwierigkeit:** Hard
*   **Autor der VM:** DarkSpirit
*   **Datum des Writeups:** 14. Oktober 2022
*   **Original-Writeup:** https://alientec1908.github.io/Eighty_HackMyVM_Hard/
*   **Autor:** Ben C.

## Kurzbeschreibung

Das Ziel dieser "Hard"-Challenge war es, Root-Zugriff auf der Maschine "Eighty" zu erlangen. Die Enumeration deckte SSH, einen Gopher-Dienst (Port 70) und einen gefilterten HTTP-Port (80) auf. Eine Datei (`howtoconnect.txt`) auf dem Gopher-Server enthielt eine Port-Knocking-Sequenz (4767, 2343, 3142), die Port 80 öffnete. Auf dem nun zugänglichen Webserver (VHost `susan.eighty.hmv`) wurde eine Datei `lostpasswd.txt` gefunden, die das Passwort (`8ycrois-tu0`) für den Benutzer `susan` und den Pfad zu einer Google Authenticator Datei (`/home/susan/secret/.google-auth.txt`) enthielt. Durch einen (vermuteten) Path Traversal oder LFI wurde der TOTP-Secret-Key aus dieser Datei ausgelesen. Mit Passwort und TOTP-Code gelang der SSH-Login als `susan`. Als `susan` wurde eine `doas`-Regel entdeckt, die das Ausführen von `gopher` als `root` erlaubte. Durch Starten von `doas gopher` und Eingabe von `$` im Gopher-Client wurde eine Root-Shell erlangt.

## Disclaimer / Wichtiger Hinweis

Die in diesem Writeup beschriebenen Techniken und Werkzeuge dienen ausschließlich zu Bildungszwecken im Rahmen von legalen Capture-The-Flag (CTF)-Wettbewerben und Penetrationstests auf Systemen, für die eine ausdrückliche Genehmigung vorliegt. Die Anwendung dieser Methoden auf Systeme ohne Erlaubnis ist illegal. Der Autor übernimmt keine Verantwortung für missbräuchliche Verwendung der hier geteilten Informationen. Handeln Sie stets ethisch und verantwortungsbewusst.

## Verwendete Tools

*   `arp-scan`
*   `nmap`
*   `curl`
*   `knock`
*   `hping3` (alternativ zu `knock`)
*   `gobuster`
*   Google Authenticator (oder kompatible App/Tool)
*   `ssh`
*   `find`
*   `whereis`
*   `doas` (auf Zielsystem)
*   `gopher` (Client, als Exploit-Vektor)
*   Standard Linux-Befehle (`vi`, `ls`, `cat`, `cd`, `id`, `stty`, `python`)

## Lösungsweg (Zusammenfassung)

Der Angriff auf die Maschine "Eighty" gliederte sich in folgende Phasen:

1.  **Reconnaissance & Port Knocking:**
    *   IP-Findung mittels `arp-scan` (Ziel: `192.168.2.150`, Hostname `eighty.hmv`).
    *   `nmap`-Scan identifizierte SSH (22/tcp), Gopher (70/tcp) und einen gefilterten HTTP-Port (80/tcp).
    *   Auf dem Gopher-Dienst (Port 70) wurde die Datei `/howtoconnect.txt` gefunden, die die Port-Knocking-Sequenz `4767 2343 3142` enthielt.
    *   Mittels `knock 192.168.2.150 4767 2343 3142` wurde Port 80 geöffnet. Ein Nmap-Scan bestätigte dies und löste die IP zum Hostnamen `susan.eighty.hmv` auf.

2.  **Web Enumeration & Credential Discovery:**
    *   `gobuster` auf `http://susan.eighty.hmv` (Port 80) fand das Verzeichnis `/web/` und darin die Datei `lostpasswd.txt`.
    *   `lostpasswd.txt` enthielt das Passwort `8ycrois-tu0` und den Pfad `/home/susan/secret/.google-auth.txt`.
    *   Durch einen (vermuteten) Path Traversal oder LFI auf `http://susan.eighty.hmv/web../secret/.google-auth.txt` wurde der TOTP-Secret-Key (`2GN7K...`) ausgelesen.

3.  **Initial Access (als `susan` via SSH mit TOTP):**
    *   Der extrahierte TOTP-Secret-Key wurde in eine Authenticator-App importiert.
    *   Ein SSH-Login als `susan@eighty.hmv` mit dem Passwort `8ycrois-tu0` und dem aktuell generierten TOTP-Code war erfolgreich.
    *   Die User-Flag wurde im Home-Verzeichnis von `susan` gelesen.

4.  **Privilege Escalation (von `susan` zu `root` via `doas gopher`):**
    *   Als `susan` wurde mittels `find` und `cat` die `doas`-Konfigurationsdatei `/usr/local/etc/doas.conf` gefunden.
    *   Diese enthielt die Regel `permit nolog susan as root cmd gopher`, die `susan` erlaubte, `gopher` als `root` ohne Passwort auszuführen.
    *   Der Befehl `doas gopher gopher://127.0.0.1` wurde ausgeführt.
    *   Innerhalb des Gopher-Clients wurde das Zeichen `$` eingegeben und Enter gedrückt.
    *   Dies startete eine Shell als `root`. Die Root-Flag wurde gelesen.

## Wichtige Schwachstellen und Konzepte

*   **Port Knocking:** Verwendet, um den HTTP-Dienst auf Port 80 freizuschalten.
*   **Information Disclosure:**
    *   Port-Knocking-Sequenz in einer Textdatei auf dem Gopher-Server.
    *   Passwort und Pfad zur 2FA-Datei in `lostpasswd.txt` auf dem Webserver.
    *   TOTP-Secret-Key durch (vermuteten) Path Traversal / LFI.
*   **SSH mit Zwei-Faktor-Authentifizierung (TOTP):** Der Zugriff erforderte Passwort und einen zeitbasierten Code.
*   **Unsichere `doas`-Regel (`gopher`):** Das Erlauben von `gopher` als `root` ermöglichte eine einfache Privilegieneskalation durch Shell-Escape im Gopher-Client.
*   **Shell Escape in `gopher` Client:** Eingabe von `$` führte zur Ausführung einer System-Shell.

## Flags

*   **User Flag (`/home/susan/user.txt`):** `hmv8use0red`
*   **Root Flag (`/root/r0ot.txt` - Dateiname aus Log):** `rooted80shmv`

## Tags

`HackMyVM`, `Eighty`, `Hard`, `Port Knocking`, `Gopher`, `Information Disclosure`, `2FA`, `TOTP`, `SSH`, `doas`, `Privilege Escalation`, `Shell Escape`, `Linux`
