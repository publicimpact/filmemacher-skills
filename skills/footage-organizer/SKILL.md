---
name: footage-organizer
description: Sortiert und analysiert rohe Videoaufnahmen für den Schnitt. Nimmt einen unsortierten Footage-Ordner (MP4, MOV, MXF, BRAW, R3D usw.), zieht per ffmpeg Standbilder aus jedem Clip, wertet sie visuell aus und sortiert die Clips in beschriftete Ordner nach Location und Setup. Benennt Dateien nur nach Zustimmung um, räumt auf und liefert eine schnittfertige Struktur für jedes Schnittprogramm. Verwenden, wenn jemand Footage sortieren, Drehmaterial organisieren, Clips vorbereiten, Rushes strukturieren oder Dailies ordnen möchte, etwa bei „ich habe 40 Clips vom Dreh“ oder „kannst du mein Material sortieren“.
---

# Footage Organizer

Du nimmst einen unsortierten Ordner mit Videoaufnahmen und machst daraus eine schnittfertige Projektstruktur. Die Kerntechnik ist **visuelle Analyse**: Du ziehst Standbilder aus jedem Clip und wertest sie aus, um den Inhalt zu verstehen. Dazu kommen die technischen Metadaten (Auflösung, Bildrate, Kamera, Codec, Dauer) und, falls vorhanden, Skript-Dokumente.

Voraussetzung: ffmpeg mit ffprobe (Mac: `brew install ffmpeg`, Windows: `winget install ffmpeg`, Linux: Paketverwaltung).

## Phase 1: Bestandsaufnahme und Briefing

1. Zugriff auf den Footage-Ordner sicherstellen.
2. Alle Videodateien auflisten und zählen. Formate: `.mp4`, `.mov`, `.mxf`, `.mkv`, `.avi`, `.braw`, `.r3d`, `.ari`, `.dng` (Bildsequenzen), `.m4v`, `.webm`.
3. Für jede Datei die Metadaten per `ffprobe` lesen:
   ```bash
   ffprobe -v quiet -print_format json -show_streams -show_format "<clip>"
   ```
   Relevant: Auflösung, Bildrate, Codec, Dauer, Dateigröße, Erstellungsdatum, Kameramodell (aus den Metadaten).
4. Kameraquellen anhand von Auflösung und Codec erkennen:
   - 4K/6K ProRes oder BRAW: Blackmagic
   - R3D: RED
   - MXF mit ARRI-Tags: ARRI Alexa
   - 4K H.264/HEVC: Sony, Canon, Fuji, Drohne (DJI), iPhone
   - Hochformat (z. B. 1080 × 1920 oder 2160 × 3840): Smartphone
5. Zusammenfassung ausgeben: Anzahl Clips, erkannte Kameras, Gesamtdauer, Gesamtgröße.
6. Fragen, ob Skript-Dokumente vorliegen (Drehbuch, Treatment, Shotlist als .docx, .pdf oder .txt). Falls ja, vor Phase 2 einlesen.
7. Fragen, ob eine bestimmte Ordnerstruktur gewünscht ist oder die Standardstruktur passt.

## Phase 2: Visuelle Analyse per Standbild

Du kannst keine Videos abspielen, aber Standbilder ziehen und ansehen.

1. Einen temporären Ordner für Standbilder **außerhalb** des Footage-Ordners anlegen.
2. Für jeden Clip ein JPEG bei 30 % der Laufzeit ziehen:
   ```bash
   ffmpeg -y -ss <zeitpunkt> -i "<clip>" -vframes 1 -vf scale=640:-1 -q:v 3 "<ausgabe.jpg>"
   ```
   Bei Clips unter 3 Sekunden: bei 0,5 s.
3. Die Standbilder in Gruppen mit dem Lese-Werkzeug ansehen (es zeigt Bilder an).
4. Für jeden Clip festhalten:
   - Personen im Bild (Sprecher, Interviewer, niemand)
   - Handlung (Kameragespräch, Interview, Action, Produktdetail, Atmosphäre, Establishing)
   - Location (innen oder außen, welcher Raum, welche Umgebung)
   - Kleidung (wichtig für die Gruppierung bei mehreren Looks)
   - Sichtbare Requisiten, Logos, Text, Technik
   - Einstellungsgröße (Totale, Halbtotale, Nahe, Close-up, Over-the-Shoulder)
5. Bei unklaren Clips zusätzliche Bilder bei 10 %, 50 % und 90 % ziehen.

## Phase 3: Abgleich mit dem Skript (optional)

Falls Skript-Dokumente vorliegen:

1. Einlesen per `pandoc` (für .docx) oder mit dem Lese-Werkzeug (für .txt, .md, .pdf).
2. Die Szenen des Skripts anhand der Bilder den Clip-Gruppen zuordnen.
3. Die Bezeichnungen aus dem Skript für Ordner- und Dateinamen übernehmen, damit alles zur Produktion passt.

## Phase 4: Sortieren nach Inhalt

### Kategorien aus dem Material ableiten, nicht vorher festlegen

