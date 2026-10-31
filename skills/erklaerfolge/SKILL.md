---
name: erklaerfolge
description: Baut mit dem Nutzer ein kurzes Erklärvideo im Hochformat (60 bis 90 s) nach dem Prinzip eine Frage, ein echter Beweis, eine Erkenntnis. Eine Figur ohne Gesicht erklärt, echtes Material läuft in einem Leuchtkasten im Bild, die Stimme entsteht in einem Durchgang und das Bild folgt ihr. Arbeitet in Phasen mit Freigaben und nennt vor jedem bezahlten Schritt die Kosten. Verwenden bei Erklärvideo, Erklärfolge, Erklärserie, Explainer, „bau mir ein Video, das erklärt, wie …“ oder wenn jemand ein Thema mit echtem Material und KI-Bildern erklären will.
compatibility: Für Claude Code oder ähnliche Coding-Agenten mit Zugriff auf den Rechner. Braucht Python 3 mit OpenCV und NumPy, ffmpeg, Node.js mit Playwright, Whisper sowie Konten bei Higgsfield und ElevenLabs.
---

# Erklärfolge

Du baust mit dem Nutzer ein kurzes Erklärvideo im Hochformat. Vorbild ist die Folge „Macht Höhe fitter?“; wie sie entstand, steht im Repository [publicimpact/making-of-hoehe](https://github.com/publicimpact/making-of-hoehe), samt aller Prompts. **Übernimm das Prinzip, nicht das Beispiel:** Thema, Figur, Farben, Takte und Texte entstehen für das Thema des Nutzers neu.

## Grundregeln

- **In Phasen arbeiten.** Nach jeder Phase das Ergebnis zeigen und auf das Go des Nutzers warten.
- **Kosten vorher nennen.** Vor jedem Schritt, der Geld oder Credits kostet, Modell, Anzahl und Kosten nennen und auf ein Go warten. Preise ändern sich; vor dem ersten Auftrag beim Anbieter nachsehen.
- **Prinzip:** Eine Frage, ein echter Beweis, eine Erkenntnis. Echt stellt die Frage, die Figur erklärt, echt beweist.
- **Jede Zahl braucht eine Quelle:** Studie, offizielle Seite oder Originalaufnahme mit Zeitstempel. Was keine Quelle hat, kommt nicht ins Bild.
- **Echtes Material nur, wenn der Nutzer es zeigen darf.** Gibt es keine Aufnahmen, nimm echte Quellen: Dokumente, Screenshots, Messwerte.
- **Nichts erfinden, was man prüfen kann.** Gerät, Ort, Person nur nennen, wenn die Quelle es belegt.

## Phase 1: Fragen klären

Frag den Nutzer zuerst:

1. Welche Frage beantwortet das Video?
2. Welches echte Material gibt es? Darf er es zeigen?
3. Welche Stimme: sein Stimmklon oder eine Profistimme?
4. Welcher Look für die Figur, welche Akzentfarbe?
5. Wie lang, für welche Plattform?

## Phase 2: Recherche

Eine Quellenliste mit jeder Zahl, die ins Video soll: Wert, Quelle, Fundstelle (Seite, PMID, Zeitstempel), Prüfdatum. Bei Videomaterial die Stelle selbst ansehen und den Zeitstempel notieren. Zahl in Stimme und Bild gleich runden.

## Phase 3: Look und Figur

1. Zwei bis drei Looks als Standbild derselben Szene nebeneinander zeigen, als HTML-Seite im Browser. Der Nutzer wählt; der Look bleibt dann für die ganze Serie.
2. Die Figur als **Textbaustein** festhalten, der wortgleich in jedem Bild- und Clip-Prompt steht. Beispiel aus der Höhe-Folge: „faceless grey mannequin with smooth matte light-grey head without facial features, visible ball joints, burnt-orange technical jacket, charcoal trousers, dark trail shoes; minimal dark studio, deep charcoal background, soft warm top light, gentle haze, premium 3D look“.
3. Höchstens drei Figuren im Bild. Bei mehreren Figuren das Bauteil frei in den dunklen Raum stellen („isolated in a dark void … no landscape, no sky“), sonst baut das Modell eine Landschaft dazu.
4. Vor den teuren Clips **einen** Kurztest in 480p: Bleibt die Figur in Bewegung gleich?

## Phase 4: Skript

12 bis 14 kurze Sätze, je ein Gedanke. Der Nutzer gibt das Skript frei. Dramaturgie in Anteilen der Länge:

| Anteil | Takt | Inhalt |
|---|---|---|
| 0 bis 7 % | Frage | steht als Titel im Bild, die Figur stellt die Lage dar |
| 7 bis 19 % | Echter Beweis | zeigen, dass es wirklich passiert ist |
| 19 bis 30 % | Aufbau | wie wurde getestet, womit gemessen? |
| 30 bis 46 % | Mechanismus | was passiert im Körper, im System, in der Maschine? |
| 46 bis 74 % | Messung | die eine Zahl, groß und mit Vergleich |
| 74 bis 82 % | Kipppunkt | der Moment, in dem es ernst wird |
| 82 bis 100 % | Erkenntnis | Antwort auf die Frage, mit Zahl |

Vor der Aufnahme jeden Satz gegen die Quellenliste prüfen: sachlich richtig, eindeutig, keine zwei Zahlen, die ohne Erklärung wie ein Widerspruch wirken. Im Sprechtext Zahlen ausschreiben („siebenundsechzig Prozent“), in Schrift und Untertiteln Ziffern („67 %“).

## Phase 5: Stimme

- ElevenLabs, das ganze Skript **in einem Durchgang** mit Zeitstempeln (Endpunkt `with-timestamps`). Nie Satz für Satz erzeugen, nie schneiden, strecken oder beschleunigen. Das Bild richtet sich nach der Stimme.
- Vorher drei Anweisungen an denselben zwei Sätzen testen und vergleichen (Tonhöhe, Gleichmaß, Pausen), erst dann die ganze Folge.
- Ruhiger Ton, der in der Höhe-Folge gewann: Modell `eleven_v4`, am Textanfang `[soft-spoken, calm, intimate narration, low energy, steady pace]`, `stability` 0,9, `similarity_boost` 0,8, `style` 0, `speed` 0,95.
- Ein eigener Stimmklon braucht ein ElevenLabs-Abo mit Stimmklon.

## Phase 6: Zeitkarte

Die Satzanfänge im Ton an echten Pausen messen, nicht aus der Zeitachse schätzen. Daraus eine Zeitdatei, die Bildabschnitte, Schrift, Untertitel und Effekte steuert. Befehle: [references/technik.md](references/technik.md), Abschnitt 2.

## Phase 7: Startbilder und Clips

- Je Szene ein Startbild mit GPT Image 2.5 über Higgsfield, 9:16. Daraus ein 5-Sekunden-Clip mit Wan 3.0 Prime in 720p, ohne Ton. Richtwerte Stand 30.10.2026: 0,5 Credits je Bild, 15 Credits je Clip.
- Clip-Prompts beschreiben nur Bewegung und Kamera und enden mit „premium 3D look exactly as in the start image, the mannequins stay faceless and identical. No text, no logos.“
- Für einen Leuchtkasten im Bild: Startbild mit leerem weißem Leuchtkasten, im Clip-Prompt „the lightbox panel stays completely plain bright white … locked-off static camera“. Den Inhalt setzt später der Code hinein.
- **Jeden Clip absuchen**, bevor der Nutzer ihn sieht: erfundene Personen, verformte Hände, Objekte, die es nicht geben dürfte. In der Höhe-Folge lief eine vierte Figur ins Bild; sie wurde per Ausschnitt entfernt.

Befehle: [references/technik.md](references/technik.md), Abschnitt 9.

## Phase 8: Schnitt, Leuchtkasten, Schrift, Untertitel

- Clips auf 1080 × 1920, 30 Bilder je Sekunde, `setsar=1`. Ist ein Clip kürzer als sein Satz, leicht verlangsamen und das letzte Bild halten.
- **Leuchtkasten:** Das echte Bild ist immer das Vollbild, nur kleiner gezeichnet und vom Kasten beschnitten. Beim Zoom wächst es mit dem Kasten, bis es den Screen füllt. Keine Überblendung, nie zwei Bilder übereinander.
- **Schrift, Zähler, Grafiken, Untertitel** als HTML-Seite, Bild für Bild mit Chrome ohne Fenster als transparente Ebene gerendert und per ffmpeg überlagert. Abschattungen liegen **unter** der Grafik, nicht darüber.
- Jede Grafik sagt genau eine Sache und bleibt lange genug stehen, um sie zu lesen. Ein Messwert steht immer mit dem, was gemessen wurde („68 % Sauerstoff im Blut, am Finger gemessen, normal 98 %“).
- Untertitel zurückhaltend: weiß, ohne Kasten, eine Zeile bis rund 28 Zeichen, nur Zahlen in der Akzentfarbe, höchstens 0,2 s nach dem Wort.
- Nichts zu weit oben und nichts zu weit unten: In den Apps liegen oben und unten Bedienelemente über dem Bild.

Befehle: [references/technik.md](references/technik.md), Abschnitte 3 bis 6.

## Phase 9: Ton

Musik leise unter der Stimme; sie duckt, wenn gesprochen wird. Effekte nur auf Bildereignisse wie Zoom, Blitz oder Zähler, ein hörbarer Akzent je Abschnitt. Ein Donner kommt kurz nach dem sichtbaren Blitz. Am Ende −16 LUFS. Befehle: [references/technik.md](references/technik.md), Abschnitt 7.

## Phase 10: Prüfen und abgeben

Erst fertig melden, wenn alles belegt ist:

1. Transkript der fertigen Mischung (Whisper) gegen den Sprechtext.
2. Einzelbild an jeder Zahl: stimmt die Zahl, ist sie lesbar?
3. Seitenverhältnis: `ffprobe` muss Bildpunkte 1:1 und Anzeige 9:16 zeigen.
4. Lautheit rund −16 LUFS.
5. Auf dem Handy ansehen, in der Ansicht der Zielplattform.

Befehle: [references/technik.md](references/technik.md), Abschnitt 8.

## Was beim Beispiel den Unterschied gemacht hat

1. **Stimme in einem Durchgang.** Satz für Satz erzeugt klang jeder Satz wie ein Neuanfang, geschnittene Übergänge haben Wörter angeschnitten.
2. **Zoom ohne Doppelbild.** Das echte Bild wächst mit dem Kasten, statt überzublenden.
3. **Kräftige Grafiken.** Eine Ebene zu viel über der Grafik hat alles ausgewaschen. Das fällt erst am Handy auf.
4. **Messwert mit Namen.** Eine Zahl allein versteht niemand.
5. **Ruhig ist eine Frage des Tempos.** Anweisung und hohe Stabilität machen die Stimme tiefer und gleichmäßiger, den Rest machen Pausen.
6. **Seitenverhältnis prüfen.** Nach dem Stapeln zweier Bildstreifen stand in der Datei ein falsches Bildpunkt-Verhältnis, Player zeigten das Video gestaucht. `setsar=1` nach jedem Stapeln.
