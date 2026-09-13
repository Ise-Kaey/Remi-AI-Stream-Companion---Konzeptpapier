# Remi-AI Stream Companion - Konzeptpapier

[Deutsch (Hier)] | [English](README-EN.md) | [Dokumentation](docs/index.md)

---

**Remi** ist ein sozialer KI-Agent für Live-Streaming. Er hört auf Zuruf, liest
den Chat mit, erinnert sich an die Leute davor - und entscheidet selbst, ob,
wann und wie er reagiert.

> ℹ️ **Dieses Repository ist Dokumentation, kein Programm.** Die Implementierung
> ist Closed Alpha und nicht öffentlich. Hier steht, wie das System aufgebaut
> ist, wie es mit Zuschauerdaten umgeht und warum die Steuerungsebene so
> entworfen wurde, wie sie entworfen wurde.

## Worum es hier geht

Zwei Themen, die bei Chat-Agenten selten zusammen behandelt werden:

**Sicherheit.** In einem öffentlichen Chat schreiben viele einander fremde
Menschen gleichzeitig in ein gemeinsames Kontextfenster. Die üblichen
Absicherungen von KI-Assistenten setzen einen wohlgesonnenen Nutzer voraus. Hier
ist feindlicher Input der Normalfall, nicht die Ausnahme.

**Datenschutz.** Ein Agent, der sich an Leute erinnert, speichert Daten über
Leute. Welche das sind, wie lange sie bleiben und was davon das Gerät verlässt,
sollte man nachlesen können, bevor man so etwas in einem Stream einsetzt.

| Seite | Inhalt |
| --- | --- |
| [Technik](docs/de/technik.md) | Das Steuerungsprotokoll, und warum das Modell nur vorschlägt |
| [Sicherheit](docs/de/sicherheit.md) | Bedrohungsmodell, Invarianten, Durchsetzung, Grenzen |
| [Daten](docs/de/daten.md) | Was gespeichert wird, was übertragen wird, Widerspruch |

## Was ist das?

Remi sitzt zwischen dem Chat-Frontend und den KI-Diensten und hält alles, was
Zustand hat: Gesprächsverlauf, Nutzerprofile, Moderationshistorie, Stream-Status
und Persona. Das Frontend liefert nur Ereignisse und führt Aktionen aus.

```text
   Client                      Remi                       KI-Dienste
   ──────                      ────                       ──────────
   Chat rein         ──▶       Persona · Verlauf   ──▶    Antworttext
   Stream-Events     ──▶       Gedächtnis · Regeln ──▶    Sprachausgabe
   Timeout / Ban     ◀──       Sprachpfad
                               Mikrofon ──▶ ──▶ Lautsprecher
```

Der Server selbst hat **keine Plattform-Anbindung** - keine Kanäle, keine
Zugangsdaten, keine Schnittstelle zu Twitch, YouTube oder Discord.
Moderationsaktionen wandern über eine Warteschlange zum Client, der sie ausführt
und quittiert. Der Server könnte niemanden sperren, selbst wenn er wollte.

Das ist der Grund, warum die Anbindung austauschbar ist - und zugleich der
Grund, warum bestimmte Daten gar nicht erst entstehen.

### Was er tut

Knapp umrissen, damit die späteren Aussagen einzuordnen sind - was der Agent
kann, bestimmt, worüber Sicherheit und Datenschutz überhaupt zu reden haben.

| | |
| --- | --- |
| **Sprache** | Hört auf Tastendruck oder auf ein Aktivierungswort, erkennt das Gesagte lokal und antwortet gesprochen. Nach der Antwort bleibt das Mikrofon kurz offen, damit Rückfragen ohne erneuten Tastendruck gehen |
| **Chat** | Liest mit und antwortet nicht auf alles - nach Wahrscheinlichkeit, aber immer bei direkter Ansprache. Begrüßt Zuschauer beim ersten Auftauchen, fasst auf Anforderung zusammen |
| **Gedächtnis** | Ein gemeinsamer Gesprächsverlauf für Sprache und Chat, dazu Profile zu einzelnen Zuschauern. Beides fließt in jede Antwort ein |
| **Moderation** | Kann Verwarnung, Timeout, Bann und Begnadigung **vorschlagen**. Ausgeführt wird ausschließlich vom Client - der Server hat keine Plattformrechte |
| **Persona** | Wer der Agent ist, steht in austauschbaren Textdateien und lässt sich im laufenden Betrieb wechseln |

Die Moderation ist dabei der Teil mit Zähnen, und deshalb der Grund, warum die
[Sicherheitsseite](docs/de/sicherheit.md) so ausführlich ausfällt.

