# Installationsanleitung – Lokale KI-Umgebung

Installationsanleitung

Lokale KI-Umgebung mit Debian, Docker, Ollama, Open WebUI und Whisper

Ziel

zu-Text mit Whisper – vollständig auf dem eigenen 

Lokale KI-Plattform für PDF-Analyse, Chat und Sprach-

Server.

1. Debian vorbereiten
System aktualisieren:

sudo apt update

sudo apt upgrade -y

2. Docker installieren
Falls Docker noch nicht installiert ist, zuerst die Paketquellen und Abhängigkeiten einrichten:

sudo apt install -y ca-certificates curl

sudo install -m 0755 -d /etc/apt/keyrings

sudo curl -fsSL https://download.docker.com/linux/debian/gpg -o /etc/apt/keyrings/docker.asc

sudo chmod a+r /etc/apt/keyrings/docker.asc

echo \

"deb [arch=$(dpkg --print-architecture) signed-by=/etc/apt/keyrings/docker.asc] 

https://download.docker.com/linux/debian \

$(. /etc/os-release && echo "$VERSION_CODENAME") stable" | \

sudo tee /etc/apt/sources.list.d/docker.list > /dev/null

sudo apt update

sudo apt install -y docker-ce docker-ce-cli containerd.io docker-buildx-plugin docker-compose-plugin

Docker aktivieren und den Benutzer zur Docker-Gruppe hinzufügen:

sudo systemctl enable docker

sudo systemctl start docker

sudo usermod -aG docker $USER

newgrp docker

docker ps

3. Ollama installieren
Ollama ist die lokale KI-Engine für eure Sprachmodelle:

curl -fsSL https://ollama.com/install.sh | sh

systemctl status ollama

ollama -v

4. Ollama im Netzwerk erreichbar machen
Damit Open WebUI im Docker-Container auf Ollama zugreifen kann, muss Ollama auf allen 

Interfaces lauschen:

sudo mkdir -p /etc/systemd/system/ollama.service.d

cat <<'EOF' | sudo tee /etc/systemd/system/ollama.service.d/override.conf

[Service]

Environment="OLLAMA_HOST=0.0.0.0:11434"

EOF

sudo systemctl daemon-reload

sudo systemctl restart ollama

ss -tulpn | grep 11434

curl http://127.0.0.1:11434/api/tags

Erwartung: In der Ausgabe sollte später „*:11434“ oder „0.0.0.0:11434“ sichtbar sein.

5. Modell in Ollama laden

ollama pull llama3

ollama list

Ein erster Funktionstest ist direkt im Terminal möglich:

ollama run llama3

6. Open WebUI installieren
Open WebUI ist die Weboberfläche für den lokalen KI-Server:

docker run -d \

  --name open-webui \

  -p 3000:8080 \

  -v open-webui:/app/backend/data \

  --restart always \

  ghcr.io/open-webui/open-webui:main

docker ps

Die Oberfläche ist danach erreichbar unter:

http://SERVER-IP:3000

Beim ersten Öffnen wird der erste Benutzer als Admin angelegt.

7. Open WebUI mit Ollama verbinden
In Open WebUI die Verbindung zu Ollama einrichten:

• Admin Panel 

→

 Einstellungen 

→

 Verbindungen

• Ollama-URL auf die Server-IP mit Port 11434 setzen, z. B.:

http://192.168.x.x:11434

oder

http://100.x.x.x:11434

Danach „Save“ und „Refresh Models“ ausführen. Anschließend sollte z. B. „llama3:latest“ im 

Webinterface auswählbar sein.

8. Whisper direkt in Open WebUI verwenden
Open WebUI bringt eigene Speech-to-Text-Funktionen mit. Für den Testbetrieb genügt die lokale 

Whisper-Konfiguration innerhalb von Open WebUI:

• Admin Panel 

→

 Settings 

→

 Audio

• Speech-to-Text Engine: Whisper

• STT Model: base

• Sprache: de

Damit werden gesprochene Eingaben lokal über Whisper transkribiert.

Hinweis zum Mikrofonzugriff
Wenn Open WebUI über eine unsichere HTTP-IP geöffnet wird, blockieren Browser den 

Mikrofonzugriff oft. Für einen sauberen Test empfiehlt sich deshalb ein SSH-Tunnel vom eigenen 

Rechner:

ssh -L 3000:localhost:3000 paul@SERVER-IP

Danach im Browser lokal öffnen:

http://localhost:3000

