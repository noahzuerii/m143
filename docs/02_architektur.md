# 02 – Architektur & Diagramme

Dieses Dokument beschreibt die Architektur des Backup- und Restore-Systems sowie die eingesetzten Komponenten.

---

## 🏗️ Übersicht der Architektur

Das System basiert auf einer **hybriden AWS-Umgebung**, die sowohl EC2- als auch RDS-Ressourcen nutzt.  
Backups werden automatisiert in **S3** gespeichert und mit **Lifecycle-Richtlinien** verwaltet.

---

## 🔧 Komponenten

1. **EC2-Instanz (t3.micro)**  
   - Betriebssystem: Amazon Linux 2023  
   - Rolle: Ausführung der Backup-Skripte und Cronjobs  
   - Verantwortlich für: Datei-Backups, Datenbank-Dumps, AMI-Erstellung  
   - Zugriff nur für Administrator:innen (SSH, IAM-Rolle)
 ![Wo meine Backup Scripts sind](./img/EC2_Aufbau.png)

2. **RDS MySQL (db.t3.micro)**  
   - Datenbank: `school`  
   - Enthält sensible Schülerdaten (Stammdaten, Noten, Absenzen)  
   - Backups: tägliche RDS-Snapshots + Dumps via EC2
<img width="2308" height="1250" alt="image" src="https://github.com/user-attachments/assets/14de0c59-5ff7-4e29-a202-e721f19fe254" />

3. **S3 Bucket**  
   - Name: `backup-raw-bachmann-pe24c`  
   - Speicherung aller Backups (Dateien, DB-Dumps)  
   - **Versionierung** aktiv  
   - **Lifecycle-Policy:**  
     - 30 Tage → Standard Storage  
     - 31–90 Tage → Glacier  
     - 90 Tage → automatische Löschung  
<img width="2066" height="812" alt="image" src="https://github.com/user-attachments/assets/5f7d67ea-1d67-444b-908e-dabf2d1a2720" />

4. **Cronjobs**  
   - Steuern den täglichen und wöchentlichen Ablauf der Skripte  
   - Logs werden in `/var/backups/logs` gespeichert  
<img width="1113" height="250" alt="image" src="https://github.com/user-attachments/assets/38091a62-43fe-409d-b718-27a16bed3546" />

5. **Mail-Benachrichtigung**  
   - SMTP-Versand über Gmail-App-Passwort  
   - Meldung von Erfolg/Fehlschlag an Administrator  
<img width="600" alt="image" src="https://github.com/user-attachments/assets/bdc16c7d-b089-499f-98f8-9a17758d9d10" />

6. **IAM & Security Groups**  
   - Prinzip des **Least Privilege**  
   - RDS ist nur für EC2 zugänglich  
   - S3-Zugriff nur über definierte IAM-Rollen

---

## 🔐 Sicherheitsaspekte

- **Verschlüsselung:**  
  - S3-Objekte mit SSE-S3  
  - RDS-Backups verschlüsselt durch AWS  
- **Netzwerk:**  
  - Kein Public Access auf RDS  
  - Nur SSH von Admin-IP auf EC2  
- **Zugriff:**  
  - Getrennte Rollen für Admins / Backups  
  - MFA für IAM-Accounts (falls verfügbar)


**Legende:**  
- Blau = AWS Ressourcen (EC2, RDS, S3)  
- Grün = Backup-Prozesse (Cronjobs, Skripte)  
- Rot = Restore-Szenarien
