# RueckbauDB — Datenbestand und `data.json`

Stand: 2026-08-25 · letzter Import: **Restarchiv aus `Import/`** — 9 neue Hersteller,
62 Konfigurationen, 435 Positionen, 503 Dokumente (siehe Kapitel 5)

Alle Pfade in diesem Dokument sind **relativ zum Ablageort der `data.json`** (= Wurzelverzeichnis
dieses Ordners). Ein Pfad wie `Dokumente/Gamesa/G80 - 100m/…` ist also immer als
`<Wurzel>/Dokumente/Gamesa/G80 - 100m/…` zu lesen. Absolute Pfade kommen bewusst nirgends vor,
damit der Ordner als Ganzes verschiebbar und auf jedem Rechner lauffähig bleibt.

---

## 1. Überblick

| | |
|---|---|
| Datei | `data.json` (575 KB, UTF-8, JSON mit `\uXXXX`-Escapes, Umlaute in Pfaden als NFD) |
| Kategorien | 9 |
| Quellen (Belegdokumente) | 59 |
| Konfigurationen (Anlagentyp × Nabenhöhe × Variante) | 106 + 1 Testdatensatz (Id 190) |
| Positionen (Bauteile) | 752 |
| Dokument-Verknüpfungen (`Konfigurationen[].Dokumente`) | 1.084 auf 467 verschiedene Dateien |
| Hersteller | 13 (AN Bonus, DeWind, Enercon, Fuhrländer, Gamesa, GE, Kenersys, NEG Micon, Nordex, REpower, Schuler, Vensys, Vestas) + „Test Hersteller" |

### Verzeichnisstruktur

```
.                                   <- Wurzel = Ablageort der data.json
├── data.json                       <- erzeugter Datenbestand
├── DOKUMENTATION.md                <- dieses Dokument
├── Import/                         <- Eingang: neuer Ordner hier hinein -> data.json wird ergänzt
│                                      (aktuell leer = Ruhezustand)
├── Dokumente/                      <- importierte Quelldokumente (584 Dateien, 17 Ordner)
│   ├── AN Bonus/   ├── Gamesa/      ├── NEG Micon NM60_1000 - 80m/  ├── Senvion/
│   ├── Dewind/     ├── GE/          ├── Nordex/                     ├── Vensys V62 - 69m/
│   ├── Enercon/    ├── Kenersys/    ├── Nordtank 1500 - 67,5m/      ├── Vestas/
│   ├── Fuhrländer FL1000 - 70m/     ├── REpower/                    ├── Wind World/
│   └── Schuler SDD100 - 100m/                                       └── __Senvion/
└── Alle Dokumente/                 <- Gesamtarchiv (585 Dateien, 16 Hersteller)
```

### Ablauf

1. Ein neuer Ordner wird nach `Import/` gelegt — Struktur `<Hersteller>/<Modell-/Nabenhöhen-Ordner>/`.
2. Die Dokumente werden ausgewertet, `data.json` wird um `Quellen`, `Konfigurationen` und
   `Positionen` ergänzt.
3. Der Ordner wird **unverändert** von `Import/` nach `Dokumente/` verschoben; die Pfade in
   `Konfigurationen[].Dokumente[].Pfad` zeigen ab dann auf `Dokumente/…`, **nie** auf `Import/`.
4. `Import/` ist danach wieder leer — das ist der erwartete Ruhezustand.

`Dokumente/` enthält seit dem Import vom 2026-08-25 das vollständige Archiv aus `Alle Dokumente/`
(bis auf die Protokolldatei `___All_Errors.txt`); jede Datei liegt dort zusätzlich als Archivkopie.
**Nicht jede Datei unter `Dokumente/` ist ausgewertet**: 117 Dateien gehören zu Anlagentypen, für
die sich aus den vorliegenden Unterlagen keine Gewichtsangaben gewinnen ließen, und sind deshalb mit
keiner Konfiguration verknüpft (Liste in Anhang G). Der Nachweis, welche Dateien ausgewertet wurden,
läuft ausschließlich über `Konfigurationen[].Dokumente`.

---

## 2. Aufbau der `data.json`

```jsonc
{
  "Kategorien":     [ … ],   // Stammdaten: Bauteilgruppen + Farben für die Anzeige
  "Quellen":        [ … ],   // Belegdokumente mit Vertrauensgrad
  "Konfigurationen":[ … ]    // je Anlagentyp/Nabenhöhe, enthält "Positionen" und "Dokumente"
}
```

### 2.1 `Kategorien`

| Feld | Typ | Bemerkung |
|---|---|---|
| `Id` | int | Primärschlüssel |
| `Name` | string | wird in `Positionen.Kategorie` **als Text** referenziert, nicht über die Id |
| `FarbeBg` / `FarbeText` | string | Hex-Farben |
| `Sort` | int | Anzeigereihenfolge, 0-basiert |

| Id | Name | FarbeBg | FarbeText | Sort |
|---|---|---|---|---|
| 1 | Maschinenhaus | `#2e9e5b` | `#2e9e5b` | 0 |
| 2 | Generator | `#b8860b` | `#b8860b` | 1 |
| 3 | Nabe | `#8e44ad` | `#8e44ad` | 2 |
| 4 | Rotorblatt | `#558b2f` | `#558b2f` | 3 |
| 5 | Turm | `#2f6fb0` | `#2f6fb0` | 4 |
| 6 | Fundament | `#6b6b6b` | `#6b6b6b` | 5 |
| 7 | Elektro | `#c99700` | `#c99700` | 6 |
| 8 | Ersatzteil | `#7f8c8d` | `#7f8c8d` | 7 |
| 9 | Sonstiges | `#95a5a6` | `#95a5a6` | 8 |

### 2.2 `Quellen`

| Feld | Typ | Pflicht | Bemerkung |
|---|---|---|---|
| `Id` | int | ja | Primärschlüssel |
| `Dateiname` | string | ja | **nur der Dateiname, kein Pfad**; mehrere Dateien mit `; ` getrennt |
| `Dokumenttyp` | string | nein | fehlt bei 6 Quellen |
| `Datum` | string (`YYYY-MM-DD`) | nein | nur bei Quelle 5 und 92 gesetzt |
| `Vertrauensgrad` | string | ja | `hoch` / `mittel` |

### 2.3 `Konfigurationen`

| Feld | Typ | Belegt | Bemerkung |
|---|---|---|---|
| `Id` | int | 44/44 | Primärschlüssel |
| `Hersteller` | string | 44/44 | |
| `Modell` | string | 44/44 | bei Nordex und Gamesa inkl. Leistung, z. B. `N100/2500`, `G80/2000` |
| `Variante` | string | 30/44 | fehlt bei allen Nordex-Konfigurationen |
| `Beschreibung` | string | 30/44 | fehlt bei allen Nordex-Konfigurationen |
| `NennleistungKw` | int | 30/44 | fehlt bei allen Nordex-Konfigurationen |
| `RotordurchmesserM` | int | 44/44 | |
| `NabenhoeheM` | int/float | 43/44 | fehlt bei Konfiguration 26 (DeWind D8) |
| `TurmTyp` | string | 30/44 | fehlt bei allen Nordex-Konfigurationen |
| `QuelleId` | int | 44/44 | → `Quellen.Id` |
| `Notiz` | string | 4/44 | |
| `Positionen` | array | 44/44 | siehe 2.4 |
| `Dokumente` | array | 44/44 | siehe 2.5 |

### 2.4 `Positionen`

