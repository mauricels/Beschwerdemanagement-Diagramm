# Zusammenfassung, potenzielle Probleme und Feedback

Stand: Camunda-8-Version mit fünf Dateien, die über Aufruf-Aktivitäten (Call Activities) verknüpft sind.

## Zusammenfassung

Das Modell beschreibt, wie ein E-Bike-Händler eine Kundenreklamation bearbeitet. Es besteht aus einem Hauptprozess und vier Teilprozessen.

- **`0_Hauptprozess`:**
  - Die Beschwerde des Kunden geht als Nachricht ein und wird aufgenommen.
  - Danach wird der Fehler geklärt und die Kostenübernahme geprüft.
  - Sind die Kosten nicht gedeckt, bekommt der Kunde einen Kostenvoranschlag.
  - Sind sie gedeckt, entscheidet die Schadenshöhe: Ein geringer Schaden führt zur Ersatzteillösung (Versand mit Anleitung oder Einbau in der Werkstatt). Ein hoher Schaden geht direkt in die Werkstatt.
  - Lanes: Kundenservice, Werkstatt, Schadensmanagement.
  - Externe Pools: Kunde, Hersteller, Versicherung.
- **`1_Fehler_klaeren`:** Fehlerspeicher auslesen → Bekannter Fehler? → bekannte Lösung zuordnen oder Detaildiagnose durchführen.
- **`2_Kostenuebernahme_klaeren`:**
  - Bei Garantie mit nötiger Herstellerfreigabe geht ein Garantieantrag an den Hersteller.
  - Ohne Garantie oder bei Ablehnung wird `2a` aufgerufen.
- **`2a_Versicherungsfall_abwickeln`:**
  - Ursache prüfen, Schaden melden, auf die Antwort warten.
  - Alle 14 Tage wird nachgefasst ("Bei Versicherung nochmal melden"). Der Timer bricht das Warten nicht ab.
  - Ergebnis: Deckungszusage oder keine Deckung.
- **`3_Ersatzteilloesung_festlegen`:** Ersatzteil bestellen → Spezialwerkzeug nötig? → Selbsteinbau gewünscht? → Einbau durch Kunde oder Werkstatt.

## Potenzielle Probleme

### Formal (BPMN Quick Guide)

- Die Startereignisse der Teilprozesse haben keine Namen.
- Vor mehreren verzweigenden Gateways steht keine Aktivität, die die Entscheidung trifft. Das betrifft "Höhe des Schadens?" und "Innerhalb Garantie?".
- Gateways folgen direkt aufeinander:
  - Garantie → Herstellerfreigabe
  - Spezialwerkzeug → Selbsteinbau
- Namen enthalten Artikel: "Antwort der Versicherung abwarten", "Höhe des Schadens?".
- Die Pools in den Teil-Dateien sind nach einer Abteilung benannt statt nur nach dem Teilnehmer.

### Modellierung

- Die Aufgaben haben keinen Typ (User, Manual oder Service Task).
- Gateway-Ausgänge heißen Ja/Nein statt nach dem Ergebnis, z. B. "anerkannt/abgelehnt".
- In `2_Kostenuebernahme_klaeren` kreuzen die Nachrichtenflüsse zum Hersteller einen Sequenzfluss.
- Die Dateien sind als ausführbar markiert. Deshalb zeigt der Modeler Hinweise zu fehlenden technischen Angaben, z. B. Nachrichtennamen.

### Fachlich

- Nach dem Kostenvoranschlag endet der Prozess. Was passiert, wenn der Kunde annimmt und selbst zahlt, fehlt.
- Beim Hersteller wird ohne Frist gewartet. Auch bei der Versicherung gibt es keine Eskalation, die Erinnerungen laufen endlos weiter.
- Für den Selbsteinbau gibt es keinen Rückweg, falls der Einbau scheitert oder das Teil falsch ist.
- Gesetzliche Gewährleistung (Händler) und freiwillige Garantie (Hersteller) werden nicht unterschieden.
- "Fehler klären" liegt in der Lane Kundenservice, obwohl die Diagnose in der Werkstatt stattfindet.
- Bei hohem Schaden sind die Ersatzteilbeschaffung, ein Leihrad und Statusmeldungen an den Kunden nicht modelliert.

## Feedback vom Professor

| Kritikpunkt | Was gemeint ist | Möglicher Umbau |
|---|---|---|
| Mehrere Endpunkte in den Teilprozessen, weniger im Hauptprozess | Der Hauptprozess hat drei Endereignisse (Kostenvoranschlag versendet, Ersatzteil versendet, Reparatur abgeschlossen) | Die Varianten in die Teilprozesse verlagern. Im Hauptprozess die Wege über ein zusammenführendes Gateway zu einem Endereignis bündeln, z. B. "Beschwerde abgeschlossen" |
| "Beschwerde eingegangen" detaillieren | Das Startereignis sagt nicht, wie die Beschwerde ankommt | Kanäle als eigene Startereignisse modellieren (Telefon, E-Mail, Filiale, Webformular) oder "Beschwerde aufnehmen" als Teilprozess ausführen (Kundendaten erfassen, Kaufbeleg prüfen, Fehlerbild dokumentieren) |
| Ersatzteile bestellen detaillierter | "Ersatzteil bestellen" ist nur eine einzelne Aufgabe | Ausführen in: Verfügbarkeit prüfen → Lager oder Lieferant? → Bestellung auslösen → Lieferung abwarten (mit Frist) → Wareneingang prüfen |
| Substantive bei Prozessnamen | Prozesse sollen als Substantiv benannt werden | Fehlerklärung, Kostenübernahmeprüfung, Versicherungsfallabwicklung, Ersatzteilbeschaffung, Beschwerdemanagement. Aufgaben bleiben laut Quick Guide Objekt + Verb (z. B. "Ersatzteil bestellen") |

## Die Box "Kostenübernahme klären" kleiner machen

Die Box ist breiter als die Standardgröße von 100 × 80, weil unten fünf Nachrichtenflüsse zu Hersteller und Versicherung andocken.

- **Im Camunda Modeler:** Box anklicken und am Eckpunkt unten rechts auf etwa 100 × 80 ziehen. Die Nachrichtenflüsse bleiben verbunden, rücken aber zusammen. Danach die Andockpunkte und Beschriftungen von Hand nachziehen, damit sie sich nicht überlappen.
- **Dauerhaft sauberer:** Im Hauptprozess weniger Nachrichtenflüsse zeigen, z. B. nur einen Fluss pro Partner und Richtung, oder einige Flüsse an der Seite der Box andocken. Die vollständige Kommunikation ist ohnehin in `2_Kostenuebernahme_klaeren` und `2a_Versicherungsfall_abwickeln` zu sehen.