9. Optional: separater Whisper-ASR-Webservice
Falls zusätzlich ein eigener Whisper-Dienst benötigt wird, kann ein separater ASR-Webservice 

gestartet werden. Dieser Schritt ist optional und nicht erforderlich, wenn Whisper direkt in Open 

WebUI verwendet wird.

docker run -d \

  --name whisper \

  -p 9000:9000 \

  -e ASR_MODEL=base \

  -e ASR_ENGINE=openai_whisper \

  -v whisper-cache:/root/.cache \

  --restart always \

  onerahmet/openai-whisper-asr-webservice:latest

docker ps

docker logs -f whisper

Die API ist danach erreichbar unter:

http://SERVER-IP:9000

Beispiel für einen Transkriptionsaufruf:

curl -X POST "http://localhost:9000/asr?task=transcribe&language=de" \

  -F "audio_file=@test.mp3"

10. Python-Umgebung für PDF-Tools
Für PDF-Analyse und Text-Extraktion wird eine eigene Python-Umgebung verwendet:

sudo apt install -y python3 python3-pip python3-venv poppler-utils

mkdir -p ~/pdf-tools

cd ~/pdf-tools

python3 -m venv .venv

source .venv/bin/activate

pip install pymupdf pdfplumber requests

Funktionstest:

python -c "import fitz, pdfplumber, requests; print('OK')"

11. PDF-Analyse-Script mit Ollama
Im Verzeichnis ~/pdf-tools kann ein kleines Script angelegt werden, das Text aus einem PDF 

extrahiert und an Ollama zur Zusammenfassung sendet:

nano ~/pdf-tools/pdf_analyse.py

Beispielinhalt für das Script:

import sys

import fitz

import requests

OLLAMA_URL = "http://localhost:11434/api/generate"

MODEL = "llama3:latest"

def extract_text(pdf_path: str) -> str:

    doc = fitz.open(pdf_path)

    parts = []

    for page in doc:

        parts.append(page.get_text())

    doc.close()

    return "\n".join(parts).strip()

def summarize_text(text: str) -> str:

    prompt = f"""

Analysiere das folgende Dokument und gib strukturiert aus:

1. Kernaussagen

2. Wichtige Beobachtungen

3. Relevante Punkte für einen Bericht

4. Kurze sachliche Zusammenfassung

Dokument:

{text}

"""

    response = requests.post(

        OLLAMA_URL,

        json={

            "model": MODEL,

            "prompt": prompt,

            "stream": False,

        },

        timeout=600,

    )

    response.raise_for_status()

    return response.json()["response"]

def main() -> None:

    if len(sys.argv) != 2:

        print("Usage: python pdf_analyse.py <pdf_datei>")

        sys.exit(1)

    pdf_path = sys.argv[1]

    text = extract_text(pdf_path)

    if not text:

        print("Kein Text extrahiert. Dieses PDF enthält wahrscheinlich nur Bilder.")

        sys.exit(2)

    result = summarize_text(text)

    print(result)

if __name__ == "__main__":

    main()

Testlauf:

cd ~/pdf-tools

source .venv/bin/activate

python pdf_analyse.py test.pdf

12. PDFs direkt in Open WebUI testen
Maschinenlesbare PDFs können direkt in Open WebUI hochgeladen und analysiert werden. 

Beispielprompt:

Analysiere dieses Dokument und gib strukturiert aus:

1. Kernaussagen

2. relevante Beobachtungen

3. wichtige Punkte für einen Bericht

4. kurze Zusammenfassung

13. Ergebnis: aktueller Zielzustand
Nach dieser Installation steht lokal auf dem Server folgende Umgebung zur Verfügung:

Debian

├─

 Docker

│ ├─

 Open WebUI (Port 3000)

│ └─

 optional Whisper ASR Webservice (Port 9000)

├─

 Ollama (Port 11434)

│ └─

 llama3

└─

 Python PDF Tools

└─

 PDF 

→

 Text 

→

 Ollama Analyse

Damit lassen sich bereits folgende Anwendungsfälle testen:

• lokaler Chat mit Llama 3

• PDF-Upload und Analyse in Open WebUI

• lokale Speech-to-Text-Funktion mit Whisper

• optional separater Whisper-ASR-Dienst für Audiodateien

• PDF-Textanalyse per Python-Script mit Ollama

Hinweis: Diese Anleitung bildet den aktuellen Teststand ab und lässt Tesseract/OCR bewusst außen vor.

  
  
  
   
