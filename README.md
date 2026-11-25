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
