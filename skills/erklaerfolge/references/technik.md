# Technik der Erklärfolge

Befehle und Bauweise für die Phasen 5 bis 10, dazu Higgsfield für Phase 7 (Abschnitt 9). Alle Werkzeuge sind öffentlich: Python 3 mit `opencv-python` und `numpy`, ffmpeg, Node.js mit Playwright, Whisper. Zielformat 1080 × 1920, 30 Bilder je Sekunde.

Installation:

```bash
python3 -m venv .venv && source .venv/bin/activate
pip install opencv-python numpy openai-whisper
npm install playwright && npx playwright install chromium
```

Die eigene Umgebung `.venv` vermeidet Konflikte mit dem Python des Systems (auf dem Mac verweigert pip die Installation sonst oft). In jeder neuen Sitzung zuerst `source .venv/bin/activate`. Bilder und Clips: Abschnitt 9.

## 1. Stimme mit Zeitstempeln

Das ganze Skript in einem Durchgang an ElevenLabs schicken, mit Anweisung am Anfang. Den Schlüssel nur als Umgebungsvariable übergeben, nie in Dateien oder ins Repository schreiben.

`anfrage.json`:

```json
{
  "text": "[soft-spoken, calm, intimate narration, low energy, steady pace] Erster Satz. Zweiter Satz. …",
  "model_id": "eleven_v4",
  "voice_settings": { "stability": 0.9, "similarity_boost": 0.8, "style": 0, "speed": 0.95 }
}
```

```bash
curl -s -X POST "https://api.elevenlabs.io/v1/text-to-speech/<STIMM-ID>/with-timestamps?output_format=mp3_44100_192" \
  -H "xi-api-key: $ELEVEN_KEY" -H "Content-Type: application/json" \
  -d @anfrage.json > antwort.json
python3 -c "import json,base64; d=json.load(open('antwort.json')); open('stimme.mp3','wb').write(base64.b64decode(d['audio_base64'])); json.dump(d['alignment'], open('zeiten.json','w'), ensure_ascii=False)"
```

`zeiten.json` enthält `characters`, `character_start_times_seconds` und `character_end_times_seconds`. Die Zeichen der Anweisung am Anfang stehen mit darin; beim Suchen der Sätze überspringen.

Ohne API: die Stimme in der ElevenLabs-Oberfläche erzeugen und die Wortzeiten danach mit Whisper messen:

```bash
whisper stimme.mp3 --model medium --language de --word_timestamps True --output_format json
```

## 2. Satzanfänge im Ton messen

Die Zeitachse der Stimme liegt oft ein paar Zehntel neben dem Ton. Schnitte danach treffen Wörter. Darum:

1. Echte Pausen finden:
   ```bash
   ffmpeg -i stimme.mp3 -af silencedetect=n=-45dB:d=0.12 -f null /dev/null 2>&1 | grep silence_
   ```
2. Für jede Satzgrenze die erwartete Stelle aus `zeiten.json` nehmen (Mitte zwischen letztem Zeichen des Satzes und erstem Zeichen des nächsten) und die nächstgelegene echte Pause im Umkreis von 1 s wählen. Gibt es keine, die leiseste Stelle im Umkreis von 0,7 s.
3. Den Sprechbeginn nach der Pause dort setzen, wo der Pegel −42 dB übersteigt.
4. Eine Zeitdatei schreiben, die alles Weitere steuert, zum Beispiel:
   ```json
   { "saetze": [ { "nr": 1, "start": 0.00, "ende": 4.61 }, { "nr": 2, "start": 4.86, "ende": 12.40 } ],
     "woerter": { "67": 35.18, "68": 52.02 } }
   ```

Die Aufnahme selbst bleibt ungeschnitten. Jeder Bildabschnitt läuft vom Anfang seines Satzes bis zum Anfang des nächsten.

## 3. Clips aufbereiten

Jeden Clip auf die Länge seines Abschnitts bringen:

```bash
ffmpeg -i clip.mp4 -vf "scale=1080:1920:flags=lanczos,fps=30,setsar=1,tpad=stop_mode=clone:stop_duration=0.3" \
  -t <dauer> -c:v libx264 -crf 14 -pix_fmt yuv420p -an abschnitt-03.mp4
```

Ist der Clip zu kurz, leicht verlangsamen und Zwischenbilder rechnen (`<faktor>` = Abschnittsdauer geteilt durch Cliplänge, höchstens etwa 1,3):

```bash
ffmpeg -i clip.mp4 -vf "setpts=<faktor>*PTS,minterpolate=fps=30:mi_mode=mci:mc_mode=aobmc:vsbmc=1,scale=1080:1920:flags=lanczos,setsar=1" \
  -t <dauer> -c:v libx264 -crf 14 -pix_fmt yuv420p -an abschnitt-07.mp4
```

