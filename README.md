# RueckbauDBDaten

Datenbestand für **RueckbauDB**: Massen, Stückzahlen und Abmessungen der Bauteile von
Windenergieanlagen, jeweils belegt durch die Herstellerunterlage, aus der die Zahl stammt.
Grundlage für Rückbau- und Verwertungskalkulationen.

Der gesamte Datenbestand steckt in einer einzigen Datei: **`data.json`**.

---

## Was in Git liegt — und was nicht

| | Inhalt |
|---|---|
| **versioniert** | `data.json`, `README.md`, `DOKUMENTATION.md`, `.gitignore` |
| **ignoriert** | `Alle Dokumente/`, `Dokumente/`, `Import/`, `*.bak`, `.DS_Store` |

Die beiden Dokumentordner umfassen **je rund 1,5 GB** an Hersteller-PDFs, Zeichnungen, Scans
und Outlook-Nachrichten — das gehört nicht in ein Git-Repository. `Import/` ist nur ein
Durchlaufordner.

> **Wichtig nach einem frischen Clone:** `data.json` verweist über relative Pfade auf Dateien
> unter `Dokumente/`. Ohne den danebengelegten Dokumentbestand sind alle 1.084
> Dokumentverknüpfungen tot. Die Anwendung sollte das abfangen, statt zu fehlschlagen.

---

## Verzeichnisstruktur

```
.
├── data.json                       <- der Datenbestand (versioniert)
├── README.md                       <- diese Datei
├── DOKUMENTATION.md                <- Schema, Prüfbericht, Anhänge, Importprotokolle
├── Import/                         <- Eingang für neue Unterlagen        (ignoriert)
├── Dokumente/                      <- ausgewertete Quelldokumente        (ignoriert, ~1,5 GB)
│   ├── AN Bonus/  Dewind/  Enercon/  Gamesa/  GE/  Kenersys/
│   ├── Nordex/  REpower/  Senvion/  Vensys .../  Vestas/  ...
└── Alle Dokumente/                 <- Archiv-/Rohkopie des Bestands      (ignoriert, ~1,5 GB)
```

---

## Kennzahlen

| | |
|---|---|
| `data.json` | 575 KB, UTF-8 |
| Kategorien | 9 (Maschinenhaus, Generator, Nabe, Rotorblatt, Turm, Fundament, Elektro, Ersatzteil, Sonstiges) |
| Quellen (Belegdokumente) | 59 |
| Konfigurationen | 107 |
| Hersteller | 14 |
| Positionen (Bauteile) | 752 |
| Dokumentverknüpfungen | 1.084 auf 467 verschiedene Dateien, davon 82 als Beleg |

Konfigurationen je Hersteller:

| Hersteller | Anz. | Hersteller | Anz. | Hersteller | Anz. |
|---|---|---|---|---|---|
| Enercon | 24 | Vestas | 10 | REpower | 6 |
| AN Bonus | 23 | GE | 10 | DeWind | 4 |
| Nordex | 14 | NEG Micon | 6 | Gamesa | 3 |
| Kenersys | 3 | Fuhrländer | 1 | Schuler | 1 |
| Vensys | 1 | Test Hersteller | 1 | | |

---

## Aufbau der `data.json`

```jsonc
{
  "Kategorien":     [ … ],   // Bauteilgruppen + Anzeigefarben
  "Quellen":        [ … ],   // Belegdokumente mit Vertrauensgrad (hoch: 39, mittel: 20)
  "Konfigurationen":[ … ]    // Anlagentyp × Nabenhöhe × Variante
}
```

Eine Konfiguration trägt die Stammdaten der Anlage sowie zwei Listen — die Bauteile und die
zugehörigen Dateien:

```jsonc
{
  "Id": 189,
  "Hersteller": "Gamesa",
  "Modell": "G80/2000",
  "Variante": "NH 100 m (WZ II / IEC IIA)",
  "NennleistungKw": 2000,
  "RotordurchmesserM": 80,
  "NabenhoeheM": 100,
  "TurmTyp": "Stahl",
  "QuelleId": 92,

  "Positionen": [
    { "Id": 1281, "KonfigurationId": 189, "Sort": 3, "Kategorie": "Turm",
      "Bezeichnung": "Turmsektion 1 (unten)", "Stueckzahl": 1,
      "EinzelgewichtKg": 62800, "WertOriginal": "62.800 kg (WZ II / IEC IIA)",
      "LaengeM": "15.609", "BreiteM": "4.037", "QuelleId": 92 }
  ],

  "Dokumente": [
    { "Pfad": "Dokumente/Gamesa/G80 - 100m/Gamesa G80.pdf",
      "Name": "Gamesa G80.pdf", "Quelle": true, "Kb": 537 }
  ]
}
```

