# Support-Playbook (intern)
*Version 3.1 · für Support-Mitarbeitende Level 1*

## Reaktionszeiten (SLA)
| Kategorie | Erstreaktion | Lösung |
|---|---|---|
| Sicherheitsmeldung (Bremse, Licht, Rahmen) | **1 Stunde** | 24 Stunden |
| Abrechnungsfehler | 24 Stunden | 5 Werktage |
| App-Fehler | 48 Stunden | nach Priorisierung |
| Allgemeine Anfrage | 72 Stunden | — |

## Eskalationsregel Sicherheit
Jede Meldung, die auf ein **sicherheitsrelevantes Defizit** hinweist — Bremsversagen,
Lenkerspiel, defekte Beleuchtung bei Dunkelheit, Rahmenbruch — wird **sofort** an das
Ops-Team eskaliert und das betroffene Rad per Fernzugriff gesperrt.
Kein Level-1-Mitarbeitender schließt einen solchen Fall eigenständig.

## Erstattungen
- Bis **25,00 €**: Level 1 darf eigenständig erstatten.
- **25,01 € bis 100,00 €**: Freigabe durch Teamlead erforderlich.
- Über **100,00 €**: Freigabe durch Ops-Lead, schriftliche Begründung.

## Häufige Fälle
**„Fahrt wurde nicht beendet"** — Prüfen, ob das Schloss ein Endsignal gesendet hat.
Falls nicht: Fahrt manuell auf den Zeitpunkt des letzten GPS-Stillstands zurücksetzen,
Differenz erstatten.

**„Rad nicht entsperrbar"** — Nutzer bitten, den QR-Code bei Tageslicht erneut zu scannen.
Bei wiederholtem Fehler Rad als `unlock_fault` markieren.