Die nützlichste Gruppierung ist nach **Location und Setup** (wo die Kamera stand, welches Outfit die Person trägt), weil Cutter so denken.

Typische Gruppen:

- Kameragespräche an verschiedenen Orten (alle Takes eines Setups zusammen)
- Walk-and-Talk und Bewegungsaufnahmen
- Establishing- und Atmosphäre-Shots
- Produkt- und Detailaufnahmen
- Interviews und Testimonials
- Behind the Scenes
- Luftaufnahmen (Drohne)
- B-Roll (Hände, Close-ups, Details)

### Flache Ordnerstruktur mit Nummern

Keine Verschachtelung. Flach ist besser für den Schnitt.

```
01 - [Location/Setup] - [Outfit, falls relevant]
02 - [Location/Setup] - [Outfit, falls relevant]
03 - [Walking Shots - Location]
...
```

LUT-Dateien (`.cube`) nie löschen, sondern in den Unterordner `LUTs/` verschieben.

### Erst den Plan zeigen, dann verschieben

Verschieben bricht wie Umbenennen die Verknüpfungen in einem Schnittprojekt. Darum vor dem ersten Verschieben:

1. Den Plan als Tabelle zeigen: Ordnername, Clips, Anzahl.
2. Fragen: **„Sind die Clips schon in einem Schnittprojekt verknüpft (Premiere, Final Cut, Resolve, Avid)?“** Falls ja, nichts verschieben, sondern den Plan als `sortierung.txt` in den Footage-Ordner legen.
3. Fragen, ob es eine Sicherung des Ordners gibt. Falls nein, zuerst eine Sicherung empfehlen.
4. Erst nach einem ausdrücklichen Ja verschieben.

## Phase 5: Dateien umbenennen (nur mit ausdrücklicher Zustimmung)

**Niemals Dateien umbenennen ohne ausdrückliche Zustimmung des Nutzers.**

Umbenennen kann bestehende Verknüpfungen zerstören, etwa wenn Clips schon in einem Schnittprojekt, einer EDL oder einer XML-Schnittliste stehen. Einmal umbenannt, sind diese Verknüpfungen kaputt.

Vorher fragen:

1. **„Soll ich die Dateien umbenennen?“** Standard ist Nein. Nur bei einem ausdrücklichen Ja weitermachen.
2. **„Sind die Clips schon in einem Schnittprojekt verknüpft (Premiere, Final Cut, Resolve, Avid)?“** Falls ja, dringend vom Umbenennen abraten.
3. **„Hast du eine Sicherung des Ordners?“** Falls nein, zuerst eine Sicherung empfehlen.

Erst wenn alle drei Fragen geklärt sind und der Nutzer das Umbenennen ausdrücklich will:

```
01 - Beschreibender Name.MP4
02 - Beschreibender Name.MP4
```

- fortlaufend ab 1 in jedem Ordner
- erster Buchstabe groß
- höchstens rund 60 Zeichen
- beschreiben, was passiert: „Hook Takes (3 Varianten)“, „Walk Through Showroom“, „Produktdetail Nahaufnahme“, „Fehlstart (2 s)“
- Sammelclips mit Dauer in Klammern, z. B. `02 - Interview Takes (8 min).MXF`
- sehr kurze Clips (unter 3 s) mit Dauer, damit der Cutter weiß, was ihn erwartet

## Phase 6: Aufräumen

Ohne Rückfrage:

1. `.DS_Store` entfernen
2. den Ordner mit den Standbildern aus Phase 2 löschen
3. leere Ordner entfernen
4. LUT-Dateien (`.cube`) in `LUTs/` verschieben, falls sie im Hauptordner liegen

Nur verschieben, nicht löschen:

5. **mögliche Duplikate** (Dateien mit „copy“ im Namen oder gleicher Größe und ähnlichem Namen) nach `_aussortiert/duplikate/`
6. **XML-Begleitdateien der Kamera** nach `_aussortiert/kamera-xml/` (manche Schnittprogramme lesen daraus Timecode oder Farbprofil)

Am Ende die Liste von `_aussortiert/` zeigen. Endgültig gelöscht wird nur, was der Nutzer ausdrücklich freigibt.

## Phase 7: Prüfen und Abschlussbericht

1. Clips in allen Ordnern zählen. Die Summe muss mit dem Original übereinstimmen (die Dateien in `_aussortiert/` mitgezählt).
2. Prüfen, dass im Hauptordner keine Clips mehr ungruppiert liegen.
3. Zusammenfassung ausgeben:

```
Fertig. Struktur:

01 - Kameragespräch Wohnzimmer - Look 1    → 12 Clips
02 - Kameragespräch Küche - Look 2         →  8 Clips
03 - Walk and Talk Garten                  →  5 Clips
04 - Produktdetails Close-up               →  9 Clips
05 - Establishing Exterior                 →  3 Clips
_aussortiert/duplikate                     →  3 Clips

Gesamt: 40 Clips, davon 3 mögliche Duplikate zur Prüfung.
Schnittfertig für dein Schnittprogramm.
```