Gesamtgewicht einer Konfiguration = `Σ (EinzelgewichtKg × Stueckzahl)`.
`WertOriginal` hält das Originalzitat aus dem Dokument fest, inklusive Unschärfe („ca. 18,5–20,0 t").

Die vollständigen Feldtabellen stehen in [DOKUMENTATION.md](DOKUMENTATION.md), Abschnitt 2.

---

## Pfadkonvention

Jeder Pfad in `Konfigurationen[].Dokumente[].Pfad` ist **relativ zum Ablageort der `data.json`**:

* beginnt immer mit `Dokumente/` — **nie** mit `Import/`,
* immer `/` als Trenner, auch unter Windows,
* nie absolut, nie mit `..`, nie mit Laufwerksbuchstaben,
* Umlaute in der Normalform, die das Dateisystem liefert (macOS: NFD).

Öffnen in C#:

```csharp
var basis = Path.GetDirectoryName(dataJsonPfad)!;
var voll  = Path.Combine(basis, doc.Pfad.Replace('/', Path.DirectorySeparatorChar));
```

`Quelle: true` heißt: aus dieser Datei stammen Zahlen dieser Konfiguration.
`Quelle: false` ist Begleitmaterial, das nur zur Anlage gehört.

---

## Ablauf beim Import

1. Neuen Ordner nach `Import/` legen, Struktur `<Hersteller>/<Modell- bzw. Nabenhöhen-Ordner>/`.
2. Unterlagen auswerten, `data.json` um `Quellen`, `Konfigurationen`, `Positionen` und
   `Dokumente` ergänzen.
3. Ordner **unverändert** von `Import/` nach `Dokumente/` verschieben.
4. Pfade auf `Dokumente/…` setzen und prüfen. `Import/` ist danach wieder leer.

Vor jedem Lauf eine Sicherung anlegen (`data.json.bak` — ist ignoriert), und danach prüfen:

* jeder `Dokumente[].Pfad` existiert relativ zum `data.json`-Verzeichnis,
* kein `Pfad` beginnt mit `Import/`, ist absolut oder enthält ein `..`-Segment,
* `Kb` und `Name` passen zur Datei,
* jede `QuelleId` und jede `Kategorie` existiert,
* `Sort` je Konfiguration lückenlos ab 0,
* `Positionen.KonfigurationId` == `Konfigurationen.Id`.

Ergänzungen sollten **rein additiv** sein — der Bestand wächst, bestehende Einträge werden nicht
umgeschrieben. `data.json` wird von System.Text.Json geschrieben (2 Leerzeichen Einrückung,
`ß`/`+`-Escapes in Großschreibung); wer die Datei mit anderem Werkzeug anfasst, sollte
diesen Stil beibehalten, damit der Diff klein bleibt.

---

## Stand der Prüfung

Zuletzt geprüft am 2026-08-25. Sauber sind: JSON-Syntax, Eindeutigkeit aller Ids, Referenzen auf
`Quellen` und `Kategorien`, `Sort`-Lückenlosigkeit, positive Gewichte und Stückzahlen sowie alle
467 Dokumentpfade — jede Datei existiert, keine zeigt auf `Import/`.

Offen sind unter anderem:

* **Konfiguration 190 („Test Hersteller / Test Modell")** ist Testdatensatz ohne `QuelleId`,
  `WertOriginal` und Quellenbezug — vor dem Ausliefern entfernen.
* **Gesamtgewichte sind nicht vergleichbar**: manche Konfigurationen enthalten den Fundamentbeton,
  andere nur die Maschine, einzelne nur das Fundament. Auswertungen nach Kategorie statt über die
  Gesamtsumme.
* **`Quellen.Dateiname` trägt weiterhin nur Basisnamen**, teils mehrdeutig. Verlässlich ist nur der
  Weg über `Konfigurationen[].Dokumente`.
* **Kategorisierung uneinheitlich** (Getriebe mal `Maschinenhaus`, mal `Sonstiges`; `Ersatzteil`
  ungenutzt), und Maße sind Strings, während Gewichte `int` sind.

Die vollständige Befundliste mit Begründung steht in [DOKUMENTATION.md](DOKUMENTATION.md),
Abschnitt 3.2; die Importprotokolle in den Abschnitten 4 und 5.
