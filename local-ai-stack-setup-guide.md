# 🧠 Lokale KI-Plattform

![Debian](https://img.shields.io/badge/Debian-Server-red)
![Docker](https://img.shields.io/badge/Docker-Container-blue)
![Ollama](https://img.shields.io/badge/Ollama-LLM-green)
![Open WebUI](https://img.shields.io/badge/Open%20WebUI-Interface-orange)
![Whisper](https://img.shields.io/badge/Whisper-Speech%20to%20Text-purple)

Lokale KI-Umgebung für:

* 💬 **LLM Chat**
* 📄 **PDF Analyse**
* 🎤 **Speech-to-Text**
* 🔒 **100 % lokal – keine Cloud**

Die Plattform basiert auf:

* **Debian**
* **Docker**
* **Ollama**
* **Open WebUI**
* **Whisper**
* **Python Tools**

---

# 📑 Inhaltsverzeichnis

* [Architektur](#-architektur)
* [Voraussetzungen](#-voraussetzungen)
* [Installation](#-installation)
* [Ollama konfigurieren](#-ollama-konfigurieren)
* [Modelle installieren](#-modelle-installieren)
* [Open WebUI installieren](#-open-webui-installieren)
* [Whisper Speech-to-Text](#-whisper-speech-to-text)
* [Optionaler Whisper ASR Server](#-optionaler-whisper-asr-server)
* [Python PDF Tools](#-python-pdf-tools)
* [PDF Analyse Script](#-pdf-analyse-script)
* [Test der Umgebung](#-test-der-umgebung)
* [Use Cases](#-use-cases)

---

# 🏗 Architektur

```
Debian Server
│
├── Docker
│   ├── Open WebUI (Port 3000)
│   └── Whisper ASR Server (optional, Port 9000)
│
├── Ollama (Port 11434)
│   └── LLM Modelle (z.B. llama3)
│
└── Python Tools
    └── PDF → Text → Ollama Analyse
```

---

# ⚙ Voraussetzungen

Server:

* Debian 11 / 12
* mindestens **8 GB RAM** (empfohlen 16 GB)
* mindestens **20 GB Speicher**
* Internetzugang für Modell-Downloads

Tools:

* Docker
* Python 3
* Curl

---

# 🚀 Installation

## System aktualisieren

```bash
sudo apt update
sudo apt upgrade -y
```

---

# 🐳 Docker installieren

## Abhängigkeiten

```bash
sudo apt install -y ca-certificates curl
```

## Repository hinzufügen

```bash
sudo install -m 0755 -d /etc/apt/keyrings
sudo curl -fsSL https://download.docker.com/linux/debian/gpg -o /etc/apt/keyrings/docker.asc
sudo chmod a+r /etc/apt/keyrings/docker.asc
```

```bash
echo \
"deb [arch=$(dpkg --print-architecture) signed-by=/etc/apt/keyrings/docker.asc] \
https://download.docker.com/linux/debian \
$(. /etc/os-release && echo "$VERSION_CODENAME") stable" | \
sudo tee /etc/apt/sources.list.d/docker.list > /dev/null
```

## Docker installieren

```bash
sudo apt update
sudo apt install -y docker-ce docker-ce-cli containerd.io docker-buildx-plugin docker-compose-plugin
```

## Docker starten

```bash
sudo systemctl enable docker
sudo systemctl start docker
```

## Benutzer hinzufügen

```bash
sudo usermod -aG docker $USER
newgrp docker
```

Test:

```bash
docker ps
```

---

# 🤖 Ollama installieren

Ollama stellt die **lokalen Sprachmodelle** bereit.

```bash
curl -fsSL https://ollama.com/install.sh | sh
```

Status prüfen:

```bash
systemctl status ollama
```

Version prüfen:

```bash
ollama -v
```

---

# 🌐 Ollama im Netzwerk erreichbar machen

Open WebUI läuft im Docker-Container und muss auf Ollama zugreifen können.

```bash
sudo mkdir -p /etc/systemd/system/ollama.service.d
```

```bash
cat <<'EOF' | sudo tee /etc/systemd/system/ollama.service.d/override.conf
[Service]
Environment="OLLAMA_HOST=0.0.0.0:11434"
EOF
```

Service neu starten:

```bash
sudo systemctl daemon-reload
sudo systemctl restart ollama
```

Port prüfen:

```bash
ss -tulpn | grep 11434
```

---

# 📦 Modelle installieren

Beispiel: **Llama 3**

```bash
ollama pull llama3
```

Modelle anzeigen:

```bash
ollama list
```

Test:

```bash
ollama run llama3
```

---

# 🌐 Open WebUI installieren

Weboberfläche für die lokale KI.

```bash
docker run -d \
  --name open-webui \
  -p 3000:8080 \
  -v open-webui:/app/backend/data \
  --restart always \
  ghcr.io/open-webui/open-webui:main
```

Container prüfen:

```bash
docker ps
```

Zugriff:

```
http://SERVER-IP:3000
```

Beim ersten Start wird automatisch ein **Admin Benutzer** erstellt.

---

# 🔗 Open WebUI mit Ollama verbinden

In der WebUI:

```
Admin Panel
→ Settings
→ Connections
```

Ollama URL:

```
http://SERVER-IP:11434
```

Dann:

```
Save
Refresh Models
```

---

# 🎤 Whisper Speech-to-Text

Open WebUI besitzt integriertes **Speech-to-Text**.

Einstellungen:

```
Admin Panel
→ Settings
→ Audio
```

Konfiguration:

```
Speech-to-Text Engine: Whisper
STT Model: base
Language: de
```

---

# 🎧 Optionaler Whisper ASR Server

Separater Transkriptionsserver.

```bash
docker run -d \
  --name whisper \
  -p 9000:9000 \
  -e ASR_MODEL=base \
  -e ASR_ENGINE=openai_whisper \
  -v whisper-cache:/root/.cache \
  --restart always \
  onerahmet/openai-whisper-asr-webservice:latest
```

API erreichbar unter:

```
http://SERVER-IP:9000
```

Beispiel:

```bash
curl -X POST "http://localhost:9000/asr?task=transcribe&language=de" \
-F "audio_file=@test.mp3"
```

---

# 🐍 Python PDF Tools

Pakete installieren:

```bash
sudo apt install -y python3 python3-pip python3-venv poppler-utils
```

Arbeitsverzeichnis:

```bash
mkdir -p ~/pdf-tools
cd ~/pdf-tools
```

Virtuelle Umgebung:

```bash
python3 -m venv .venv
source .venv/bin/activate
```

Pakete installieren:

```bash
pip install pymupdf pdfplumber requests
```

Test:

```bash
python -c "import fitz, pdfplumber, requests; print('OK')"
```

---

# 📄 PDF Analyse Script

Script erstellen:

```
nano ~/pdf-tools/pdf_analyse.py
```

Das Script:

* extrahiert Text aus PDF
* sendet Text an **Ollama**
* erstellt eine strukturierte Zusammenfassung

Start:

```bash
python pdf_analyse.py dokument.pdf
```

---

# 🧪 Test der Umgebung

Teste folgende Funktionen:

### Chat mit LLM

```
Open WebUI → Chat → llama3
```

### PDF Analyse

PDF hochladen und prompt eingeben:

```
Analysiere dieses Dokument und gib strukturiert aus:

1. Kernaussagen
2. Beobachtungen
3. Wichtige Punkte
4. Zusammenfassung
```

### Speech-to-Text

* Mikrofon aktivieren
* Spracheingabe testen

---

# 💡 Use Cases

Diese Umgebung ermöglicht:

* lokale **KI-Chats**
* **PDF Analyse**
* **Sprachtranskription**
* **Audio-Analyse**
* **Dokumentzusammenfassungen**
* **vollständig private KI Nutzung**

---

# 🔒 Datenschutz

Alle Komponenten laufen **lokal auf dem eigenen Server**.

Es werden **keine Daten an externe Cloud-Dienste übertragen**.

---

# 📌 Hinweis

Diese Anleitung beschreibt den aktuellen **Teststand der Plattform**.

OCR (Tesseract) ist **bewusst noch nicht integriert**.