Nach jedem `vstack`, `hstack` oder Skalieren eines Teilbilds `setsar=1` setzen. Sonst landet ein falsches Bildpunkt-Verhältnis in der Datei. Player zeigen das Video dann gestaucht.

## 4. Leuchtkasten

Ein Python-Skript mit OpenCV setzt das echte Material Bild für Bild in den weißen Kasten des Clips:

1. **Kasten finden:** im ersten Bild des Clips Graustufen, Schwelle bei etwa 215, größte zusammenhängende helle Fläche (`cv2.connectedComponentsWithStats`), ein paar Pixel Rand dazu. Das funktioniert, weil der Clip-Prompt eine ruhige Kamera und einen leeren weißen Kasten verlangt.
2. **Inhalt vorbereiten:** jedes Quellbild auf 9:16 zuschneiden (nicht verzerren) und auf 1080 × 1920 bringen. Der Inhalt ist immer dieses Vollbild, nur kleiner gezeichnet.
3. **Zeichnen:** Inhalt mittig in den Kasten, auf den Kasten beschnitten. Echtes Video: je Ausgabebild genau ein Quellbild, keine Mischung zweier Bilder.
4. **Zoom:** Über rund 1,2 s mit `easeInOutCubic` skaliert ein Faktor `s` das ganze Bild um die Kastenmitte; die Kastenmitte wandert dabei zur Bildmitte. Bei `s = 1920 / Kastenhöhe` ist der Inhalt genau bildfüllend; danach wächst nur noch der Kasten über den Rand, bis der Rahmen draußen ist. Ab dann das Quellbild direkt ausgeben.
   ```
   e(u) = 4u³                 für u < 0,5
   e(u) = 1 − (−2u + 2)³ / 2  sonst
   ```
5. **Ausgabe:** Bilder roh an ffmpeg übergeben (`-f rawvideo -pix_fmt bgr24 -s 1080x1920 -r 30 -i -`), kodiert als H.264.

Keine Überblendung zwischen Kasten und Vollbild: Zwei verschieden große Bilder übereinander sehen aus wie ein Fehler.

## 5. Schrift, Grafiken und Untertitel als HTML-Ebene

Jede Ebene ist eine HTML-Seite in 1080 × 1920 mit durchsichtigem Hintergrund (`html, body { margin: 0; background: transparent; }`). Die Seite zeigt nichts von selbst, sondern bietet eine Funktion `window.zeige(t)`, die den Zustand für die Sekunde `t` setzt: welcher Text sichtbar ist, wie weit ein Zähler steht, wo eine Linie endet. Die Zeitdatei als `zeiten.js` einbinden (`window.ZEITEN = {…}`), weil der Browser Dateien über `file://` nicht per `fetch` lädt.

`ebene.mjs`:

```js
// Rendert eine HTML-Ebene Bild für Bild als transparente PNGs.
// Aufruf: node ebene.mjs ebene.html <dauer-in-sekunden> <ausgabeordner>
import { chromium } from 'playwright';
import path from 'node:path';
import fs from 'node:fs';

const [html, dauer, ordner] = process.argv.slice(2);
const fps = 30;
fs.mkdirSync(ordner, { recursive: true });
const browser = await chromium.launch();
const page = await browser.newPage({ viewport: { width: 1080, height: 1920 } });
await page.goto('file://' + path.resolve(html));
await page.evaluate(() => document.fonts.ready);
const anzahl = Math.round(Number(dauer) * fps);
for (let i = 0; i < anzahl; i++) {
  await page.evaluate((t) => window.zeige(t), i / fps);
  await page.screenshot({ path: path.join(ordner, String(i).padStart(5, '0') + '.png'), omitBackground: true });
}
await browser.close();
```

Überlagern:

```bash
ffmpeg -i basis.mp4 -framerate 30 -i ebene/%05d.png \
  -filter_complex "[0][1]overlay=0:0:eof_action=pass,format=yuv420p" \
  -c:v libx264 -crf 16 -an mit-ebene.mp4
```

Hinweise:

- Abschattungen hinter einer Grafik gehören in derselben Seite **unter** die Grafik. Liegen sie darüber, wirkt alles ausgewaschen.
- Untertitel aus den Wortzeiten: Stücke bis rund 28 Zeichen, Einsatz höchstens 0,2 s nach dem Wort, weiß ohne Kasten, Zahlen in der Akzentfarbe.
- Folgt eine Grafik einem runden Objekt im Bild (z. B. dem Zifferblatt einer Uhr), die Position Bild für Bild mit `cv2.HoughCircles` messen und als Liste an die Seite geben.

## 6. Zusammensetzen

`liste.txt` mit allen Abschnitten in Reihenfolge:

```
file 'abschnitt-01.mp4'
file 'abschnitt-02.mp4'
```

```bash
ffmpeg -f concat -safe 0 -i liste.txt -c:v libx264 -crf 14 -pix_fmt yuv420p -r 30 basis.mp4
```

Danach die Ebenen aus Abschnitt 5 nacheinander überlagern.