### Streaming ist der Referenzfall, nicht die Grenze

Live-Streaming ist der Einsatz, für den Remi gebaut und täglich erprobt ist -
und der anspruchsvollste: viele fremde Sprecher gleichzeitig, Echtzeit, und
jeder Fehler ist sofort hörbar. Die Architektur selbst weiß davon nichts.
Austauschbar sind drei Dinge:

| Teil | Legt fest |
| ---- | --------- |
| **Persona** | Wer der Agent ist und wie er spricht |
| **Datenbanken** | Woran er sich erinnert |
| **Client** | Woher Ereignisse kommen und wer Aktionen ausführt |

Wer alle drei ersetzt, bekommt einen anderen Agenten auf derselben Maschinerie:

- **Smart Home** - ein Client an der Hausautomation, Persona und Datenbank auf
  Geräte und Gewohnheiten zugeschnitten. Für einen Sprachassistenten, der ohne
  Cloud auskommen soll, ist die lokale Spracherkennung der interessante Teil
- **Meetings** - derselbe Sprachpfad als Protokollant, die Datenbanken als
  wachsende Wissensbasis statt als Zuschauergedächtnis

Es gibt heute keinen zweiten Einsatzfall - das ist eine Entwurfseigenschaft,
kein Feature. Aber es erklärt, warum die Befehlsebene generisch gehalten ist und
die Client-Schnittstelle dokumentiert wird.

## Wie es lokal bleibt

Der Agent besteht aus vier Verarbeitungsstufen, die sich darin unterscheiden, ob
Daten das Gerät verlassen:

| Stufe | Wo | Verlässt Daten die Maschine? |
| --- | --- | --- |
| Spracherkennung | frei konfigurierbar, Vorgabe lokal | kommt darauf an - lokaler Erkenner: nichts. Cloud-Erkenner: der Ton selbst. Das Weckwort wird immer lokal erkannt |
| Gedächtnispflege | frei konfigurierbar, Vorgabe lokal, standardmäßig aus | kommt darauf an - lokal: nichts. Auf einem Endpunkt: je nach Aufgabe die Äußerung, ein ganzer Sitzungsabschnitt, die Figurenbeschreibung oder eine erfundene Anekdote |
| Haupt-Sprachmodell | frei konfigurierbar | nur bei Cloud-Anbieter |
| Sprachausgabe | frei konfigurierbar | kommt darauf an - Cloud-Adapter: Text geht raus. Lokaler Adapter: nichts |

**Beide Enden der Skala lassen sich einstellen.** Zeigt die Konfiguration auf
ein selbst betriebenes Sprachmodell und eine lokale Stimme, verlässt bei
lokaler Erkennung - der Vorgabe - **nichts mehr** die Maschine. Zeigen
umgekehrt alle vier Stufen auf Dienste, verlässt praktisch alles das Gerät,
und zwar an **bis zu vier verschiedene Anbieter**: der Ton an den Erkenner,
der Verlauf an das Sprachmodell, der gesprochene Satz an die Stimme, und je
nach Aufgabe Abschnitte oder Erfundenes an die Gedächtnispflege.

Eines bleibt bauartbedingt lokal, auch dann: **das Weckwort.** Die Schleife,
die dauernd mithört, bekommt immer einen Erkenner auf dem Gerät - sonst ginge
jede Sprachregung im Raum hinaus.

Dazwischen liegt der Normalfall, und die Stufen sind einzeln einstellbar.
Details und die wichtige Einschränkung dazu - *was genau in einen Prompt
geht* - stehen auf der [Datenseite](docs/de/daten.md).

## Was dieses Repository nicht ist

- **Kein Quellcode.** Die Implementierung ist Closed Alpha.
- **Keine Installationsanleitung.** Es gibt hier nichts zu installieren.
- **Keine Schnittstellendokumentation.** Endpunkte, Parameter und Beispiele
  gehören zur Implementierung und stehen bewusst nicht hier.
- **Kein Angebot und kein Support.** Dies ist eine Beschreibung, keine Zusage.

Wer die Konzepte übernehmen möchte: die Ideen auf den drei Seiten sind nicht
geheim, und sie sind mit Absicht so aufgeschrieben, dass sie sich ohne diesen
Code nachbauen lassen.

## Lizenz

Die Texte in diesem Repository stehen unter
[CC BY 4.0](https://creativecommons.org/licenses/by/4.0/deed.de). Die
beschriebene Software ist davon nicht erfasst.
