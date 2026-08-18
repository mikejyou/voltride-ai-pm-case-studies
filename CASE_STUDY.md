# Belastbare Support-Auskünfte aus sechs internen Dokumenten — mit Quelle und mit dem Mut zu schweigen

Für den E-Bike-Sharing-Dienst VoltRide habe ich einen Assistenten für den Level-1-Support gebaut, der Fragen aus sechs internen Dokumenten beantwortet, jede Aussage mit einer Quelle belegt und bei nicht dokumentierten Fragen ablehnt — und ich habe gemessen, was jede der beiden Fehlerrichtungen kostet.

`Dauer: ~10 Stunden` · `Rolle: Konzept, Umsetzung, Evaluation` · `Stack: Python, sentence-transformers (lokal), Groq / gpt-oss-20b, pandas`

> **Hinweis zur Datenbasis**
> Diese Case Study basiert auf einem vollständig fiktiven Unternehmen und synthetischen Daten.
> Das war eine bewusste Entscheidung: Portfolio-Arbeit muss öffentlich sein, und Daten meines
> Arbeitgebers sind das nicht. Der Vorteil des synthetischen Datensatzes ist, dass er
> Ground-Truth-Labels hat — dadurch ist die Qualität des Systems messbar statt behauptet.

---

## 1. Das Problem

Der Level-1-Support beantwortet den ganzen Tag Fragen zu Preisen, Fristen, Erstattungsgrenzen und Geschäftsgebieten. Die Antworten stehen verstreut in sechs internen Dokumenten, teilweise widersprüchlich.

Die beiden Fehlerrichtungen sind unterschiedlich teuer. Eine erfundene Auskunft — „Für 60 € darfst du selbst erstatten" — geht an einen zahlenden Kunden und wird zur Zusage. Eine unnötige Ablehnung kostet eine Eskalation. Bei 200 Anfragen am Tag ist die erste Sorte die teure.

## 2. Warum das mit klassischer Software nicht geht

Eine Volltextsuche findet Dokumente, keine Antworten. „Was kostet eine 20-minütige Fahrt im Tarif Basic?" steht nirgendwo — die Zahl entsteht erst aus Entsperrgebühr plus Minutenpreis. Ein Sprachmodell kann das. Was es nicht von allein kann: den Unterschied zwischen *steht im Dokument* und *klingt plausibel*. Genau darin liegt die Produktarbeit.

## 3. Der Ansatz

Dokumente an Überschriften in Abschnitte schneiden → Abschnitte lokal als Vektoren ablegen → zu jeder Frage die drei ähnlichsten Abschnitte holen → das Modell antwortet **nur** aus diesen Abschnitten, in einem festen Format mit Feldern für Status, Antwort, Quellen und Widerspruch.

**Verworfen, mit Begründung:**

- **Ein Modell als Bewerter (LLM-as-a-Judge).** Bequemer, aber der Bewerter ist selbst eine Fehlerquelle, die ich wiederum gegen meine Labels hätte validieren müssen. Stattdessen eine sichtbare Prüfmuster-Tabelle: pro Frage steht explizit da, woran „richtig" erkannt wird.
- **Ein größeres Modell.** Naheliegend, aber falsch — die Messung zeigt, dass das Modell nie der Engpass war.
- **Eine gemeinsame Qualitätskennzahl.** Ein System mit 95 % Trefferquote und 0 % Ablehnungsquote ist gefährlich, kein Erfolg. Die beiden Zahlen werden nie zusammengefasst.

## 4. Wie ich Qualität gemessen habe

**Der Testdatensatz:** 25 Fragen mit bekannter Wahrheit — 14 Faktfragen, 3 Mehrschrittfragen, **5 echte Lücken, die abgelehnt werden müssen**, 2 dokumentierte Widersprüche, 1 Fangfrage.

**Drei Kennzahlen, getrennt:** Trefferquote auf den 17 beantwortbaren Fragen. Ablehnungsquote auf den 5 Lücken. Und — das war der Wendepunkt — **Beleg-Recall**: stand die belegende Textstelle überhaupt im gelieferten Kontext?

Drei Läufe, ein Eingriff pro Lauf, nur die Ablehnungsregel im Prompt verändert:

| Lauf | Trefferquote (n=17) | Ablehnungsquote (n=5) | falsche Ablehnung | Beleg-Recall |
|---|---|---|---|---|
| `basis` | 76 % | 80 % | 18 % | 74 % |
| `streng` | 71 % | 80 % | 18 % | 74 % |
| `sehr_streng` | 71 % | **100 %** | 24 % | 76 % |

**Der zentrale Befund:** Alle drei Läufe scheitern an denselben vier Fragen. Das Retrieval hängt nicht vom Prompt ab, ist also in allen Läufen identisch — ich habe dreimal an einer Schraube gedreht, die nicht das Problem war.

**Von den 13 Fragen, bei denen der Beleg nachweislich im Kontext stand, wurden 12 richtig beantwortet.** Die eine Ausnahme ist ein Fehlalarm meines Beleg-Tests, kein Modellfehler. Umgekehrt: jeder einzelne echte Fehler entstand eine Stufe vor dem Modell, beim Suchen.

