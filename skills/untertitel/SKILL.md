---
name: untertitel
description: Erstellt englische Untertitel als SRT-Datei für Videos in beliebiger Sprache, etwa Deutsch oder Französisch. Transkribiert und übersetzt mit OpenAI Whisper lokal auf dem eigenen Rechner, ohne Upload, ohne API-Schlüssel und ohne laufende Kosten. Brennt die Untertitel auf Wunsch per ffmpeg ins Video. Verwenden, wenn jemand Untertitel erstellen oder übersetzen, eine SRT-Datei erzeugen, Filmsequenzen untertiteln oder Aufnahmen transkribieren möchte, etwa bei „ich brauche englische Untertitel“, „kannst du das untertiteln“ oder „Whisper“.
---

# Untertitel

Du erstellst englische Untertitel für Videos in beliebiger Sprache. Die Kerntechnik ist **OpenAI Whisper**, das lokal auf dem Rechner des Nutzers läuft: kein Upload, kein API-Schlüssel, keine laufenden Kosten. Ergebnis ist eine `.srt`-Datei, die sich direkt in Premiere, Final Cut, DaVinci Resolve oder jedes andere Schnittprogramm laden lässt.

## Phase 1: Installation prüfen

```bash
whisper --help
```

Fehlt der Befehl, prüfen, ob Python da ist:

```bash
python3 --version
```

Fehlt Python: den Nutzer bitten, ein aktuelles Python 3 von python.org zu installieren und dann zurückzukommen.

Ist Python da, Whisper installieren. Auf dem Mac am einfachsten mit Homebrew:

```bash
brew install openai-whisper
```

Ohne Homebrew sowie auf Windows und Linux:

```bash
pip install -U openai-whisper
```

Wird `pip` nicht gefunden, `pip3 install -U openai-whisper` versuchen. Meldet pip „externally-managed-environment“, mit `pipx install openai-whisper` installieren (pipx gibt es per `brew install pipx` oder über die Paketverwaltung).

Whisper braucht ffmpeg für den Ton:

```bash
ffmpeg -version
```

Fehlt ffmpeg:

- Mac: `brew install ffmpeg`
- Windows: `winget install ffmpeg`

Danach noch einmal `whisper --help` ausführen.

## Phase 2: Eingabe

Frage den Nutzer:

> Welche Videodatei(en) soll ich untertiteln? Bitte gib den vollständigen Pfad an, bei mehreren Dateien alle Pfade.

Danach:

> In welcher Sprache ist der Ton? (z. B. Deutsch, Französisch, Italienisch, Spanisch, oder „automatisch erkennen“, wenn du unsicher bist)

Sprachen als Whisper-Codes:

- Deutsch: `de`
- Französisch: `fr`
- Italienisch: `it`
- Spanisch: `es`
- Portugiesisch: `pt`
- Niederländisch: `nl`
- Polnisch: `pl`
- Russisch: `ru`
- Automatisch erkennen: Option `--language` weglassen

## Phase 3: Untertitel erzeugen

Für jede Datei:

```bash
whisper "<pfad-zur-datei>" --model medium --language <sprachcode> --task translate --output_format srt --output_dir "<ausgabe-ordner>"
```

- `--task translate` lässt Whisper direkt ins Englische übersetzen, statt nur abzuschreiben.
- `--model` immer angeben. Das Standardmodell `turbo` kann nicht übersetzen und liefert trotz `--task translate` den Text in der Originalsprache.
- Als `--output_dir` den Ordner des Videos nehmen, dann liegt die SRT-Datei direkt daneben.

Beispiel für ein deutsches Video auf dem Schreibtisch:

```bash
whisper "$HOME/Desktop/interview.mp4" --model medium --language de --task translate --output_format srt --output_dir "$HOME/Desktop/"
```

Mit automatischer Spracherkennung:

```bash
whisper "<pfad>" --model medium --task translate --output_format srt --output_dir "<ausgabe-ordner>"
```

Will der Nutzer keine Übersetzung, sondern Untertitel in der Originalsprache: `--task transcribe` statt `--task translate`.

## Phase 4: Modell wählen

| Modell | Qualität | Tempo | Speicherbedarf laut Whisper |
|---|---|---|---|
| `small` | gut | schnell | rund 2 GB |
| `medium` | sehr gut (Standard für Übersetzungen) | mittel | rund 5 GB |
| `large` | am besten | langsam | rund 10 GB |
| `turbo` | gut, aber **keine Übersetzung** | schnell | rund 6 GB |

Auf älteren Rechnern `--model small`. Für Festivals, Kinoeinreichungen oder professionelle Veröffentlichungen `--model large` empfehlen.

## Phase 5: Einbrennen (optional)

Sobald die SRT-Datei fertig ist, **immer** diesen Hinweis ausgeben:

> Die SRT-Datei ist fertig. Du kannst sie so in dein Schnittprogramm laden, oder ich brenne die Untertitel direkt ins Video ein. Dann stellst du alles selbst ein:
>
> - **Schriftgröße**
> - **Schriftfarbe** (Standard Weiß, oder Gelb für den klassischen Doku-Look)
> - **Outline** (schwarzer Rand um den Text, verbessert die Lesbarkeit auf hellem und dunklem Grund)
> - **Position** (Standard unten Mitte, oder oben, wenn unten Grafiken liegen)
> - **Schriftart** (Standard Arial)
>
> Was möchtest du?
> 1. SRT importieren (ich erkläre, wie das in Premiere, Final Cut oder DaVinci geht)
> 2. Untertitel einbrennen, mit deinen Einstellungen

Wählt der Nutzer 1: die Anleitung für sein Schnittprogramm ausgeben (siehe Phase 6). Wählt er 2: nach den Einstellungen fragen und dann so vorgehen.

**ffmpeg mit libass prüfen.** Einbrennen braucht den Filter `subtitles`:

```bash
ffmpeg -hide_banner -filters | grep subtitles
```

Fehlt er (das Homebrew-Paket `ffmpeg` bringt ihn nicht mit), auf dem Mac `brew install ffmpeg-full` installieren. Das Paket ersetzt das normale ffmpeg nicht, darum mit vollem Pfad aufrufen: `"$(brew --prefix ffmpeg-full)/bin/ffmpeg"`.

**Bezugsraster auf Videogröße setzen.** Werte in `force_style` beziehen sich bei SRT-Dateien nicht auf Bildpunkte des Videos, sondern auf ein Raster von 384 × 288. Damit Schriftgröße und Abstand in echten Bildpunkten gelten:

1. Breite und Höhe des Videos lesen:
   ```bash
   ffprobe -v error -select_streams v:0 -show_entries stream=width,height -of csv=p=0 "<eingabedatei>"
   ```
2. Die SRT-Datei in ASS umwandeln:
   ```bash
   ffmpeg -i "<untertitel>.srt" "<untertitel>.ass"
   ```
3. In der ASS-Datei die Zeilen `PlayResX: 384` und `PlayResY: 288` auf Breite und Höhe des Videos ändern.
4. Einbrennen (Beispiel 1920 × 1080):
   ```bash
   ffmpeg -i "<eingabedatei>" \
     -vf "subtitles='<untertitel>.ass':force_style='FontName=Arial,FontSize=54,PrimaryColour=&HFFFFFF&,OutlineColour=&H000000&,Outline=3,Alignment=2,MarginV=60'" \
     -c:a copy "<ausgabedatei-mit-untertiteln>.mp4"
   ```

Startwerte in Bildpunkten:

| Format | FontSize | Outline | MarginV |
|---|---|---|---|
| Querformat 1920 × 1080 | 54 | 3 | 60 |
| Hochformat 1080 × 1920 (Reels, Shorts) | 64 | 4 | 460, damit der Text über den Bedienelementen der Apps steht |
| Story 1080 × 1920 | 64 | 4 | 260 |

`Alignment`: `2` unten Mitte (Standard), `8` oben Mitte, `5` Bildmitte. Farben stehen im Format `&HBBGGRR&`: Weiß `&HFFFFFF&`, Gelb für den Doku-Look `&H00FFFF&`.

Nach dem Einbrennen ein Standbild ziehen und ansehen, bevor du fertig meldest:

```bash
ffmpeg -ss 5 -i "<ausgabedatei-mit-untertiteln>.mp4" -frames:v 1 pruefbild.png
```

Beim Einbrennen wird das Video neu gerendert. Das Original bleibt unberührt.

## Phase 6: Abschlussbericht

```
Fertig. Untertitel erstellt:

  interview.srt  →  ~/Desktop/interview.srt

Direkt importierbar in Premiere Pro, Final Cut Pro, DaVinci Resolve und CapCut.

In Premiere: Sequenz → Beschriftungen → Untertitel aus Datei importieren → SRT auswählen.
In Final Cut: Ablage → Importieren → Untertitel → SRT-Datei wählen.
In DaVinci Resolve: SRT in den Media Pool importieren und auf die Timeline ziehen.
```

Wurde eine eingebrannte Fassung erstellt, auch deren Pfad nennen. Bei mehreren Dateien alle Ausgabepfade auflisten.

## Häufige Fehler

- **„externally-managed-environment“** bei `pip install`: siehe Phase 1 (`brew install openai-whisper` oder `pipx install openai-whisper`).
- **„No module named whisper“** nach der Installation: Terminal neu starten und noch einmal versuchen.
- **Warnung zu CUDA oder FP16:** kann ignoriert werden. Whisper läuft dann auf dem Prozessor und braucht etwas länger.
- **Sehr langsam auf älteren Rechnern:** `--model small`.
- **Text bleibt in der Originalsprache:** `--model` fehlt oder steht auf `turbo`. `--model medium` setzen.
- **Schlechte Übersetzung bei Fachbegriffen:** `--model large`.
- **Fehler „No such filter: subtitles“:** ffmpeg ohne libass, siehe Phase 5.
