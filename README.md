# Lokaler KI-Testserver

## Dokumente · Chat · Transkription

Prototyp eines vollständig **lokalen KI-Systems** für Dokumentanalyse,
Chat und Audio-Transkription.\
Die Plattform läuft auf einem Debian-Server und nutzt **Ollama**, **Open
WebUI** und **Whisper**, um KI-Funktionen ohne Cloud-Abhängigkeit
bereitzustellen.

## Systemarchitektur

    Benutzer (Browser)
            │
            ▼
       Open WebUI (Port 3000)
            │
            ▼
          Ollama (Port 11434)
            │
            ▼
          Llama 3

Audio Pipeline: Audio → Whisper → Text → LLM Analyse

Dokumenten Pipeline: PDF Upload → Open WebUI → Llama 3 → Analyse

------------------------------------------------------------------------

# Installierte Komponenten

  Komponente         Status         Beschreibung
  ------------------ -------------- -------------------------------
  Debian Server      Aktiv          Stabiler Testserver
  Docker             Aktiv          Containerbetrieb für Services
  Ollama             Aktiv          LLM Runtime
  Llama 3            Installiert    Modell für Chat und Analyse
  Open WebUI         Aktiv          Weboberfläche
  Whisper            Konfiguriert   Speech‑to‑Text
  Whisper ASR        Optional       API Testservice
  Python PDF Tools   Installiert    PyMuPDF, pdfplumber

------------------------------------------------------------------------

# Aktuelle Funktionen

## KI‑Chat

Zugriff auf das lokale Modell **Llama 3** über Open WebUI.

## Speech‑to‑Text

Audio kann lokal mit **Whisper** transkribiert werden.

## Dokumentanalyse

Textbasierte PDFs können hochgeladen und durch das LLM analysiert
werden.

------------------------------------------------------------------------

# Projektstatus

Die Kernplattform läuft erfolgreich:

-   Lokales LLM
-   Weboberfläche
-   Speech‑to‑Text
-   PDF‑Analyse

Der Server dient aktuell als **Testumgebung für eine zukünftige
automatisierte Dokumentationspipeline**.

------------------------------------------------------------------------

# Nächste Schritte

-   Verarbeitung von Scan‑PDFs (OCR)
-   Batch‑Verarbeitung
-   Job‑Queue Architektur
-   Integration in Dokumentationsworkflows