Was die Verschärfung gebracht hat, war exakt eine Sache: die einzige Halluzination des Projekts wurde zur Ablehnung. Bezahlt habe ich mit einer brauchbaren Teilauskunft („Aarburg, Preis nicht angegeben"), die zum blanken „steht nicht drin" wurde. **Die Ablehnungsregel ist eine Versicherung gegen Retrieval-Fehler, kein Qualitätshebel.**

### Fehlerklassen

| Klasse | Häufigkeit | Ursache | Gegenmaßnahme | Wirkung |
|---|---|---|---|---|
| `beleg_nicht_im_kontext` | 4 von 17 | falscher Abschnitt geliefert; die Städtetabelle ist **ein** Vektor, in dem jede einzelne Stadt untergeht | Tabellen zeilenweise schneiden | **Regression, siehe 5** |
| `pruefer_zu_streng` | 4 von 17 | Muster verlangte die Formulierung der Musterlösung statt der Aussage | „falsch" von „unvollständig" getrennt | 53 % → 71 % |
| `kennzahl_verzerrt` | 1 Kennzahl | Ablehnungen als falsche Quellenangabe gezählt | nur beantwortete Zeilen zählen | 68 % → 100 % |
| `halluzination` | 1 von 25 | Beleg fehlte, das Modell füllte die Lücke | strengere Ablehnungsregel | wirkt, aber teuer |

**Vorher/Nachher:** Meine Automatik meldete zunächst 53 % Trefferquote. Nach der Handcodierung waren es 76 %. **24 Prozentpunkte Messfehler — verursacht vom Messinstrument, nicht vom System.** Beispiel: „Bis 25,00 € eigenständig, für 60,00 € Freigabe durch Teamlead" galt als falsch, weil das Wort „nein" fehlte.

## 5. Was nicht funktioniert hat

**Der Eingriff, der netto nichts brachte.** Die Städtetabelle zeilenweise zu schneiden reparierte drei Fragen und machte drei andere kaputt. Beleg-Recall vorher wie nachher 68 %. Ursache: Ich wiederhole die Tabellenkopfzeile in jeder Zeile, damit „620" nicht kontextlos dasteht — damit stand das Wort „Geschäftsgebiet" plötzlich in sechs kurzen Abschnitten und schlug bei der Frage „Was kostet das Abstellen außerhalb des Geschäftsgebiets?" den einen richtigen Abschnitt. Sichtbar wurde das nur, weil ich die *Zusammensetzung* verglichen habe und nicht die Prozentzahl.

**Der Unterschied zwischen 76 % und 71 % war gar kein Modellunterschied.** Das Modell gab in allen drei Läufen dieselbe Antwort auf Q07. Nur für den ersten Lauf existierte mein Handurteil. Ein Label, das für einen von drei Läufen gesetzt ist, erzeugt eine Differenz, die aussieht wie ein Ergebnis.

**Der teuerste Posten war das Werkzeug, nicht das Modell.** Das Notebook musste sechsmal neu gebaut werden — fehlende Zeilenumbrüche im Dateiformat, ein Ordner-Upload, der nie durchlief, Komma statt Semikolon als Trennzeichen, zweimal Zellen, die auf Variablen früherer Zellen zugriffen und nach einem Neustart abstürzten. Nur zwei dieser Runden brachten inhaltlich etwas. Ich habe mit einem KI-Assistenten als Sparringspartner gearbeitet; ein guter Teil des Aufwands ging in Korrekturschleifen, die mit dem eigentlichen Problem nichts zu tun hatten.

**Rate Limits.** Das kostenlose Kontingent brach vier Aufrufe mitten im Lauf ab, und die Fehlermeldung war im Code auf 110 Zeichen gekürzt — genau die entscheidende Stelle fehlte. Ich musste eine Diagnosezelle bauen, die die Zähler direkt aus den Antwort-Headern liest. Dabei zeigte sich: die Grenze zählt das **reservierte** Ausgabebudget mit. Ein Testaufruf mit 16 Token geht durch, derselbe mit 1.600 Token wird abgelehnt — ein zu kleiner Test meldet „alles frei", obwohl nichts frei ist.

Über mehrere Sitzungen hinweg hat mich zweierlei gerettet: ein Cache, der **nur erfolgreiche** Aufrufe speichert — Fehlschläge im Cache hätten still falsche Messwerte erzeugt — und eine Zeile, die den Stand vor jedem Verbindungsabbruch sichert:

```python
import shutil
shutil.make_archive('cs2_arbeit', 'zip', '/content/cs2_arbeit')
```

## 6. Kosten und Betrieb

Drei komplette Läufe über 25 Fragen: **0,0094 USD** (60.122 Eingabe-, 6.773 Ausgabetoken). Hochgerechnet auf 200 Supportfragen pro Tag: **rund 0,75 USD im Monat**. Die Embeddings laufen lokal und kosten nichts.

Ein größeres Modell wäre die falsche Investition. Die Messung zeigt, dass das Modell alles richtig beantwortet hat, was es sehen konnte. Die Betriebskosten liegen woanders: in der Pflege der Dokumente und im Nacharbeiten der Fälle, in denen die Suche den falschen Abschnitt liefert.

## 7. Was ich anders machen würde

**Zuerst das Retrieval messen, dann alles andere.** Der Beleg-Recall kostet keinen einzigen Modellaufruf und hätte mir drei Prompt-Läufe erspart. Ich habe ihn erst nach dem dritten Lauf gebaut.

**Das Messinstrument genauso ernst nehmen wie das System.** Zwei meiner Kennzahlen waren falsch, bevor eine davon etwas über das System aussagte. Eine Kennzahl ohne Blick auf die Verteilung dahinter ist eine Behauptung.

**Beim Chunking bei der Struktur bleiben, die das Dokument vorgibt.** Mein Eingriff in die Tabellen war gut gemeint und hat ein neues Problem erzeugt, das ohne Vorher/Nachher-Vergleich unsichtbar geblieben wäre.

---

*Code, Notebook mit ausgeführten Ausgaben, Eval-Datensatz und alle Rohergebnisse: [GitHub-Link einfügen]*
