# Skills von Public Impact

Kostenlose Skills für Claude Code und andere KI-Agenten, für alle, die mit KI Videos machen. Einmal installiert, für jedes Projekt nutzbar.

Ein Skill ist eine Arbeitsanleitung, die dein KI-Agent liest und Schritt für Schritt ausführt: Er fragt nach, was er wissen muss, führt die Befehle auf deinem Rechner aus und zeigt dir das Ergebnis. Jeder Skill ist ein Ordner mit einer `SKILL.md` im offenen Format [Agent Skills](https://agentskills.io). Ohne Agent lässt sich die `SKILL.md` auch als Anleitung lesen, die Befehle stehen darin.

| Skill | Was er macht | Was du brauchst |
|---|---|---|
| [erklaerfolge](skills/erklaerfolge/) | Baut mit dir ein kurzes Erklärvideo im Hochformat: eine Frage, ein echter Beweis, eine Erkenntnis. Eine Figur ohne Gesicht erklärt, echtes Material läuft im Leuchtkasten. Wie das Vorbild entstand: [making-of-hoehe](https://github.com/publicimpact/making-of-hoehe) | Python, ffmpeg, Node.js, Konten bei Higgsfield und ElevenLabs (kostenpflichtig; der Skill nennt vor jedem Schritt die Kosten) |
| [footage-organizer](skills/footage-organizer/) | Sieht sich jeden Clip an, erkennt Inhalt und Kamera und sortiert alles in beschriftete Ordner, schnittfertig | ffmpeg |
| [drehpapiere](skills/drehpapiere/) | Kameramann-Vertrag, Protagonist-Vertrag, Einverständniserklärung, Drehgenehmigung und Equipmentliste fürs Carnet ATA, druckfertig als HTML | nichts weiter |
| [untertitel](skills/untertitel/) | Deutsch, Französisch oder jede andere Sprache rein, englische SRT-Datei raus. Whisper läuft lokal, kein Upload | Python, ffmpeg |

## Installieren

**Mit dem Werkzeug `skills`** (für Claude Code und viele andere Agenten):

```bash
npx skills add publicimpact/skills
```

Nur einen Skill:

```bash
npx skills add publicimpact/skills --skill untertitel
```

**In Claude Code als Plugin:**

```
/plugin marketplace add publicimpact/skills
/plugin install video@publicimpact
```

Danach heißen die Skills `/video:untertitel`, `/video:drehpapiere` und so weiter.

**Von Hand:** den Ordner eines Skills, etwa `skills/untertitel`, nach `~/.claude/skills/untertitel` kopieren.

## Benutzen

Claude Code im Ordner deines Projekts öffnen und einfach sagen, was du brauchst. Der Agent erkennt den passenden Skill an deiner Bitte, zum Beispiel:

- „Ich habe 40 Clips vom Dreh, sortier mein Material.“
- „Ich brauche einen Vertrag für meinen Kameramann.“
- „Mach mir englische Untertitel für interview.mp4.“
- „Bau mit mir eine Erklärfolge zum Thema …“

Oder direkt aufrufen: `/untertitel` (installiert mit `npx skills` oder von Hand) beziehungsweise `/video:untertitel` (als Plugin). Fehlt ein Programm wie ffmpeg oder Whisper, sagt der Skill, wie du es installierst.

## Hinweise

- Die Drehpapiere sind Mustervorlagen und ersetzen keine Rechtsberatung.
- Untertitel und Footage Organizer laufen lokal und kosten nichts. Die Erklärfolge nutzt bezahlte Dienste und fragt vor jedem bezahlten Schritt.
- Footage Organizer, Drehpapiere und Untertitel entstanden zuerst für die Community Filmemacher Deutschland.
- Lizenz: MIT, siehe [LICENSE](LICENSE). Nutzen, anpassen und weitergeben, auch kommerziell.

---

Public Impact · [publicimpact.ai](https://publicimpact.ai) · hey@publicimpact.ai
