# Paperless-ngx – selbst betriebene Dokumentenarchivierung

## Zweck
Archiv für Rechnungen und Dokumente mit OCR und Volltextsuche.
Ziel der n8n-Rechnungsstrecke (Mail → PDF-Anhang → Paperless-API).

## Aufbau
- webserver: Paperless-ngx (UI, REST-API, OCR via Tesseract/OCRmyPDF)
- db: PostgreSQL 17
- broker: Redis (Task-Queue)
- tika + gotenberg: Office-Dateien und .eml
Web-UI nur an 127.0.0.1 gebunden.

## Start
    cp .env.example .env
    mkdir -p consume export
    docker compose up -d
    docker compose exec webserver python3 manage.py createsuperuser

## Stolperstellen
- macOS: Homebrew-Cask docker-desktop installiert, App aber nie gestartet → CLI-Plugins
  (u. a. compose) waren tote Symlinks. Lösung: Docker Desktop einmal starten.
- `PAPERLESS_OCR_MODE=skip` ist ab v3 ungültig (Warnung paperless.W003) → `auto`.
  Konsequenz: Image-Version gepinnt (3.5.1) statt `latest`.

## Backup
    docker compose exec -T webserver document_exporter ../export

## API-Zugang für n8n
Eigener Benutzer `n8n` ohne Admin-Rechte. Minimalrechte: Dokument anzeigen/hinzufügen;
Tags, Korrespondenten, Dokumenttypen anzeigen; Tasks anzeigen.
Token via `manage.py drf_create_token n8n`, gespeichert im Passwortmanager, nicht im Repo.
Upload ist asynchron: `post_document` liefert nur eine Task-ID, Erfolg erst über `/api/tasks/`.

## Weitere Stolperstellen
- API-Benutzer `n8n` bekam beim Abfragen von `/api/tasks/` HTTP 403. Ohne dieses Recht kann
  n8n nicht prüfen, ob ein Upload erfolgreich war. Lösung: zusätzlich „Task: anzeigen“.
- **Duplikate werden in v3.5.1 angenommen.** Test: dieselbe PDF erneut über die API
  hochgeladen → Task `success`, neues Dokument. Die Annahme aus älteren Versionen
  (Ablehnung per Prüfsumme) gilt nicht mehr. Konsequenz: Deduplizierung vollständig
  in n8n (Message-ID + SHA-256 des Anhangs, Unique-Constraint in Postgres).
  Zusätzlich sieht der n8n-Benutzer fremde Dokumente nicht, eine Prüfung über die
  Paperless-API wäre also ohnehin lückenhaft.
- n8n (eigener Stack, im Docker-Netz `paperless_default`) bekam HTTP 400: Django lässt nur
  Hostnamen aus `ALLOWED_HOSTS` zu, abgeleitet aus `PAPERLESS_URL` (nur `localhost`).
  Lösung: `PAPERLESS_ALLOWED_HOSTS=localhost,webserver`.