| Feld | Typ | Belegt | Bemerkung |
|---|---|---|---|
| `Id` | int | 315/315 | global eindeutig |
| `KonfigurationId` | int | 315/315 | redundant zur Verschachtelung, stimmt überall überein |
| `Sort` | int | 315/315 | je Konfiguration lückenlos 0..n-1 |
| `Kategorie` | string | 315/315 | Textreferenz auf `Kategorien.Name` |
| `Bezeichnung` | string | 315/315 | ASCII-transliteriert (`Fundamentkoerper`, `Zubehoer`, …) |
| `Stueckzahl` | int | 315/315 | ≥ 1 |
| `EinzelgewichtKg` | **int** | 315/315 | Gewicht **je Stück** |
| `WertOriginal` | string | 315/315 | Originalzitat aus dem Dokument, inkl. Unschärfe („ca. 18,5–20,0 t") |
| `LaengeM` / `BreiteM` / `HoeheM` | **string** | teilweise | Zahl als Zeichenkette, Punkt als Dezimaltrenner |
| `QuelleId` | int | 315/315 | → `Quellen.Id`, kann von der Konfigurations-Quelle abweichen |

**Gesamtgewicht** einer Konfiguration = `Σ (EinzelgewichtKg × Stueckzahl)`.

### 2.5 `Dokumente` — die relativen Pfade

Jede Konfiguration führt die zugehörigen Dateien mit **relativem Pfad ab dem `data.json`-Verzeichnis**:

```jsonc
"Dokumente": [
  { "Pfad": "Dokumente/Gamesa/G80 - 100m/Gamesa G80.pdf",
    "Name": "Gamesa G80.pdf",
    "Quelle": true,      // aus dieser Datei stammen Zahlen dieser Konfiguration
    "Kb": 537 }
]
```

| Feld | Typ | Bemerkung |
|---|---|---|
| `Pfad` | string | relativ, immer mit `/`, beginnt immer mit `Dokumente/` |
| `Name` | string | Basisname, identisch mit dem letzten Pfadsegment |
| `Quelle` | bool | `true` = Beleg, aus dem Positionen abgeleitet wurden; `false` = Begleitmaterial |
| `Kb` | int | Dateigröße, `round(Bytes / 1024)` |

Öffnen in C#:

```csharp
var basis = Path.GetDirectoryName(dataJsonPfad)!;
var voll  = Path.Combine(basis, doc.Pfad.Replace('/', Path.DirectorySeparatorChar));
```

---

## 3. Prüfergebnis

### 3.1 Fehlerfrei

| Prüfung | Ergebnis |
|---|---|
| JSON syntaktisch gültig | OK |
| `Konfigurationen.Id` / `Positionen.Id` eindeutig | OK |
| `Positionen.KonfigurationId` == umschließende Konfiguration | OK, alle 752 |
| `QuelleId` → existierende Quelle | OK, alle Referenzen (Ausnahme: Testdatensatz 190 führt gar keine `QuelleId`) |
| `Positionen.Kategorie` → existierende Kategorie | OK |
| Verwaiste Quellen (nirgends referenziert) | keine |
| `Sort` je Konfiguration lückenlos 0..n-1 | OK |
| `EinzelgewichtKg` > 0, `Stueckzahl` ≥ 1 | OK |
| **`Dokumente[].Pfad`: alle 1.084 Verknüpfungen auflösbar** | OK |
| **`Pfad` zeigt auf `Import/`** | 0 Treffer |
| **`Pfad` absolut, mit `..` oder `\`** | 0 Treffer |
| **`Kb` == tatsächliche Dateigröße** | OK, alle 1.084 |
| **`Name` == Basisname des Pfades** | OK |

### 3.2 Offene Befunde

#### B1 — Quelle 91: Dateiname enthält einen Schrägstrich

```json
{ "Id": 91, "Dateiname": "Zeichnung Bewehrungsplan Fundament Blatt 4c/4d.pdf" }
```

Diese Datei existiert nicht; gemeint sind zwei Dateien (`… Blatt 4c.pdf` und `… Blatt 4d.pdf`).
Der `/` wirkt beim Zusammenbauen eines Pfades zusätzlich als Verzeichnistrenner.
Korrektur: als zwei mit `; ` getrennte Dateinamen führen. In `Konfigurationen[186].Dokumente` ist
korrekt nur Blatt 4c mit `"Quelle": true` markiert.

#### B2 — `Quellen.Dateiname` bleibt ohne Pfad *(auf Konfigurationsebene gelöst)*

Über `Konfigurationen[].Dokumente[].Pfad` ist jedes Dokument eindeutig auflösbar. `Quellen` selbst
trägt weiter nur Basisnamen, und fünf davon sind mehrdeutig:

| Quelle | Dateiname | Treffer unter `Dokumente/` |
|---|---|---|
| 6 | `Turmgewichte-600 kw.pdf` | 6 (je einmal pro Nabenhöhen-Ordner) |
| 10 | `Betriebsanleitung D6 1000- 1250KW.pdf` | 2 |
| 45 | `N80 transport data.pdf` | 2 |
| 46 / 47 | `K0801_025550_…_Rueckbauaufwand_K08g.pdf` | 2 |

Wer `Quellen` isoliert auswertet, landet weiterhin bei mehreren Kandidaten. Empfehlung:
`Quellen` ebenfalls um ein `Dateien: [{ "Pfad": … }]` ergänzen oder die Auflösung ausschließlich
über `Konfigurationen[].Dokumente` vornehmen und das im Modell festschreiben.

#### B3 — Eine Quelle deckt mehrere Ordner ab (1:n)

Der Pfad hängt an der **Kombination Quelle × Konfiguration**, nicht an der Quelle allein:
Quelle 6 wird von sechs Konfigurationen benutzt und liegt sechsmal vor, je einmal pro
Nabenhöhen-Ordner. Genau dafür sitzt `Dokumente` an der Konfiguration und nicht an der Quelle.

Ordnerübergreifende Belege gibt es wirklich: Konfiguration 114 (N90/2500) verweist auf
`K0801_025550_…pdf`, die im Ordner `Nordex N90 -80m` nicht liegt. Ebenso beziehen die beiden
60-m-Gamesa-Konfigurationen ihre Turmgewichte aus `Dokumente/Gamesa/G80 - 100m/Gamesa G80.pdf`
(allgemeines G80-Datenblatt, das alle Nabenhöhen abdeckt).

#### B4 — Konfiguration 124 mit fachfremder Quelle

Konfiguration 124 ist eine **S77/1500** (NH 100 m), verweist aber auf Quelle 50 =
`S70-1-techn-description-de.pdf`, also das S70-Datenblatt. Im selben Ordner liegt mit
`GA Nr. 0823.S77.03.12 Anl. 2.pdf` ein unbenutztes S77-Dokument. Bitte prüfen, ob die Quelle
vertauscht wurde.

#### B5 — Bezugsgröße „Gesamtgewicht" ist nicht vergleichbar

Die Konfigurationen erfassen unterschiedlich viel Umfang:

- Konfiguration 189 (G80, NH 100 m): 1.569 t — davon **1.153 t Fundamentbeton und -stahl**.
- Konfiguration 117 (N117/2400): 1.760 t — **ausschließlich** Fundamentpositionen.
- Konfiguration 50/51 (DeWind D6): 38 t — nur Triebstrang und Blätter, **kein Turm**.
- Konfiguration 34–36 (AN Bonus 1,0 MW): nur Maschinenhaus + Turm, keine Nabe, keine Blätter.
- Konfiguration 186 (DeWind D6 1250): zwei Bewehrungspositionen des Sockels — Fragment.

Jede Zahl ist für sich plausibel, eine Summe oder ein Vergleich über Konfigurationen hinweg ist
ohne Umfangsangabe irreführend. Empfehlung: Flags wie `EnthaeltFundament` / `EnthaeltTurm` je
Konfiguration, oder Auswertung strikt nach Kategorie (Anhang B) statt über die Gesamtsumme.

#### B6 — Uneinheitliche Kategorisierung

- `Getriebe`: bei Konfiguration 50/51 unter **`Sonstiges`**, bei 115 unter **`Maschinenhaus`**.
- `Azimutlager` / `Azimutdrehverbindung`: bei 50/51 **`Sonstiges`**, bei 115 **`Maschinenhaus`**.
- Kategorie `Generator` wird nur von 50, 51 und 115 benutzt; sonst steckt der Generator im
  Sammelposten „Maschinenhaus komplett" — so auch bei den neuen Gamesa-Konfigurationen.
- Kategorie `Ersatzteil` (Id 8) wird von **keiner** Position verwendet.

#### B7 — Datentypen uneinheitlich

`EinzelgewichtKg` ist `int`, `LaengeM` / `BreiteM` / `HoeheM` sind **Strings** (`"4.34"`).
Empfehlung: als `decimal?` schreiben (invariant, Punkt als Dezimaltrenner) oder bewusst als String
belassen und im Modell dokumentieren.

#### B8 — Fehlende Stammdaten bei Nordex

Die 14 Nordex-Konfigurationen (Id 113–126) haben kein `NennleistungKw`, `Variante`, `Beschreibung`
und `TurmTyp`. Die Leistung steckt im `Modell` (`N100/2500` → 2500 kW) und ließe sich ableiten.
Bei Konfiguration 26 (DeWind D8) fehlt die `NabenhoeheM`.

#### B9 — Mehrdeutige fachliche Schlüssel

Hersteller + Modell + Nabenhöhe ist **nicht** eindeutig; unterschieden wird nur über `Variante`:

- AN Bonus 1,3 MW, NH 49 m → 40 („High Gust") und 41
- AN Bonus 1,3 MW, NH 68 m → 43, 44, 47 („Transport")
- AN Bonus 1,3 MW, NH 80 m → 45 und 48
- **Gamesa G80/2000, NH 60 m → 187 (WZ II / IEC IIA) und 188 (WZ III / IEC IA)**

Eine Auswahl „Hersteller → Modell → Nabenhöhe" muss die Variante zwingend mit anzeigen.

#### B10 — Ein Viertel der abgelegten Dokumente ist Beleg

Unter `Dokumente/` liegen 81 Dateien; **22** sind mit `"Quelle": true` als Beleg
verknüpft, 59 nicht (Anhang C). Davon sind 15 Dateien in keiner
`Dokumente[]`-Liste enthalten — sie sind aus der Anwendung heraus gar nicht erreichbar, darunter
der komplette Ordner `Dokumente/Nordex/Nordex N62 -69m/` (15 Gutachten, Prüfberichte und Zeichnungen), aus dem
keine Konfiguration entstanden ist.

`Alle Dokumente/` (585 Dateien, 16 Hersteller) ist weiterhin überwiegend unausgewertet; offen sind
u. a. Enercon, Vestas, GE, Senvion, REpower, Vensys, Nordtank, Schuler, Kenersys, Wind World,
NEG Micon und Fuhrländer.

#### B11 — Hinweis aus dem Download-Log

`Alle Dokumente/___All_Errors.txt` meldet für den Lauf vom 27.07.2026 eine nicht heruntergeladene
Datei: `Senvion/V-1.1-GP.00.10-A-M Spezifikation f?r Transport.pdf`. Das `?` deutet auf ein
Encoding-Problem im Dateinamen hin (vermutlich „für"). Bei Senvion ist der Bestand unvollständig.

#### B12 — Id-Lücken und Ordnernamen

- `Konfigurationen.Id` hat Lücken (52–112, 127–185); `Positionen.Id` reicht bis 1289. Die Ids
  stammen aus einer Datenbank und werden nicht lückenlos neu vergeben — sie taugen nicht als
  dauerhafte externe Referenz, solange `data.json` neu geschrieben wird.
- Ordnernamen sind uneinheitlich (`AN Bonus 0,6_44-50m` vs. `An Bonus 0,6_44-55m`,
  `Dewind D6 - 1000` vs. `Dewind D6 -1250`, `Nordex N117_2400` vs. `Nordex N117-2400 Hambuch`).
  Solange Pfade explizit gespeichert werden, ist das nur ein Schönheitsfehler — jede Heuristik
  „Ordner aus Modellname ableiten" würde daran scheitern.

---

## 4. Import Gamesa G80 (Lauf vom 2026-08-19)

### Verschobene Dateien

`Import/Gamesa/` → `Dokumente/Gamesa/`, Struktur unverändert:

| Datei | Größe | Rolle |
|---|---|---|
| `Dokumente/Gamesa/Maße und Gewichte WEA G80.pdf` | 109 KB | **Beleg** — Transporteinheiten und Fundamentsektionen für T. 60/78/100 m |
| `Dokumente/Gamesa/G80 - 100m/Gamesa G80.pdf` | 537 KB | **Beleg** — Technisches Datenblatt GD005900 Rev. 3, 16.11.2005 |
| `Dokumente/Gamesa/G80 - 100m/Wirfus_Auszug_Typenprüfung.pdf` | 20.394 KB | Begleitmaterial — Typenprüfung DIBt WZ II, Flachgründung NH 78/100 m; **reiner Scan ohne Textebene**, nicht ausgewertet |
| `Dokumente/Gamesa/G80 - 60m/Packing list Re-Powering 5 x V80_2MW_60m(1).xls` | 51 KB | Begleitmaterial — reale Packliste, 5 × 60-m-Turm |
| `Dokumente/Gamesa/G80 - 60m/Re_ Weights of G80 with 60m HH.msg` | 216 KB | Begleitmaterial — E-Mail-Auskunft vom 26.02.2026 |

`Import/` war danach wieder leer.

### Neue Quellen

| Id | Dokumenttyp | Datum | Vertrauensgrad |
|---|---|---|---|
| 92 | Technisches Datenblatt GD005900 Rev. 3 (Allgemeine Beschreibung und Spezifikation G80) | 2005-11-16 | hoch |
| 93 | Maße und Gewichte (G80 WTG T. 60-78-100 m, Transporteinheiten) | — | hoch |

### Neue Konfigurationen

Gamesa G80/2000: 2000 kW, Rotor 80 m, Blattlänge 39 m, konischer Stahlrohrturm.

| Id | Variante | NH | Pos. | Gesamt | davon Fundament | ohne Fundament |
|---|---|---|---|---|---|---|
| 187 | WZ II / IEC IIA | 60 m | 10 | 816,6 t | 586,9 t | 229,7 t |
| 188 | WZ III / IEC IA | 60 m | 10 | 842,8 t | 601,5 t | 241,3 t |
| 189 | WZ II / IEC IIA | 100 m | 12 | 1.569,1 t | 1.176,1 t | 393,0 t |

Der 60-m-Turm existiert in zwei Windzonen-Ausführungen, die sich um rund 11 t unterscheiden — daher
zwei Konfigurationen (vgl. B9). Das Datenblatt enthält zusätzlich einen **78-m-Turm**; dafür wurde
keine Konfiguration angelegt, weil im Import kein zugehöriger Ordner lag.

### Getroffene Annahmen

1. **Keine Doppelzählung im Triebstrang.** Das Datenblatt beziffert Getriebe (16.500 kg), Generator
   (7.100 kg), Antriebswelle (6.100 kg), Gondelgehäuse (2.000 kg), Blattlager (3 × 1.475 kg),
   Wellenlagergehäuse (1.600 kg), Wellenlager (2 × 485 kg) und Nabenabdeckung (310 kg) einzeln.
   Diese Teile stecken bereits in der Transporteinheit „Gondel" (68 t) bzw. „Nose" (19 t) und sind
   deshalb **nicht** zusätzlich als Positionen geführt — sonst entstünde derselbe Fehler wie in B5.
2. **Beton mit 2,5 t/m³** in Gewicht umgerechnet, wie bereits bei den Nordex-Konfigurationen. Der
   Originalwert in m³ steht in `WertOriginal`.
3. **Fundamentmengen für WZ II** sind im Datenblatt nicht beziffert („siehe Typenprüfungen"),
   deshalb die Mengen nach **IEC Klasse IIA** — der Turm ist laut Datenblatt für WZ II und IEC IIA
   identisch. Die Bemessungsklasse steht in jeder Fundamentposition in der `Bezeichnung`.
4. **Rotorblattgewicht 6.719 kg** aus dem Datenblatt Rev. 3. Die anderen Unterlagen nennen
   gerundete 6.500 kg (Maße-und-Gewichte-Blatt, Typenprüfung 2003, E-Mail).
5. **Turmgewichte aus dem Datenblatt Rev. 3**, nicht aus dem Maße-und-Gewichte-Blatt. Die beiden
   Quellen weichen für den 60-m-Turm ab (32,0 / 51,1 / 39,4 t gegenüber 34,0 / 56,0 / 43,0 t); die
   Werte des Maße-und-Gewichte-Blatts liegen näher an der WZ-III-Ausführung. Die reale Packliste im
   Ordner `G80 - 60m` bestätigt die Größenordnung mit 30,6 / 52,1 / 40,4 t.
6. Die **Typenprüfung Wirfus** liegt nur als Scan ohne Textebene vor und wurde nicht ausgewertet.
   Sie enthält eine ältere Fassung der 100-m-Turmgewichte (61.500 / 61.000 / 55.000 / 55.000 /
   50.000 kg) und die Bewehrungspläne des Flachfundaments. Für belastbare Bewehrungsmengen müsste
   sie per OCR erschlossen werden.

Die Annahmen 1, 3, 5 und 6 stehen zusätzlich im Feld `Notiz` der jeweiligen Konfiguration.

### Validierung beim Import

Nach dem Schreiben der `data.json` sollte geprüft und bei Verstoß protokolliert werden:

- jeder `Dokumente[].Pfad` existiert relativ zum `data.json`-Verzeichnis;
- kein `Pfad` beginnt mit `Import/`, ist absolut, enthält `..` oder `\`;
- `Kb` und `Name` passen zur Datei;
- jede `QuelleId` in `Konfigurationen`/`Positionen` existiert;
- jede `Kategorie` existiert in `Kategorien`;
- `Sort` je Konfiguration lückenlos ab 0;
- `Positionen.KonfigurationId` == `Konfigurationen.Id`;
- Dateinamen in der Normalform, die das Dateisystem liefert (macOS: NFD).

---

## 5. Import des Restarchivs (Lauf vom 2026-08-25)

### 5.1 Was im Eingang lag

`Import/` enthielt eine vollständige Kopie von `Alle Dokumente/` — 584 Dateien in 17
Herstellerordnern, davon 81 Dateien, die bereits in `Dokumente/` lagen (AN Bonus, Dewind, Gamesa,
Nordex). Diese 81 Dateien wurden byteweise gegen die vorhandenen Kopien geprüft, sind identisch und
wurden als Dubletten entfernt. Die übrigen **503 Dateien wurden nach `Dokumente/` verschoben**,
Ordnerstruktur unverändert. `Import/` ist wieder leer.

| | |
|---|---|
| Dateien im Eingang | 584 |
| davon Dubletten (identisch, entfernt) | 81 |
| nach `Dokumente/` verschoben | 503 |
| davon PDFs mit Textebene | 198 |
| davon PDFs ohne Textebene (reine Scans) | 244 |
| Office-/Mail-/Bilddateien | 61 |
| neue Quellen | 44 (Id 94–137) |
| neue Konfigurationen | 62 (Id 191–252) |
| neue Positionen | 435 |

### 5.2 Vorgehen bei der Auswertung

1. Von allen 442 neuen PDFs wurde mit `pdftotext -layout` die Textebene gezogen. 244 davon sind
   reine Scans ohne Textebene; ein OCR-Werkzeug stand nicht zur Verfügung, sie wurden **nicht**
   ausgewertet. Das betrifft unter anderem sämtliche Kopien von `Gewichte div. Anlagen.pdf`,
   die Typenprüfungen und die meisten Fundament- und Bewehrungspläne.
2. Aus den textführenden Dokumenten wurden die Gewichtstabellen gelesen. Bevorzugt wurden
   Herstellerblätter der Art „Gewichte und Abmessungen“ / „Weights and Dimensions“ und Packlisten;
   Händlerauskünfte per E-Mail nur dort, wo kein Herstellerbeleg vorlag.
3. `.msg`, `.xls`, `.xlsx`, `.docx` und `.ods` wurden ebenfalls ausgelesen — aus ihnen stammen
   unter anderem die vollständigen Komponentenlisten für REpower MM82/MM92/MD77, Vestas V80 100 m,
   Vestas V44, Enercon E-66 65 m, Enercon E-70 E4 85 m und Schuler SDD 100.
4. Für jede Konfiguration wurden alle Dateien ihres Ordners verknüpft; `Quelle: true` bekommt nur,
   wer in einer der Quellen dieser Konfiguration namentlich geführt ist.

### 5.3 Nicht verwendete Dokumentgattung: Zuwegungs- und Kranstellflächen-Spezifikationen

Die Spezifikationen „Zuwegung und Kranstellfläche“ (Enercon E-82, E-92, E-101, E-115, E-126 EP4,
E-141 EP4, E-53, E-70; GE 2.x/3.2) enthalten Tabellen mit einer Spalte **Gesamtgewicht**. Diese Werte
sind **Gewichte der Transportkombination einschließlich Fahrzeug** (in derselben Tabelle steht z. B.
„Betonmischer < 40,0 t“ und „Rotorblatt außen 59,9 t“) und **keine Bauteilgewichte**. Sie wurden
deshalb bewusst nicht übernommen. Die Dokumente sind als Begleitmaterial verknüpft.

### 5.4 Getroffene Annahmen

1. **Keine Doppelzählung im Turmkopf.** Wo eine Quelle sowohl ein Summengewicht (Gondel gesamt,
   Rotor komplett, Nabe inkl. Blätter) als auch Einzelteile nennt, ist nur eine Ebene als Position
   geführt. Beispiele: Enercon E-40 — „Rotornabe mit Generator und Spinnerkappe ca. 28 t inkl.
   Blätter“ wurde um 3 × 1,0 t Blattmasse auf 25 t reduziert; Kenersys K100 — „Nabe inkl.
   Rotorblätter 56 t“ wurde um 3 × 10,5 t auf 24,5 t reduziert; Vestas V90 — „Rotor 38.000 kg“
   wurde um 3 × 6.660 kg auf 18.020 kg reduziert. Der Originalwortlaut steht jeweils in
   `WertOriginal`, die Rechnung in der `Notiz` der Konfiguration.
2. **Beton mit 2,5 t/m³** in Gewicht umgerechnet, wie bei den Nordex- und Gamesa-Konfigurationen
   (Enercon E-70 E4-Fundament, E-101/E-115-Fundament, E-147 EP5-Fundament). Der Originalwert in m³
   steht in `WertOriginal`.
3. **Enercon E-82: Turm und Flachgründung gemeinsam.** Das Werkstoff-Datenblatt D0708046-0 gibt
   Stahl, Beton, Kupfer, Aluminium, Kunststoff und Beschichtung für „Türme mit Flachgründung“ an;
   eine Trennung zwischen Turm und Fundament ist daraus nicht möglich. Die sechs Werkstoffmengen
   sind deshalb geschlossen unter Kategorie `Turm` geführt und die Konfigurationen 209–212 in
   Anhang F **nicht** als „Fundament: ja“ markiert, obwohl das Fundament im Gewicht enthalten ist.
4. **Enercon E-70 E4/BF/112: 22 Betonsektionen mit zusammen 988 t.** Die Quelle nennt nur die
   Summe; als `EinzelgewichtKg` ist der gerundete Mittelwert 44.909 kg × 22 geführt.
5. **Enercon E-70 E4, Packliste 85 m:** In der Tabelle sind Bezeichnungs- und Zahlenspalten ab
   Zeile 3 um eine Zeile gegeneinander verschoben. Die Zuordnung wurde über die Abmessungen und die
   Summenprobe (366,4 t) rekonstruiert.
6. **Widersprüchliche Quellen** wurden nicht gemittelt: geführt ist jeweils der Herstellerbeleg, die
   abweichende Angabe steht in der `Notiz`. Betroffen sind u. a. Enercon E-58 (GuA-Blatt 133 t
   gegenüber Betriebsanleitung 118 t), Enercon E-66/S/63/3F (GuA 139 t gegenüber Packliste 147 t),
   REpower MM82 (Zeichnung 34,5 t gegenüber Packliste 36 t Kopfsegment), Schuler SDD 100
   (Packliste gegenüber Massenblatt), Fuhrländer FL 1000 (Rückbau-Datenblatt gegenüber
   Händlerauskunft) und Vestas V66 NH 67 m (Händlerauskunft 63 t Gondel gegenüber Datenblatt 57 t).
7. **Enercon E-53:** Die Betonmenge der Flachgründung (672.350 kg) stammt aus dem Werkstoffblatt und
   ist dort für die Turmvariante `3K/03` angegeben, die Konfiguration beschreibt `3K/02`. Der Wert
   ist als Größenordnung zu lesen. Der dort ebenfalls genannte Stahlanteil (99.363 kg) ist **nicht**
   als Position geführt, weil er die bereits erfassten Turmsektionen enthält.
8. **Vestas V44 / V47:** Die Dateien `Transportmaße V44  51 m.xls` und `Transportmaße V47   50 m.xls`
   sind inhaltlich identisch. Die Blattlänge von 21,2 m passt zur V44; eine V47-Konfiguration wurde
   deshalb **nicht** angelegt, die Dateien des V47-Ordners bleiben unverknüpft.
9. **Kenersys K110:** Die Turmsegmenttabelle des Erection Manuals (9/16/21/24/25 t bei NH 95 m) ist
   nicht plausibel — die Masse nimmt nach oben zu — und wurde nicht übernommen. Die Konfiguration
   227 führt nur den Turmkopf.

### 5.5 Nachträglich ergänzte Verknüpfungen an bestehende Konfigurationen

Die Spezifikation `S70-3-Transport Spezifikation-de.pdf` liegt zweimal in den REpower-Ordnern
(`MM92 - 80m` und `REpower MD70_Fuhrländer MD70_Südwind S70 - 85m`), beschreibt aber die Nordex
S70/S77. Beide Kopien wurden den bereits vorhandenen Konfigurationen 119–126 als Begleitmaterial
(`Quelle: false`) zugeordnet — 16 zusätzliche Verknüpfungen. Die Positionen dieser Konfigurationen
wurden **nicht** verändert.

Zur Einordnung: Das Dokument nennt für die S70 NH 85 m einen Turm von 157,3 t (51,8 / 43,1 / 31,4 /
31,0 t), Gondel 56 t, Rotornabe 15 t und Rotorblatt 6 t — die vorhandene Konfiguration 126 führt
dazu nur 4 Positionen mit zusammen 244,8 t. Eine Überarbeitung der Nordex-S70/S77-Konfigurationen
anhand dieser Quelle wäre ein sinnvoller nächster Schritt, war aber nicht Gegenstand dieses Imports.

### 5.6 Neue Hersteller

| Hersteller | Konfigurationen | Ordner |
|---|---|---|
| Enercon | 24 (Id 191–214) | `Dokumente/Enercon/` |
| GE | 10 (Id 215–224) | `Dokumente/GE/` |
| Kenersys | 3 (Id 225–227) | `Dokumente/Kenersys/` |
| NEG Micon | 6 (Id 228–233) | `Dokumente/NEG Micon NM60_1000 - 80m/` |
| REpower | 6 (Id 234–239) | `Dokumente/REpower/` |
| Schuler | 1 (Id 240) | `Dokumente/Schuler SDD100 - 100m/` |
| Fuhrländer | 1 (Id 241) | `Dokumente/Fuhrländer FL1000 - 70m/` |
| Vensys | 1 (Id 242) | `Dokumente/Vensys V62 - 69m/` |
| Vestas | 10 (Id 243–252) | `Dokumente/Vestas/` |

### 5.7 Validierung nach dem Schreiben

Die in Kapitel 4 festgeschriebene Prüfliste wurde nach dem Import erneut durchlaufen — ohne Befund:
JSON gültig, alle Ids eindeutig, `KonfigurationId` durchgängig korrekt, alle `QuelleId`- und
`Kategorie`-Referenzen auflösbar, keine verwaisten Quellen, `Sort` je Konfiguration lückenlos ab 0,
alle 1.084 `Dokumente[].Pfad` vorhanden mit passendem `Name` und `Kb`, kein Pfad absolut, mit `..`,
`\` oder auf `Import/` zeigend. Die Vorversion liegt als `data.json.vor-import.bak`.

---

## Anhang A — Konfigurationsübersicht (Altbestand, Id ≤ 190)

Die Konfigurationen aus dem Import vom 2026-08-25 (Id 191–252) stehen in Anhang F.

`Gewicht` = Σ (`EinzelgewichtKg` × `Stueckzahl`) in Tonnen. `Fund.` = enthält Fundamentpositionen
(vgl. B5 — ohne diese Spalte sind die Gewichte nicht vergleichbar). `Dok.` = Anzahl verknüpfter Dateien.

| Id | Hersteller | Modell | Variante | kW | Rotor m | NH m | Pos. | Gewicht t | Fund. | Quelle | Dok. |
|---|---|---|---|---|---|---|---|---|---|---|---|
| 34 | AN Bonus | 1,0 MW | 50 m | 1000 | 62 | 50 | 2 | 88.0 | nein | 8 | 4 |
| 35 | AN Bonus | 1,0 MW | 60 m | 1000 | 62 | 60 | 2 | 101.0 | nein | 8 | 4 |
| 36 | AN Bonus | 1,0 MW | 70 m | 1000 | 62 | 70 | 2 | 130.0 | nein | 8 | 4 |
| 37 | AN Bonus | 1,3 MW | 43 m Turm | 1300 | 62 | 43 | 6 | 132.3 | nein | 8 | 4 |
| 38 | AN Bonus | 1,3 MW | 45 m Turm | 1300 | 62 | 45 | 6 | 128.9 | nein | 8 | 4 |
| 39 | AN Bonus | 1,3 MW | 48 m Turm (Trafo) | 1300 | 62 | 48 | 6 | 136.3 | nein | 8 | 4 |
| 40 | AN Bonus | 1,3 MW | 49 m Turm (High Gust) | 1300 | 62 | 49 | 6 | 135.9 | nein | 8 | 4 |
| 41 | AN Bonus | 1,3 MW | 49 m Turm | 1300 | 62 | 49 | 6 | 134.4 | nein | 8 | 4 |
| 42 | AN Bonus | 1,3 MW | 60 m Turm | 1300 | 62 | 60 | 6 | 155.6 | nein | 8 | 4 |
| 43 | AN Bonus | 1,3 MW | 68 m Turm (Variante 1) | 1300 | 62 | 68 | 7 | 162.7 | nein | 8 | 4 |
| 44 | AN Bonus | 1,3 MW | 68 m Turm (Variante 2) | 1300 | 62 | 68 | 7 | 174.3 | nein | 8 | 4 |
| 47 | AN Bonus | 1,3 MW | 68 m (Transport) | 1300 | 62 | 68 | 6 | 158.9 | nein | 9 | 4 |
| 45 | AN Bonus | 1,3 MW | 80 m Turm | 1300 | 62 | 80 | 8 | 222.7 | nein | 8 | 4 |
| 48 | AN Bonus | 1,3 MW | 80 m (Transport) | 1300 | 62 | 80 | 7 | 223.0 | nein | 9 | 4 |
| 46 | AN Bonus | 1,3 MW | 89 m Turm | 1300 | 62 | 89 | 9 | 286.6 | nein | 8 | 4 |
| 49 | AN Bonus | 1,3 MW | 90 m (Transport) | 1300 | 62 | 90 | 8 | 285.6 | nein | 9 | 4 |
| 27 | AN Bonus | 600 kW | 35 m | 600 | 44 | 35 | 5 | 58.7 | nein | 6 | 1 |
| 28 | AN Bonus | 600 kW | 40 m | 600 | 44 | 40 | 6 | 67.1 | nein | 6 | 1 |
| 29 | AN Bonus | 600 kW | 45 m | 600 | 44 | 45 | 6 | 72.6 | nein | 6 | 1 |
| 30 | AN Bonus | 600 kW | 50 m | 600 | 44 | 50 | 6 | 78.1 | nein | 6 | 1 |
| 31 | AN Bonus | 600 kW | 55 m | 600 | 44 | 55 | 6 | 86.2 | nein | 6 | 1 |
| 32 | AN Bonus | 600 kW | 58 m | 600 | 44 | 58 | 6 | 90.8 | nein | 7 | 2 |
| 33 | AN Bonus | 600 kW | 60 m | 600 | 44 | 60 | 6 | 93.9 | nein | 6 | 1 |
| 51 | DeWind | D6 | 1250 | 1250 | 62 | 68 | 5 | 37.9 | nein | 10 | 27 |
| 50 | DeWind | D6 | 1000 | 1000 | 62 | 68.5 | 5 | 38.8 | nein | 10 | 27 |
| 186 | DeWind | D6 | 1250 - NH91,5m - Fundament (Flachgruendung) | 1250 | 62 | 91.5 | 2 | 10.4 | ja | 91 | 27 |
| 26 | DeWind | D8 | 2000 | 2000 | 80 | — | 10 | 281.0 | ja | 5 | 1 |
| 189 | Gamesa | G80/2000 | NH 100 m (WZ II / IEC IIA) | 2000 | 80 | 100 | 12 | 1569.1 | ja | 92 | 3 |
| 187 | Gamesa | G80/2000 | NH 60 m (WZ II / IEC IIA) | 2000 | 80 | 60 | 10 | 816.6 | ja | 92 | 4 |
| 188 | Gamesa | G80/2000 | NH 60 m (WZ III / IEC IA) | 2000 | 80 | 60 | 10 | 842.8 | ja | 92 | 4 |
| 115 | Nordex | N100/2500 | — | — | 100 | 100 | 10 | 1904.2 | ja | 47 | 5 |
| 117 | Nordex | N117/2400 | — | — | 117 | 120 | 3 | 1760.4 | ja | 48 | 4 |
| 116 | Nordex | N117/2400 | — | — | 117 | 91 | 3 | 1353.5 | ja | 48 | 4 |
| 118 | Nordex | N117/2400 PH141-B | — | — | 117 | 141 | 2 | 1459.5 | ja | 48 | 4 |
| 113 | Nordex | N80/2500 | — | — | 80 | 100 | 15 | 432.9 | ja | 45 | 5 |
| 114 | Nordex | N90/2500 | — | — | 90 | 80 | 12 | 1126.7 | ja | 46 | 4 |
| 125 | Nordex | S70/1500 | — | — | 70 | 65 | 4 | 187.8 | nein | 50 | 4 |
| 126 | Nordex | S70/1500 | — | — | 70 | 85 | 4 | 244.8 | nein | 50 | 4 |
| 124 | Nordex | S77/1500 | — | — | 77 | 100 | 10 | 369.0 | ja | 50 | 4 |
| 119 | Nordex | S77/1500 | — | — | 77 | 61.5 | 12 | 204.8 | ja | 49 | 4 |
| 120 | Nordex | S77/1500 | — | — | 77 | 70 | 12 | 222.5 | ja | 49 | 4 |
| 121 | Nordex | S77/1500 | — | — | 77 | 80 | 13 | 240.2 | ja | 49 | 4 |
| 122 | Nordex | S77/1500 | — | — | 77 | 85 | 13 | 266.0 | ja | 49 | 4 |
| 123 | Nordex | S77/1500 | — | — | 77 | 90 | 13 | 279.1 | ja | 49 | 4 |

## Anhang B — Kategorie-Abdeckung je Konfiguration (Altbestand, Id ≤ 190)

| Id | Anlage | Maschinenhaus | Generator | Nabe | Rotorblatt | Turm | Fundament | Elektro | Ersatzteil | Sonstiges |
|---|---|---|---|---|---|---|---|---|---|---|
| 26 | DeWind D8 2000 (NH ? m) | X | · | X | X | X | X | X | · | X |
| 27 | AN Bonus 600 kW 35 m (NH 35 m) | X | · | X | X | X | · | · | · | · |
| 28 | AN Bonus 600 kW 40 m (NH 40 m) | X | · | X | X | X | · | · | · | · |
| 29 | AN Bonus 600 kW 45 m (NH 45 m) | X | · | X | X | X | · | · | · | · |
| 30 | AN Bonus 600 kW 50 m (NH 50 m) | X | · | X | X | X | · | · | · | · |
| 31 | AN Bonus 600 kW 55 m (NH 55 m) | X | · | X | X | X | · | · | · | · |
| 32 | AN Bonus 600 kW 58 m (NH 58 m) | X | · | X | X | X | · | · | · | · |
| 33 | AN Bonus 600 kW 60 m (NH 60 m) | X | · | X | X | X | · | · | · | · |
| 34 | AN Bonus 1,0 MW 50 m (NH 50 m) | X | · | · | · | X | · | · | · | · |
| 35 | AN Bonus 1,0 MW 60 m (NH 60 m) | X | · | · | · | X | · | · | · | · |
| 36 | AN Bonus 1,0 MW 70 m (NH 70 m) | X | · | · | · | X | · | · | · | · |
| 37 | AN Bonus 1,3 MW 43 m Turm (NH 43 m) | X | · | X | X | X | · | · | · | · |
| 38 | AN Bonus 1,3 MW 45 m Turm (NH 45 m) | X | · | X | X | X | · | · | · | · |
| 39 | AN Bonus 1,3 MW 48 m Turm (Trafo) (NH 48 m) | X | · | X | X | X | · | · | · | · |
| 40 | AN Bonus 1,3 MW 49 m Turm (High Gust) (NH 49 m) | X | · | X | X | X | · | · | · | · |
| 41 | AN Bonus 1,3 MW 49 m Turm (NH 49 m) | X | · | X | X | X | · | · | · | · |
| 42 | AN Bonus 1,3 MW 60 m Turm (NH 60 m) | X | · | X | X | X | · | · | · | · |
| 43 | AN Bonus 1,3 MW 68 m Turm (Variante 1) (NH 68 m) | X | · | X | X | X | · | · | · | · |
| 44 | AN Bonus 1,3 MW 68 m Turm (Variante 2) (NH 68 m) | X | · | X | X | X | · | · | · | · |
| 45 | AN Bonus 1,3 MW 80 m Turm (NH 80 m) | X | · | X | X | X | · | · | · | · |
| 46 | AN Bonus 1,3 MW 89 m Turm (NH 89 m) | X | · | X | X | X | · | · | · | · |
| 47 | AN Bonus 1,3 MW 68 m (Transport) (NH 68 m) | X | · | X | X | X | · | · | · | · |
| 48 | AN Bonus 1,3 MW 80 m (Transport) (NH 80 m) | X | · | X | X | X | · | · | · | · |
| 49 | AN Bonus 1,3 MW 90 m (Transport) (NH 90 m) | X | · | X | X | X | · | · | · | · |
| 50 | DeWind D6 1000 (NH 68.5 m) | · | X | X | X | · | · | · | · | X |
| 51 | DeWind D6 1250 (NH 68 m) | · | X | X | X | · | · | · | · | X |
| 113 | Nordex N80/2500  (NH 100 m) | X | · | X | X | X | X | X | · | X |
| 114 | Nordex N90/2500  (NH 80 m) | X | · | X | X | X | X | X | · | X |
| 115 | Nordex N100/2500  (NH 100 m) | X | X | X | X | X | X | X | · | · |
| 116 | Nordex N117/2400  (NH 91 m) | · | · | · | · | · | X | · | · | · |
| 117 | Nordex N117/2400  (NH 120 m) | · | · | · | · | · | X | · | · | · |
| 118 | Nordex N117/2400 PH141-B  (NH 141 m) | · | · | · | · | · | X | · | · | · |
| 119 | Nordex S77/1500  (NH 61.5 m) | X | · | X | X | X | X | X | · | · |
| 120 | Nordex S77/1500  (NH 70 m) | X | · | X | X | X | X | X | · | · |
| 121 | Nordex S77/1500  (NH 80 m) | X | · | X | X | X | X | X | · | · |
| 122 | Nordex S77/1500  (NH 85 m) | X | · | X | X | X | X | X | · | · |
| 123 | Nordex S77/1500  (NH 90 m) | X | · | X | X | X | X | X | · | · |
| 124 | Nordex S77/1500  (NH 100 m) | X | · | X | X | X | X | X | · | · |
| 125 | Nordex S70/1500  (NH 65 m) | X | · | X | X | X | · | · | · | · |
| 126 | Nordex S70/1500  (NH 85 m) | X | · | X | X | X | · | · | · | · |
| 186 | DeWind D6 1250 - NH91,5m - Fundament (Flachgruendung) (NH 91.5 m) | · | · | · | · | · | X | · | · | · |
| 187 | Gamesa G80/2000 NH 60 m (WZ II / IEC IIA) (NH 60 m) | X | · | X | X | X | X | · | · | · |
| 188 | Gamesa G80/2000 NH 60 m (WZ III / IEC IA) (NH 60 m) | X | · | X | X | X | X | · | · | · |
| 189 | Gamesa G80/2000 NH 100 m (WZ II / IEC IIA) (NH 100 m) | X | · | X | X | X | X | · | · | · |

## Anhang C — Dokumente ohne Belegfunktion (Altbestand: 59 von 81)

Dateien ohne `"Quelle": true`. Die mit **nicht in `Dokumente[]` gelistet** markierten Einträge
(15 Stück) sind aus der Anwendung heraus überhaupt nicht erreichbar.

- [`Dokumente/AN Bonus/AN Bonus 0,6_44-58m/Übersichtszeichnung AN Bonus 44-58m.pdf`](Dokumente/AN%20Bonus/AN%20Bonus%200%2C6_44-58m/U%CC%88bersichtszeichnung%20AN%20Bonus%2044-58m.pdf)
- [`Dokumente/AN Bonus/An Bonus 1-1_3 mw.msg`](Dokumente/AN%20Bonus/An%20Bonus%201-1_3%20mw.msg)
- [`Dokumente/AN Bonus/Re_ An Bonus 1-1_3 mw.msg`](Dokumente/AN%20Bonus/Re_%20An%20Bonus%201-1_3%20mw.msg)
- [`Dokumente/Dewind/Dewind D6 - 1000/Azimutträger Zeichnung DeWind 62.pdf`](Dokumente/Dewind/Dewind%20D6%20-%201000/Azimuttra%CC%88ger%20Zeichnung%20DeWind%2062.pdf)
- [`Dokumente/Dewind/Dewind D6 - 1000/Fundamenteinbauteil D6-1000-NH 68,5m.pdf`](Dokumente/Dewind/Dewind%20D6%20-%201000/Fundamenteinbauteil%20D6-1000-NH%2068%2C5m.pdf)
- [`Dokumente/Dewind/Dewind D6 - 1000/Fundamentzeichnung D6 -Scheid-.pdf`](Dokumente/Dewind/Dewind%20D6%20-%201000/Fundamentzeichnung%20D6%20-Scheid-.pdf)
- [`Dokumente/Dewind/Dewind D6 - 1000/Gondelverkleidung - Kabine für Windenergieanlage.pdf`](Dokumente/Dewind/Dewind%20D6%20-%201000/Gondelverkleidung%20-%20Kabine%20fu%CC%88r%20Windenergieanlage.pdf)
- [`Dokumente/Dewind/Dewind D6 - 1000/Montagezeichnung Rotornabe komplett D6 1000.pdf`](Dokumente/Dewind/Dewind%20D6%20-%201000/Montagezeichnung%20Rotornabe%20komplett%20D6%201000.pdf)
- [`Dokumente/Dewind/Dewind D6 - 1000/Montagezeichnung Rotornabe komplett.pdf`](Dokumente/Dewind/Dewind%20D6%20-%201000/Montagezeichnung%20Rotornabe%20komplett.pdf)
- [`Dokumente/Dewind/Dewind D6 - 1000/Rotorwelle.pdf`](Dokumente/Dewind/Dewind%20D6%20-%201000/Rotorwelle.pdf)
- [`Dokumente/Dewind/Dewind D6 - 1000/Zeichnung Maschinenträger Hinterteil D6 1000.pdf`](Dokumente/Dewind/Dewind%20D6%20-%201000/Zeichnung%20Maschinentra%CC%88ger%20Hinterteil%20D6%201000.pdf)
- [`Dokumente/Dewind/Dewind D6 - 1000/Zeichnung Turm D6 1000kw 62-68,5m.pdf`](Dokumente/Dewind/Dewind%20D6%20-%201000/Zeichnung%20Turm%20D6%201000kw%2062-68%2C5m.pdf)
- [`Dokumente/Dewind/Dewind D6 -1250/Azimutträger Zeichnung DeWind 62.pdf`](Dokumente/Dewind/Dewind%20D6%20-1250/Azimuttra%CC%88ger%20Zeichnung%20DeWind%2062.pdf)
- [`Dokumente/Dewind/Dewind D6 -1250/Fundament D6-1250kW 64-91,5m/Arbeitsablauf Herstellung Fundament - Zeichnung Blatt 1d.pdf`](Dokumente/Dewind/Dewind%20D6%20-1250/Fundament%20D6-1250kW%2064-91%2C5m/Arbeitsablauf%20Herstellung%20Fundament%20-%20Zeichnung%20Blatt%201d.pdf)
- [`Dokumente/Dewind/Dewind D6 -1250/Fundament D6-1250kW 64-91,5m/Arbeitsablauf Herstellung Fundament - Zeichnung Blatt 1e.pdf`](Dokumente/Dewind/Dewind%20D6%20-1250/Fundament%20D6-1250kW%2064-91%2C5m/Arbeitsablauf%20Herstellung%20Fundament%20-%20Zeichnung%20Blatt%201e.pdf)
- [`Dokumente/Dewind/Dewind D6 -1250/Fundament D6-1250kW 64-91,5m/Fundament Schalplan 2b.pdf`](Dokumente/Dewind/Dewind%20D6%20-1250/Fundament%20D6-1250kW%2064-91%2C5m/Fundament%20Schalplan%202b.pdf)
- [`Dokumente/Dewind/Dewind D6 -1250/Fundament D6-1250kW 64-91,5m/Schalplan Fundament 2a.pdf`](Dokumente/Dewind/Dewind%20D6%20-1250/Fundament%20D6-1250kW%2064-91%2C5m/Schalplan%20Fundament%202a.pdf)
- [`Dokumente/Dewind/Dewind D6 -1250/Fundament D6-1250kW 64-91,5m/Zeichnung Bewehrungsplan Blatt 3a.pdf`](Dokumente/Dewind/Dewind%20D6%20-1250/Fundament%20D6-1250kW%2064-91%2C5m/Zeichnung%20Bewehrungsplan%20Blatt%203a.pdf)
- [`Dokumente/Dewind/Dewind D6 -1250/Fundament D6-1250kW 64-91,5m/Zeichnung Bewehrungsplan Blatt 3b.pdf`](Dokumente/Dewind/Dewind%20D6%20-1250/Fundament%20D6-1250kW%2064-91%2C5m/Zeichnung%20Bewehrungsplan%20Blatt%203b.pdf)
- [`Dokumente/Dewind/Dewind D6 -1250/Fundament D6-1250kW 64-91,5m/Zeichnung Bewehrungsplan Fundament Blatt 4d.pdf`](Dokumente/Dewind/Dewind%20D6%20-1250/Fundament%20D6-1250kW%2064-91%2C5m/Zeichnung%20Bewehrungsplan%20Fundament%20Blatt%204d.pdf)
- [`Dokumente/Dewind/Dewind D6 -1250/Fundament D6-1250kW 64-91,5m/Zeichnung Fundamentring Blatt 1d.pdf`](Dokumente/Dewind/Dewind%20D6%20-1250/Fundament%20D6-1250kW%2064-91%2C5m/Zeichnung%20Fundamentring%20Blatt%201d.pdf)
- [`Dokumente/Dewind/Dewind D6 -1250/Fundament D6-1250kW 64-91,5m/Zeichnung Fundamentring Blatt 1e.pdf`](Dokumente/Dewind/Dewind%20D6%20-1250/Fundament%20D6-1250kW%2064-91%2C5m/Zeichnung%20Fundamentring%20Blatt%201e.pdf)
- [`Dokumente/Dewind/Dewind D6 -1250/Fundamentzeichnung D6 -Scheid-.pdf`](Dokumente/Dewind/Dewind%20D6%20-1250/Fundamentzeichnung%20D6%20-Scheid-.pdf)
- [`Dokumente/Dewind/Dewind D6 -1250/Gondelverkleidung - Kabine für Windenergieanlage1.pdf`](Dokumente/Dewind/Dewind%20D6%20-1250/Gondelverkleidung%20-%20Kabine%20fu%CC%88r%20Windenergieanlage1.pdf)
- [`Dokumente/Dewind/Dewind D6 -1250/Montagezeichnung Rotornabe komplett.pdf`](Dokumente/Dewind/Dewind%20D6%20-1250/Montagezeichnung%20Rotornabe%20komplett.pdf)
- [`Dokumente/Dewind/Dewind D6 -1250/Rotorwelle.pdf`](Dokumente/Dewind/Dewind%20D6%20-1250/Rotorwelle.pdf)
- [`Dokumente/Dewind/Dewind D6 -1250/Zeichnung Leerrohrführung D6 1250kW -68m NH.pdf`](Dokumente/Dewind/Dewind%20D6%20-1250/Zeichnung%20Leerrohrfu%CC%88hrung%20D6%201250kW%20-68m%20NH.pdf)
- [`Dokumente/Gamesa/G80 - 100m/Wirfus_Auszug_Typenprüfung.pdf`](Dokumente/Gamesa/G80%20-%20100m/Wirfus_Auszug_Typenpru%CC%88fung.pdf)
- [`Dokumente/Gamesa/G80 - 60m/Packing list Re-Powering 5 x V80_2MW_60m(1).xls`](Dokumente/Gamesa/G80%20-%2060m/Packing%20list%20Re-Powering%205%20x%20V80_2MW_60m%281%29.xls)
- [`Dokumente/Gamesa/G80 - 60m/Re_ Weights of G80 with 60m HH.msg`](Dokumente/Gamesa/G80%20-%2060m/Re_%20Weights%20of%20G80%20with%2060m%20HH.msg)
- [`Dokumente/Nordex/Nordex N100/K0801_010855_DE_R02_Overview_N100_R100.pdf`](Dokumente/Nordex/Nordex%20N100/K0801_010855_DE_R02_Overview_N100_R100.pdf)
- [`Dokumente/Nordex/Nordex N100/K0801_010861_DE_R02_Re_Build.pdf`](Dokumente/Nordex/Nordex%20N100/K0801_010861_DE_R02_Re_Build.pdf)
- [`Dokumente/Nordex/Nordex N100/K0811_017864_DE_R03_TechnBeschreibng_MS-Anlage_WEA_TiT.pdf`](Dokumente/Nordex/Nordex%20N100/K0811_017864_DE_R03_TechnBeschreibng_MS-Anlage_WEA_TiT.pdf)
- [`Dokumente/Nordex/Nordex N117-2400 Hambuch/K0801_011803_DE_R10_Transport_Zuwegung_Krananforderungen_K08g.pdf`](Dokumente/Nordex/Nordex%20N117-2400%20Hambuch/K0801_011803_DE_R10_Transport_Zuwegung_Krananforderungen_K08g.pdf)
- [`Dokumente/Nordex/Nordex N117_2400/03_K0801_011803_DE_R10_Transport_Zuwegung_Krananforderungen_K08g.pdf`](Dokumente/Nordex/Nordex%20N117_2400/03_K0801_011803_DE_R10_Transport_Zuwegung_Krananforderungen_K08g.pdf)
- [`Dokumente/Nordex/Nordex N117_2400/Spezifikation WEA Hersteller_Fundamente.pdf`](Dokumente/Nordex/Nordex%20N117_2400/Spezifikation%20WEA%20Hersteller_Fundamente.pdf)
- [`Dokumente/Nordex/Nordex N62 -69m/01 Zeichnung.pdf`](Dokumente/Nordex/Nordex%20N62%20-69m/01%20Zeichnung.pdf)  — **nicht in `Dokumente[]` gelistet**
- [`Dokumente/Nordex/Nordex N62 -69m/02 Zeichnung.pdf`](Dokumente/Nordex/Nordex%20N62%20-69m/02%20Zeichnung.pdf)  — **nicht in `Dokumente[]` gelistet**
- [`Dokumente/Nordex/Nordex N62 -69m/03 Zeichnung.pdf`](Dokumente/Nordex/Nordex%20N62%20-69m/03%20Zeichnung.pdf)  — **nicht in `Dokumente[]` gelistet**
- [`Dokumente/Nordex/Nordex N62 -69m/1998-05-11 Prüfbericht zur TP in statischer Hinsicht.pdf`](Dokumente/Nordex/Nordex%20N62%20-69m/1998-05-11%20Pru%CC%88fbericht%20zur%20TP%20in%20statischer%20Hinsicht.pdf)  — **nicht in `Dokumente[]` gelistet**
- [`Dokumente/Nordex/Nordex N62 -69m/2000-06-21 PB 71209-7 Nordex N62 Stahlvollwandturm und Flachfundament.pdf`](Dokumente/Nordex/Nordex%20N62%20-69m/2000-06-21%20PB%2071209-7%20Nordex%20N62%20Stahlvollwandturm%20und%20Flachfundament.pdf)  — **nicht in `Dokumente[]` gelistet**
- [`Dokumente/Nordex/Nordex N62 -69m/2000-06-29 PB II B 3-543-635 Turm NH 69m.pdf`](Dokumente/Nordex/Nordex%20N62%20-69m/2000-06-29%20PB%20II%20B%203-543-635%20Turm%20NH%2069m.pdf)  — **nicht in `Dokumente[]` gelistet**
- [`Dokumente/Nordex/Nordex N62 -69m/2000-06-29 PB II B 3-543-635 Turm NH 69m_Kapitel 8_Austauschseite.pdf`](Dokumente/Nordex/Nordex%20N62%20-69m/2000-06-29%20PB%20II%20B%203-543-635%20Turm%20NH%2069m_Kapitel%208_Austauschseite.pdf)  — **nicht in `Dokumente[]` gelistet**
- [`Dokumente/Nordex/Nordex N62 -69m/20200414_WindGuart_BE_NK-0092_NK01_Maschinengutachten.pdf`](Dokumente/Nordex/Nordex%20N62%20-69m/20200414_WindGuart_BE_NK-0092_NK01_Maschinengutachten.pdf)  — **nicht in `Dokumente[]` gelistet**
- [`Dokumente/Nordex/Nordex N62 -69m/20200414_WindGuart_BE_NK-0092_NK01_Rotorblattgutachten.pdf`](Dokumente/Nordex/Nordex%20N62%20-69m/20200414_WindGuart_BE_NK-0092_NK01_Rotorblattgutachten.pdf)  — **nicht in `Dokumente[]` gelistet**
- [`Dokumente/Nordex/Nordex N62 -69m/20200526_WINDGUARD-Certification_BE_NK-0041_NK01_Gutachten-Weiterbetrieb-Stellungnahme.pdf`](Dokumente/Nordex/Nordex%20N62%20-69m/20200526_WINDGUARD-Certification_BE_NK-0041_NK01_Gutachten-Weiterbetrieb-Stellungnahme.pdf)  — **nicht in `Dokumente[]` gelistet**
- [`Dokumente/Nordex/Nordex N62 -69m/20220927_P&S_BE_NK-0092_NK01_Protokoll-Sachkundeprüfung.pdf`](Dokumente/Nordex/Nordex%20N62%20-69m/20220927_P%26S_BE_NK-0092_NK01_Protokoll-Sachkundepru%CC%88fung.pdf)  — **nicht in `Dokumente[]` gelistet**
- [`Dokumente/Nordex/Nordex N62 -69m/210621_1205_T3_KD_T3.pdf`](Dokumente/Nordex/Nordex%20N62%20-69m/210621_1205_T3_KD_T3.pdf)  — **nicht in `Dokumente[]` gelistet**
- [`Dokumente/Nordex/Nordex N62 -69m/210708_1205_T3_KD_Trafo.pdf`](Dokumente/Nordex/Nordex%20N62%20-69m/210708_1205_T3_KD_Trafo.pdf)  — **nicht in `Dokumente[]` gelistet**
- [`Dokumente/Nordex/Nordex N62 -69m/211103_1205_T2_KD_T2.pdf`](Dokumente/Nordex/Nordex%20N62%20-69m/211103_1205_T2_KD_T2.pdf)  — **nicht in `Dokumente[]` gelistet**
- [`Dokumente/Nordex/Nordex N62 -69m/ZW19039.03.01_Zertifikat_NX1205.pdf`](Dokumente/Nordex/Nordex%20N62%20-69m/ZW19039.03.01_Zertifikat_NX1205.pdf)  — **nicht in `Dokumente[]` gelistet**
- [`Dokumente/Nordex/Nordex N80 - 100m/Abmaße Rotorblatt - 2.jpg`](Dokumente/Nordex/Nordex%20N80%20-%20100m/Abma%C3%9Fe%20Rotorblatt%20-%202.jpg)
- [`Dokumente/Nordex/Nordex N80 - 100m/Abmaße Rotorblatt -1.jpg`](Dokumente/Nordex/Nordex%20N80%20-%20100m/Abma%C3%9Fe%20Rotorblatt%20-1.jpg)
- [`Dokumente/Nordex/Nordex N80 - 100m/K0801_010861_DE_R02_Re_Build.pdf`](Dokumente/Nordex/Nordex%20N80%20-%20100m/K0801_010861_DE_R02_Re_Build.pdf)
- [`Dokumente/Nordex/Nordex N80 - 100m/K0801_025550_DE_R00_Vertriebsdoku_Rueckbauaufwand_K08g.pdf`](Dokumente/Nordex/Nordex%20N80%20-%20100m/K0801_025550_DE_R00_Vertriebsdoku_Rueckbauaufwand_K08g.pdf)
- [`Dokumente/Nordex/Nordex N90 -80m/K0801-014863-EN-R00-Erection-K08-book.pdf`](Dokumente/Nordex/Nordex%20N90%20-80m/K0801-014863-EN-R00-Erection-K08-book.pdf)
- [`Dokumente/Nordex/Nordex N90 -80m/Nordex_tower N90 -80 Meter.pdf`](Dokumente/Nordex/Nordex%20N90%20-80m/Nordex_tower%20N90%20-80%20Meter.pdf)
- [`Dokumente/Nordex/Nordex S77 - S70/Datenblatt Turm.jpg`](Dokumente/Nordex/Nordex%20S77%20-%20S70/Datenblatt%20Turm.jpg)
- [`Dokumente/Nordex/Nordex S77 - S70/S77/GA Nr. 0823.S77.03.12 Anl. 2.pdf`](Dokumente/Nordex/Nordex%20S77%20-%20S70/S77/GA%20Nr.%200823.S77.03.12%20Anl.%202.pdf)

## Anhang D — Quellenverzeichnis mit aufgelösten Pfaden (Altbestand, Quellen 5–93)

„Status" bezieht sich auf die Auflösbarkeit von `Quellen.Dateiname` allein (vgl. B2); die Pfadspalte
stammt aus `Konfigurationen[].Dokumente` mit `"Quelle": true`.

| Quelle | Dokumenttyp | Datum | Vertrauen | Konfigurationen | Status Dateiname | Pfad(e) relativ zur `data.json` |
|---|---|---|---|---|---|---|
| 5 | Transportangaben (Sizes and Weights) | 2006-08-01 | hoch | 26 | eindeutig | [`Dokumente/Dewind/DeWind D8/D8-2000-410-ger-Transportangaben.pdf`](Dokumente/Dewind/DeWind%20D8/D8-2000-410-ger-Transportangaben.pdf) |
| 6 | Service-/Wartungshandbuch (SWT-0.6-44 Mk IV, ZSM 511034) | — | mittel | 27, 28, 29, 30, 31, 33 | mehrdeutig | [`Dokumente/AN Bonus/AN Bonus 0,6_44-35m/Turmgewichte-600 kw.pdf`](Dokumente/AN%20Bonus/AN%20Bonus%200%2C6_44-35m/Turmgewichte-600%20kw.pdf)<br>[`Dokumente/AN Bonus/AN Bonus 0,6_44-40m/Turmgewichte-600 kw.pdf`](Dokumente/AN%20Bonus/AN%20Bonus%200%2C6_44-40m/Turmgewichte-600%20kw.pdf)<br>[`Dokumente/AN Bonus/AN Bonus 0,6_44-45m/Turmgewichte-600 kw.pdf`](Dokumente/AN%20Bonus/AN%20Bonus%200%2C6_44-45m/Turmgewichte-600%20kw.pdf)<br>[`Dokumente/AN Bonus/AN Bonus 0,6_44-50m/Turmgewichte-600 kw.pdf`](Dokumente/AN%20Bonus/AN%20Bonus%200%2C6_44-50m/Turmgewichte-600%20kw.pdf)<br>[`Dokumente/AN Bonus/An Bonus 0,6_44-55m/Turmgewichte-600 kw.pdf`](Dokumente/AN%20Bonus/An%20Bonus%200%2C6_44-55m/Turmgewichte-600%20kw.pdf)<br>[`Dokumente/AN Bonus/An Bonus 0,6_44-60m/Turmgewichte-600 kw.pdf`](Dokumente/AN%20Bonus/An%20Bonus%200%2C6_44-60m/Turmgewichte-600%20kw.pdf) |
| 7 | Service-/Wartungshandbuch (SWT-0.6-44 Mk IV, ZSM 511034) | — | mittel | 32 | eindeutig | [`Dokumente/AN Bonus/AN Bonus 0,6_44-58m/Turmgewichte-600 kw 44-58m.pdf`](Dokumente/AN%20Bonus/AN%20Bonus%200%2C6_44-58m/Turmgewichte-600%20kw%2044-58m.pdf) |
| 8 | Bedienungshandbuch (AN Bonus 1 MW, BV 513773) | — | hoch | 34, 35, 36, 37, 38, 39, 40, 41, 42, 43, 44, 45, 46 | eindeutig | [`Dokumente/AN Bonus/20160113 - Maße und Gewichte 1 und 13 MW AN BONUS.pdf`](Dokumente/AN%20Bonus/20160113%20-%20Ma%C3%9Fe%20und%20Gewichte%201%20und%2013%20MW%20AN%20BONUS.pdf) |
| 9 | Transport-, Strassen- und Krananforderungen (1,3 MW/62) | — | mittel | 47, 48, 49 | eindeutig | [`Dokumente/AN Bonus/AN BONUS 13 MW-62 Transport- Strassen- und Krananforderungen_TB.pdf`](Dokumente/AN%20Bonus/AN%20BONUS%2013%20MW-62%20Transport-%20Strassen-%20und%20Krananforderungen_TB.pdf) |
| 10 | Betriebsanleitung (Technische Daten) | — | mittel | 50, 51 | mehrdeutig | [`Dokumente/Dewind/Dewind D6 - 1000/Betriebsanleitung D6 1000- 1250KW.pdf`](Dokumente/Dewind/Dewind%20D6%20-%201000/Betriebsanleitung%20D6%201000-%201250KW.pdf)<br>[`Dokumente/Dewind/Dewind D6 -1250/Betriebsanleitung D6 1000- 1250KW.pdf`](Dokumente/Dewind/Dewind%20D6%20-1250/Betriebsanleitung%20D6%201000-%201250KW.pdf) |
| 45 | — | — | mittel | 113 | mehrdeutig | [`Dokumente/Nordex/Nordex N80 - 100m/N80 transport data.pdf`](Dokumente/Nordex/Nordex%20N80%20-%20100m/N80%20transport%20data.pdf) |
| 46 | — | — | mittel | 114 | mehrdeutig | [`Dokumente/Nordex/Nordex N100/K0801_025550_DE_R00_Vertriebsdoku_Rueckbauaufwand_K08g.pdf`](Dokumente/Nordex/Nordex%20N100/K0801_025550_DE_R00_Vertriebsdoku_Rueckbauaufwand_K08g.pdf)<br>[`Dokumente/Nordex/Nordex N90 -80m/N80 transport data.pdf`](Dokumente/Nordex/Nordex%20N90%20-80m/N80%20transport%20data.pdf) |
| 47 | — | — | mittel | 115 | mehrdeutig | [`Dokumente/Nordex/Nordex N100/K0801_010868_DE_R06_Vertriebsdoku_Techn_Beschreibg_N100.pdf`](Dokumente/Nordex/Nordex%20N100/K0801_010868_DE_R06_Vertriebsdoku_Techn_Beschreibg_N100.pdf)<br>[`Dokumente/Nordex/Nordex N100/K0801_025550_DE_R00_Vertriebsdoku_Rueckbauaufwand_K08g.pdf`](Dokumente/Nordex/Nordex%20N100/K0801_025550_DE_R00_Vertriebsdoku_Rueckbauaufwand_K08g.pdf) |
| 48 | — | — | mittel | 116, 117, 118 | eindeutig | [`Dokumente/Nordex/Nordex N117-2400 Hambuch/04_K0801_033496_DE_R05_Fundamente_N117.pdf`](Dokumente/Nordex/Nordex%20N117-2400%20Hambuch/04_K0801_033496_DE_R05_Fundamente_N117.pdf) |
| 49 | — | — | mittel | 119, 120, 121, 122, 123 | eindeutig | [`Dokumente/Nordex/Nordex S77 - S70/S77/K0701_008769_DE_R04_Transport.pdf`](Dokumente/Nordex/Nordex%20S77%20-%20S70/S77/K0701_008769_DE_R04_Transport.pdf)<br>[`Dokumente/Nordex/Nordex S77 - S70/S77/S70-1-techn-description-de.pdf`](Dokumente/Nordex/Nordex%20S77%20-%20S70/S77/S70-1-techn-description-de.pdf) |
| 50 | — | — | mittel | 124, 125, 126 | eindeutig | [`Dokumente/Nordex/Nordex S77 - S70/S77/S70-1-techn-description-de.pdf`](Dokumente/Nordex/Nordex%20S77%20-%20S70/S77/S70-1-techn-description-de.pdf) |
| 91 | Rundstahl-Stueckliste (Bewehrungsplan Fundament) | — | hoch | 186 | fehlt | [`Dokumente/Dewind/Dewind D6 -1250/Fundament D6-1250kW 64-91,5m/Zeichnung Bewehrungsplan Fundament Blatt 4c.pdf`](Dokumente/Dewind/Dewind%20D6%20-1250/Fundament%20D6-1250kW%2064-91%2C5m/Zeichnung%20Bewehrungsplan%20Fundament%20Blatt%204c.pdf) |
| 92 | Technisches Datenblatt GD005900 Rev. 3 (Allgemeine Beschreibung und Spezifikation G80) | 2005-11-16 | hoch | 187, 188, 189 | eindeutig | [`Dokumente/Gamesa/G80 - 100m/Gamesa G80.pdf`](Dokumente/Gamesa/G80%20-%20100m/Gamesa%20G80.pdf)<br>[`Dokumente/Gamesa/Maße und Gewichte WEA G80.pdf`](Dokumente/Gamesa/Ma%C3%9Fe%20und%20Gewichte%20WEA%20G80.pdf) |
| 93 | Maße und Gewichte (G80 WTG T. 60-78-100 m, Transporteinheiten) | — | hoch | — | eindeutig | [`Dokumente/Gamesa/G80 - 100m/Gamesa G80.pdf`](Dokumente/Gamesa/G80%20-%20100m/Gamesa%20G80.pdf)<br>[`Dokumente/Gamesa/Maße und Gewichte WEA G80.pdf`](Dokumente/Gamesa/Ma%C3%9Fe%20und%20Gewichte%20WEA%20G80.pdf) |

## Anhang E — Positionen der neuen Gamesa-Konfigurationen

| Konf. | Sort | Kategorie | Bezeichnung | Stk | kg/Stk | WertOriginal | L × B × H [m] | Quelle |
|---|---|---|---|---|---|---|---|---|
| 187 | 0 | Maschinenhaus | Gondel komplett (Transporteinheit. ohne Rotor) | 1 | 68.000 | 68.000 kg | 10.10 × 3.30 × 4.35 | 93 |
| 187 | 1 | Nabe | Nabe mit Nabenabdeckung (Transporteinheit Nose) | 1 | 19.000 | 19.000 kg | 3.30 × 3.30 × 3.46 | 93 |
| 187 | 2 | Rotorblatt | Rotorblatt (39 m. GFK-Epoxid) | 3 | 6.719 | 6.719 kg pro Blatt | 39.00 × 3.36 | 92 |
| 187 | 3 | Turm | Turmsektion 1 (unten) | 1 | 32.000 | 32.000 kg (WZ II / IEC IIA) | 10.391 × 4.034 | 92 |
| 187 | 4 | Turm | Turmsektion 2 (Mitte) | 1 | 51.079 | 51.079 kg (WZ II / IEC IIA) | 23.822 × 3.490 | 92 |
| 187 | 5 | Turm | Turmsektion 3 (oben) | 1 | 39.415 | 39.415 kg (WZ II / IEC IIA) | 24.367 × 2.780 | 92 |
| 187 | 6 | Fundament | Fundamentsektion / Fundamentring (Flachgruendung. Einbauteil) | 1 | 12.000 | 12.000 kg | 4.44 × 4.44 × 2.20 | 93 |
| 187 | 7 | Fundament | Fundamentkoerper Beton C30/37 (IEC IIA) | 1 | 510.000 | 204 m3 (x 2.5 t/m3) | 12.80 × 1.50 | 92 |
| 187 | 8 | Fundament | Sauberkeitsschicht Beton C8/10 (IEC IIA) | 1 | 41.000 | 16.4 m3 (x 2.5 t/m3) | — | 92 |
| 187 | 9 | Fundament | Betonstahl BSt 500 S/M (IEC IIA) | 1 | 23.900 | 23.9 to | — | 92 |
| 188 | 0 | Maschinenhaus | Gondel komplett (Transporteinheit. ohne Rotor) | 1 | 68.000 | 68.000 kg | 10.10 × 3.30 × 4.35 | 93 |
| 188 | 1 | Nabe | Nabe mit Nabenabdeckung (Transporteinheit Nose) | 1 | 19.000 | 19.000 kg | 3.30 × 3.30 × 3.46 | 93 |
| 188 | 2 | Rotorblatt | Rotorblatt (39 m. GFK-Epoxid) | 3 | 6.719 | 6.719 kg pro Blatt | 39.00 × 3.36 | 92 |
| 188 | 3 | Turm | Turmsektion 1 (unten) | 1 | 35.700 | 35.700 kg (WZ III / IEC IA) | 10.391 × 4.034 | 92 |
| 188 | 4 | Turm | Turmsektion 2 (Mitte) | 1 | 55.400 | 55.400 kg (WZ III / IEC IA) | 23.822 × 3.492 | 92 |
| 188 | 5 | Turm | Turmsektion 3 (oben) | 1 | 43.000 | 43.000 kg (WZ III / IEC IA) | 24.367 × 2.781 | 92 |
| 188 | 6 | Fundament | Fundamentsektion / Fundamentring (Flachgruendung. Einbauteil) | 1 | 12.000 | 12.000 kg | 4.44 × 4.44 × 2.20 | 93 |
| 188 | 7 | Fundament | Fundamentkoerper Beton C35/45 (DIBt WZ III) | 1 | 530.000 | 212 m3 (x 2.5 t/m3) | 12.50 × 1.90 | 92 |
| 188 | 8 | Fundament | Sauberkeitsschicht Beton C8/10 (DIBt WZ III) | 1 | 33.500 | 13.4 m3 (x 2.5 t/m3) | — | 92 |
| 188 | 9 | Fundament | Betonstahl BSt 500 S/M (DIBt WZ III) | 1 | 26.000 | 26.0 to | — | 92 |
| 189 | 0 | Maschinenhaus | Gondel komplett (Transporteinheit. ohne Rotor) | 1 | 68.000 | 68.000 kg | 10.10 × 3.30 × 4.35 | 93 |
| 189 | 1 | Nabe | Nabe mit Nabenabdeckung (Transporteinheit Nose) | 1 | 19.000 | 19.000 kg | 3.30 × 3.30 × 3.46 | 93 |
| 189 | 2 | Rotorblatt | Rotorblatt (39 m. GFK-Epoxid) | 3 | 6.719 | 6.719 kg pro Blatt | 39.00 × 3.36 | 92 |
| 189 | 3 | Turm | Turmsektion 1 (unten) | 1 | 62.800 | 62.800 kg (WZ II / IEC IIA) | 15.609 × 4.037 | 92 |
| 189 | 4 | Turm | Turmsektion 2 | 1 | 61.300 | 61.300 kg (WZ II / IEC IIA) | 16.961 × 3.855 | 92 |
| 189 | 5 | Turm | Turmsektion 3 | 1 | 55.195 | 55.195 kg (WZ II / IEC IIA) | 16.980 × 3.810 | 92 |
| 189 | 6 | Turm | Turmsektion 4 | 1 | 55.500 | 55.500 kg (WZ II / IEC IIA) | 23.822 × 3.494 | 92 |
| 189 | 7 | Turm | Turmsektion 5 (oben) | 1 | 51.000 | 51.000 kg (WZ II / IEC IIA) | 24.368 × 2.781 | 92 |
| 189 | 8 | Fundament | Fundamentsektion / Fundamentring (Flachgruendung. Einbauteil) | 1 | 23.000 | 23.000 kg | 4.50 × 4.50 × 3.20 | 93 |
| 189 | 9 | Fundament | Fundamentkoerper Beton C30/37 (IEC IIA) | 1 | 1.045.000 | 418 m3 (x 2.5 t/m3) | 16.00 × 1.60 | 92 |
| 189 | 10 | Fundament | Sauberkeitsschicht Beton C8/10 (IEC IIA) | 1 | 64.000 | 25.6 m3 (x 2.5 t/m3) | — | 92 |
| 189 | 11 | Fundament | Betonstahl BSt 500 S/M (IEC IIA) | 1 | 44.100 | 44.1 to | — | 92 |

## Anhang F — Konfigurationen aus dem Import vom 2026-08-25

`Gewicht` = Σ (`EinzelgewichtKg` × `Stueckzahl`) in Tonnen. `Fund.` = enthält Positionen der
Kategorie `Fundament` (vgl. B5 und Annahme 3 in 5.4 — ohne diese Spalte sind die Gewichte nicht
vergleichbar). `Dok.` = Anzahl verknüpfter Dateien.

| Id | Hersteller | Modell | Variante | kW | Rotor m | NH m | Pos. | Gewicht t | Fund. | Quelle | Dok. |
|---|---|---|---|---|---|---|---|---|---|---|---|
| 191 | Enercon | E-40/6.44 | Stahlrohrturm 56 m | 600 | 44 | 56 | 7 | 105,0 | ja | 96 | 13 |
| 192 | Enercon | E-40/6.44 | Stahlrohrturm 63 m | 600 | 44 | 63 | 7 | 116,0 | ja | 96 | 1 |
| 193 | Enercon | E-40/6.44 | Stahlrohrturm 76,8 m | 600 | 44 | 77,75 | 8 | 144,0 | ja | 96 | 14 |
| 194 | Enercon | E-44 | Gondelkomponenten | 900 | 44 | — | 4 | 36,5 | nein | 97 | 3 |
| 195 | Enercon | E-48 | Stahlrohrturm 50 m | 800 | 48 | 50 | 7 | 95,6 | nein | 101 | 12 |
| 196 | Enercon | E-48 | E-48/S/55/3K/01 | 800 | 48 | 55,6 | 8 | 96,4 | ja | 98 | 2 |
| 197 | Enercon | E-48 | E-48/S/75/3F/01 | 800 | 48 | 75,6 | 8 | 146,1 | ja | 99 | 9 |
| 198 | Enercon | E-53 | E-53/S/72/3K/02 | 800 | 52,9 | 73,25 | 6 | 794,2 | ja | 102 | 7 |
| 199 | Enercon | E-58 | E-58/S/70/5/01-10.58 | 1000 | 58,6 | 70,51 | 9 | 217,4 | ja | 104 | 6 |
| 200 | Enercon | E-66/18.70 | E-66/18.70/S/63/5/02 | 1800 | 66 | 65 | 8 | 242,2 | nein | 108 | 10 |
| 201 | Enercon | E-66/20.70 | E-66/S/63/3F/01-20.70 | 2000 | 70 | 64,75 | 9 | 240,1 | ja | 106 | 6 |
| 202 | Enercon | E-66/18.70 | 65-m-Turm, 4-Sektionen-Variante | 1800 | 66 | 65 | 10 | 246,8 | ja | 110 | 6 |
| 203 | Enercon | E-66/20.70 | E-66/S/69/4F/01-20.70 | 2000 | 70 | 69,75 | 8 | 277,9 | ja | 107 | 1 |
| 204 | Enercon | E-66/18.70 | E-66/18.70/S/84/5/01 | 1800 | 66 | 85 | 8 | 322,7 | nein | 108 | 11 |
| 205 | Enercon | E-66/18.70 | E-66/18.70/0/97/2/04 (Spannbeton) | 1800 | 66 | 98 | 5 | 984,7 | nein | 108 | 26 |
| 206 | Enercon | E-70 E4 | Stahlrohrturm 85 m (4 Sektionen) | 2300 | 71 | 85 | 13 | 366,4 | ja | 113 | 10 |
| 207 | Enercon | E-70 E4 | E-70 E4/BF/112/24/01 | 2300 | 71 | 113,5 | 4 | 1053,3 | nein | 114 | 32 |
| 208 | Enercon | E-70 E4 | E-70 E4/BF/112/24/02 – Fundament | 2300 | 71 | 113,5 | 2 | 987,6 | ja | 112 | 32 |
| 209 | Enercon | E-82 E2 | E-82 E2/BF/83/17/01 | 2000 | 82 | 85 | 10 | 1971,1 | nein | 116 | 9 |
| 210 | Enercon | E-82 E2 | E-82 E2/BF/97/20/05 | 2000 | 82 | 98 | 10 | 2424,0 | nein | 116 | 9 |
| 211 | Enercon | E-82 E2 | E-82 E2/BF/107/23/05 | 2000 | 82 | 108 | 10 | 2688,4 | nein | 116 | 12 |
| 212 | Enercon | E-82 E2 | E-82 E2/BF/137/24/01 | 2000 | 82 | 137 | 10 | 4027,2 | nein | 116 | 9 |
| 213 | Enercon | E-101 / E-115 | BF/147/31/01 bzw. /31/02 – Fundament | 3000 | 101 | 147 | 2 | 1718,8 | ja | 94 | 8 |
| 214 | Enercon | E-147 EP5 | E2/MST/155/FB/C/01 – Fundament | 5000 | 147 | 155 | 2 | 2914,8 | ja | 95 | 1 |
| 215 | GE | GE 1.5-77 / 1.6-77 | MTS, NH 65 m | 1500 | 77 | 65 | 8 | 190,8 | ja | 118 | 33 |
| 216 | GE | GE 1.5-77 / 1.6-77 | MTS, NH 80 m (L-Flansch) | 1500 | 77 | 80 | 8 | 229,1 | ja | 118 | 33 |
| 217 | GE | GE 1.5-77 / 1.6-77 | pMTS, NH 100 m | 1500 | 77 | 100 | 10 | 331,3 | ja | 118 | 33 |
| 218 | GE | GE 1.5sl | MTS WZ II, NH 64,7 m | 1500 | 77 | 64,7 | 7 | 176,0 | nein | 119 | 26 |
| 219 | GE | GE 1.5sl | MTS WZ II, NH 80 m | 1500 | 77 | 80 | 7 | 210,5 | nein | 119 | 26 |
| 220 | GE | GE 1.5sl | MTS WZ II, NH 85 m (Variante A) | 1500 | 77 | 85 | 8 | 223,2 | nein | 119 | 26 |
| 221 | GE | GE 1.5sl | MTS WZ II, NH 85 m (Variante B) | 1500 | 77 | 85 | 8 | 226,6 | nein | 119 | 26 |
| 222 | GE | GE 1.5sl | MTS WZ II, NH 96 m | 1500 | 77 | 96 | 9 | 301,5 | nein | 119 | 26 |
| 223 | GE | GE 1.5sl | WZ II, NH 96 m (4 Sektionen) | 1500 | 77 | 96 | 8 | 266,0 | nein | 119 | 26 |
| 224 | GE | GE 1.5sl | WZ II, NH 100 m | 1500 | 77 | 100 | 8 | 266,0 | nein | 119 | 26 |
| 225 | Kenersys | K100/2500 | Stahlturm NH 85 m | 2500 | 100 | 85 | 8 | 367,0 | nein | 120 | 12 |
| 226 | Kenersys | K100/2500 | Stahlturm NH 100 m | 2500 | 100 | 100 | 9 | 425,0 | nein | 120 | 12 |
| 227 | Kenersys | K110/2500 | Turmkopf | 2500 | 110 | — | 4 | 162,0 | nein | 120 | 8 |
| 228 | NEG Micon | NM 1000/60 | Round wz II, NH 80 m | 1000 | 60 | 80 | 6 | 166,6 | nein | 128 | 2 |
| 229 | NEG Micon | NM 750/48 | Round rk0/IEC II, HH 45 m | 750 | 48 | 45 | 5 | 70,6 | nein | 127 | 1 |
| 230 | NEG Micon | NM 750/48 | Round rk0/IEC II, HH 50 m | 750 | 48 | 50 | 5 | 79,1 | nein | 127 | 1 |
| 231 | NEG Micon | NM 750/48 | Round rk0/IEC II, HH 55 m | 750 | 48 | 55 | 5 | 86,2 | nein | 127 | 1 |
| 232 | NEG Micon | NM82/1500 | wz II, HH 94 m | 1500 | 82 | 94 | 6 | 262,6 | nein | 129 | 1 |
| 233 | NEG Micon | NM82/1500 | wz II, HH 109 m | 1500 | 82 | 109 | 7 | 317,4 | nein | 129 | 1 |
| 234 | REpower | MD77/1500 | Rohrturm NH 61,5 m | 1500 | 77 | 61,5 | 10 | 205,6 | ja | 121 | 13 |
| 235 | REpower | MD70/MD77 | Rohrturm NH 85 m | 1500 | 77 | 85 | 8 | 275,4 | ja | 121 | 13 |
| 236 | REpower | MD77/1500 | Rohrturm NH 100 m | 1500 | 77 | 100 | 12 | 383,8 | ja | 123 | 13 |
| 237 | REpower | MM82/2050 | Rohrturm NH 80 m | 2050 | 82 | 80 | 7 | 268,5 | nein | 125 | 7 |
| 238 | REpower | MM82/2050 | NH 100 m (Teilangaben) | 2050 | 82 | 100 | 7 | 186,5 | nein | 124 | 1 |
| 239 | REpower | MM92/2050 | Rohrturm NH 80 m | 2050 | 92 | 80 | 9 | 290,0 | ja | 126 | 2 |
| 240 | Schuler | SDD 100 | Stahlturm NH 100 m | 2500 | 100 | 100 | 10 | 534,7 | nein | 130 | 10 |
| 241 | Fuhrländer | FL 1000 | Stahlturm NH 70 m | 1000 | 54 | 70 | 8 | 149,0 | ja | 117 | 2 |
| 242 | Vensys | VENSYS 62 | Gondel | 1500 | 62 | 69 | 1 | 13,5 | nein | 137 | 18 |
| 243 | Vestas | V66-1.65/1.75 MW | 3-teiliger Modularturm, NH 60 m | 1750 | 66 | 60 | 5 | 164,2 | nein | 132 | 17 |
| 244 | Vestas | V66-1.65/1.75 MW | 3-teiliger Modularturm, NH 67 m | 1750 | 66 | 67 | 7 | 204,1 | ja | 133 | 17 |
| 245 | Vestas | V66-1.65/1.75 MW | 4-teiliger Modularturm, NH 78 m | 1750 | 66 | 78 | 5 | 219,2 | nein | 132 | 17 |
| 246 | Vestas | V80-2.0 MW | 5-teiliger Turm, NH 100 m | 2000 | 80 | 100 | 8 | 324,4 | ja | 135 | 10 |
| 247 | Vestas | V44-600 kW | Stahlturm 51 m | 600 | 44 | 51 | 7 | 98,9 | ja | 131 | 1 |
| 248 | Vestas | V90-1.8/2.0 MW | IEC IIA/IIIA, NH 80 m | 2000 | 90 | 80 | 4 | 253,0 | nein | 136 | 27 |
| 249 | Vestas | V90-1.8/2.0 MW | IEC IIA, NH 95 m | 2000 | 90 | 95 | 4 | 303,0 | nein | 136 | 27 |
| 250 | Vestas | V90-1.8/2.0 MW | IEC IIIA, NH 105 m | 2000 | 90 | 105 | 4 | 339,0 | nein | 136 | 27 |
| 251 | Vestas | V90-1.8/2.0 MW | DIBt II, NH 95 m | 2000 | 90 | 95 | 4 | 306,0 | nein | 136 | 27 |
| 252 | Vestas | V90-1.8/2.0 MW | DIBt II, NH 105 m | 2000 | 90 | 105 | 4 | 330,0 | nein | 136 | 27 |

## Anhang G — Abgelegt, aber nicht ausgewertet

Diese 117 Dateien liegen unter `Dokumente/`, sind aber mit keiner Konfiguration verknüpft: entweder
liegen sie nur als Scan ohne Textebene vor, oder sie gehören zu einem Anlagentyp, für den sich im
Archiv keine Gewichtsangaben finden. Sie sind Kandidaten für eine spätere Auswertung per OCR oder
Herstelleranfrage.

| Ordner bzw. Datei | Dateien | Grund |
|---|---|---|
| `Enercon/D0189163_4.1-Demontage und Entsorgung.pdf` | 1 | herstellerweite Demontagebeschreibung ohne Gewichte, keinem Typ zuzuordnen |
| `Enercon/E08_20250312_Übersicht_Ersatzteile Rückbauanlagen.pdf` | 1 | Ersatzteilübersicht ohne Typbezug |
| `Enercon/Enercon - Technische Beschreibung Demontage und Entsorgung.pdf` | 1 | herstellerweite Demontagebeschreibung ohne Gewichte |
| `Enercon/Enercon E-101 - 97 bis 133m` | 1 | nur Zuwegungsspezifikation (Transportgewichte, siehe 5.3) |
| `Enercon/Enercon E-126 EP4 132m` | 1 | nur Zuwegungsspezifikation (Transportgewichte, siehe 5.3) |
| `Enercon/Enercon E-141 EP4 - 159 m` | 1 | nur Zuwegungsspezifikation (Transportgewichte, siehe 5.3) |
| `Enercon/Enercon E-18 E20- 32,5-36m` | 7 | nur Scans und Fotos |
| `Enercon/Enercon E-32 und E33` | 6 | nur Scans und Fotos |
| `Enercon/Enercon E-40 - 48m Beton` | 4 | nur Scans (Typenprüfung, Turmplan) |
| `Enercon/Enercon E-53 - 50m` | 2 | nur Scans |
| `Enercon/Enercon E-53 - 60m` | 3 | nur Scans |
| `Enercon/Enercon E-58 - 89m` | 2 | `E-58 Gewichte und Abmessungen.pdf` liegt nur als Scan vor |
| `Enercon/Enercon E-66 - 65, 67m` | 3 | Unterordner E-66_15.66 - 67m (Gütterlitz): nur Bewehrungs- und Schalpläne |
| `Enercon/Enercon E-66 Verladehandbuch.pdf` | 1 | Ladungssicherung, keine Bauteilgewichte |
| `Enercon/Enercon E-70 - 98m` | 1 | nur Typenprüfung als Scan |
| `Enercon/Enercon E-82 - 137m` | 3 | Ordner enthält E-92-Unterlagen (Fundamentdatenblatt E92, Zuwegung E92), beide ohne Textebene |
| `Enercon/Enercon E-92` | 15 | enthält ausschließlich E-82-Gewichtsblätter; für die E-92 selbst (Rotor 92 m) fehlt jeder eigene Beleg |
| `GE/GE-Energy 2.5-120 (85 – 139 m )2.75-120 (110 und 139 m)` | 2 | keine Gewichtsangaben im Ordner |
| `GE/GE-Energy 2.x-100 (75 - 98,3 m) , 2.x-103 (75 - 98,3 m)` | 14 | Systembeschreibungen und Zuwegung, keine Bauteilgewichte |
| `GE/GE-Energy 3.2-100 (75 und 85 m) 3.2-103 (70 - 98,3 m)` | 1 | nur Zuwegungsspezifikation |
| `Nordex/Nordex N62 -69m` | 15 | bereits vor diesem Import abgelegt, ohne Konfiguration (Altbestand) |
| `Nordtank 1500 - 67,5m` | 9 | nur Scans und Fotos |
| `REpower/MM92 - 100m` | 1 | `90.207 - Anhang - Dokumentation Hollich.pdf` (199 MB) enthält keine geschlossene Gewichtstabelle |
| `REpower/REpower MD70_Fuhrländer MD70_Südwind S70 - 85m` | 2 | Foto und E-Mail; die Gewichte der E-Mail stehen in der Notiz zu Konfiguration 235 |
| `Senvion/Senvion 82,92,100 3,2M 114, 3,4M 104, 3,4M 114, 3,0M 122` | 1 | nur Leerrohrplan des Fundaments |
| `Senvion/V-1.1-GP.00.10-A-M-EN Transport Specification.pdf` | 1 | Transportspezifikation ohne Bauteilgewichte |
| `Vestas/Vestas Daten V136` | 6 | Fundamentzeichnungen und Bewehrungslisten ohne Summen; Anlagengewichte fehlen |
| `Vestas/Vestas Turmvarianten.pdf` | 1 | Sammeltabelle V52–V100; Spaltenzuordnung im Textextrakt nicht zuverlässig rekonstruierbar |
| `Vestas/Vestas V112, 140m, V80, V100, V112, V117, V126` | 1 | nur Fundamentzeichnung als Scan |
| `Vestas/Vestas V112, V117, V126, V136` | 3 | Fundament- und Zuwegungsunterlagen, keine Bauteilgewichte |
| `Vestas/Vestas V162` | 1 | nur Achslastberechnung |
| `Vestas/Vestas V47 - 50m` | 2 | Datei inhaltlich identisch mit der V44-Datei, Zuordnung ungeklärt (Annahme 8) |
| `Wind World/750-52` | 3 | nur Scans |
| `__Senvion` | 1 | Fehlerprotokoll eines Downloads, kein Fachdokument |
| **Summe** | **117** | |