## 7. Mischung

Stimme, Musik und Effekte in einem Durchgang. Die Musik duckt sich per Sidechain unter der Stimme; jeder Effekt bekommt Filter, Pegel und Startzeit (`adelay` in Millisekunden). Am Ende −16 LUFS.

```bash
ffmpeg -i stimme.mp3 -i musik.mp3 -i fx-zoom.mp3 -filter_complex "\
[0]highpass=f=80,loudnorm=I=-16:TP=-2:LRA=11,volume=-5dB,aresample=48000,aformat=channel_layouts=stereo,asplit=2[v][vk];\
[1]aresample=48000,aformat=channel_layouts=stereo,volume=-17dB[m0];\
[m0][vk]sidechaincompress=threshold=0.08:ratio=2.5:attack=30:release=400[m];\
[2]atrim=0:1.2,highpass=f=300,lowpass=f=7000,volume=-8dB,aresample=48000,aformat=channel_layouts=stereo,adelay=4030|4030[f1];\
[v][m][f1]amix=inputs=3:normalize=0:duration=first,loudnorm=I=-16:TP=-1.5:LRA=11[out]" \
  -map "[out]" -ar 48000 mischung.wav
ffmpeg -i mit-ebenen.mp4 -i mischung.wav -c:v copy -c:a aac -b:a 256k -shortest folge.mp4
```

- Effekte nur auf Bildereignisse, ein hörbarer Akzent je Abschnitt. Den Zeitpunkt aus dem Bild nehmen (Beginn des Zooms, sichtbarer Blitz), nicht schätzen.
- Musik, die zu früh endet, im Takt verlängern: eine Stelle suchen, an der sich die Musik wiederholt; dort anhängen.
- Abstand zwischen Stimme und Musik nachmessen und gegenhören.

## 8. Prüfen

Seitenverhältnis (muss `sample_aspect_ratio=1:1` und `display_aspect_ratio=9:16` zeigen):

```bash
ffprobe -v error -select_streams v:0 -show_entries stream=width,height,sample_aspect_ratio,display_aspect_ratio -of default=nw=1 folge.mp4
```

Lautheit (Zeile `I:` im Ergebnis, Ziel rund −16 LUFS):

```bash
ffmpeg -i folge.mp4 -af ebur128 -f null /dev/null 2>&1 | tail -12
```

Transkript gegen den Sprechtext:

```bash
whisper folge.mp4 --model medium --language de --output_format txt
```

Einzelbild an jeder Zahl:

```bash
ffmpeg -ss 35.3 -i folge.mp4 -frames:v 1 pruefbild-67.png
```

Zuletzt die Datei aufs Handy schicken und in der Vorschau der Zielplattform ansehen: Liegt nichts unter den Bedienelementen oben und unten? Sind alle Zahlen lange genug lesbar?

## 9. Startbilder und Clips bei Higgsfield

Geht in der Oberfläche auf [higgsfield.ai](https://higgsfield.ai) oder mit der Kommandozeile von Higgsfield, die dieselben Credits nutzt:

```bash
npm i -g @higgsfield/cli
higgsfield auth login
higgsfield account status
```

Vor jedem Auftrag die Kosten abfragen: derselbe Befehl mit `cost` statt `create`, mit genau denselben Werten (Auflösung und Qualität ändern den Preis). Die Kosten dem Nutzer nennen, erst nach seinem Go `create`.

Startbild im Hochformat:

```bash
higgsfield generate cost gpt_image_2_5 --prompt "<Figur-Baustein>, <Szene>" --aspect_ratio 9:16
higgsfield generate create gpt_image_2_5 --prompt "<Figur-Baustein>, <Szene>" --aspect_ratio 9:16 --wait
```

Clip aus dem Startbild, 5 s, 720p, ohne Ton:

```bash
higgsfield generate cost wan3_0_prime --prompt "<Bewegung und Kamera>. Premium 3D look exactly as in the start image, the mannequins stay faceless and identical. No text, no logos." --start-image ./start-03.png --duration 5 --resolution 720p --aspect_ratio 9:16 --generate_audio false
higgsfield generate create wan3_0_prime --prompt "<gleicher Prompt>" --start-image ./start-03.png --duration 5 --resolution 720p --aspect_ratio 9:16 --generate_audio false --wait
```

- `--wait` wartet auf das Ergebnis und gibt die Adresse der Datei aus. Bricht das Warten ab, läuft der Auftrag weiter und ist bezahlt: nicht neu starten, sondern mit `higgsfield generate list` die ID suchen und `higgsfield generate wait <id>` aufrufen.
- Modellnamen und Werte ändern sich: `higgsfield model list` zeigt die aktuellen Modelle, `higgsfield model get <name>` ihre Werte.
- `--generate_audio false` schaltet den Ton des Clips ab; Stimme und Musik kommen aus den Abschnitten 1 und 7.
