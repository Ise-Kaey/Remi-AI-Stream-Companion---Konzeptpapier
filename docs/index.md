# Dokumentation

[← README](../README.md) | [English README](../README-EN.md)

---

Diese Dokumentation beschreibt den Aufbau des **Remi-AI Stream Companion** -
eines sozialen KI-Agenten für Live-Streaming. Sie ist als Konzeptpapier gedacht:
Sie erklärt Entwurfsentscheidungen und ihren Hintergrund, nicht die Bedienung.
Die Implementierung ist Closed Alpha und nicht Teil dieses Repositories.

Die drei Seiten bauen aufeinander auf und lassen sich in dieser Reihenfolge am
Stück lesen.

## Deutsch

| Seite | Inhalt |
| ----- | ------ |
| [Technik](de/technik.md) | Das Inhaltsfeld als Steuerkanal, Richtungstrennung, und warum das Modell nur vorschlägt statt auszuführen. Dazu die Begründung, warum hier keine Tool-Calls zum Einsatz kommen |
| [Sicherheit](de/sicherheit.md) | Das Bedrohungsmodell vieler fremder Sprecher in einem Kontextfenster, die drei tragenden Invarianten, strukturelle statt filternde Abwehr - und die Grenzen des Verfahrens |
| [Daten](de/daten.md) | Welche Zuschauerdaten gespeichert werden, wie lange, was davon an Dritte übertragen wird, und wie ein Widerspruch wirkt |

## English

| Page | Contents |
| ---- | -------- |
| [Technology](en/technology.md) | The content field as a control channel, directional separation, and why the model only proposes rather than acts. Plus the reasoning against tool calls |
| [Security](en/security.md) | The threat model of many mutually unfamiliar speakers in one context window, the three load-bearing invariants, structural rather than filtering defence - and where it stops |
| [Data](en/data.md) | Which viewer data is stored, for how long, what is transmitted to third parties, and how opting out works |
