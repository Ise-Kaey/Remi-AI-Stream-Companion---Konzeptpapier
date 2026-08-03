# Technik: ein Kanal, mehrere Bedeutungen

[Deutsch (Hier)] | [English](../en/technology.md) | [Übersicht](../index.md)

---

Diese Seite erklärt, wie der Agent Handlungen auslöst - und warum das anders
gelöst ist, als man es heute üblicherweise bauen würde. Der kurze Satz vorweg,
alles Weitere ist seine Begründung:

> **Das Modell schlägt vor. Der Server entscheidet und führt aus.**

## Ein Kanal, mehrere Bedeutungen

Von außen sieht der Server aus wie ein gewöhnlicher Chat-Endpoint im verbreiteten
Format: eine Liste von Nachrichten rein, eine Antwort raus. Tatsächlich läuft
über das Inhaltsfeld dieser Nachrichten mehr als nur Chat. Es ist ein
gemultiplexter Steuerkanal.

Derselbe Text kann sein: ein normaler Zuschauerbeitrag, ein Systemereignis wie
ein Raid, oder ein deterministischer Befehl, der gar nicht erst beim
Sprachmodell landet. Auf dem Rückweg dasselbe - neben dem sichtbaren Antworttext
kann die Antwort Moderationsbefehle tragen oder ein Signal, das schlicht „nicht
antworten" bedeutet.

Ein Feld, mehrere Bedeutungsebenen. Das ist ein Protokoll, kein Textfeld.

## Warum über das Inhaltsfeld?

Weil der Client nichts anderes lesen kann. Der mitgelieferte Client
deserialisiert die Antwort und greift ausschließlich auf dieses eine Feld zu.
Ein zusätzliches Feld im JSON käme dort nie an.

Statt den Client umzubauen, wurde alles durch den einen Kanal getunnelt, den er
beherrscht. Das klingt nach einem Kompromiss, ist aber genau die Entscheidung,
die den Server client-unabhängig macht: Jeder Client, der das verbreitete
Chat-Format spricht, spricht damit automatisch auch das volle Protokoll - ohne
eine Zeile Sondercode.

## Richtungen

Die Token-Familien sind strikt nach Richtung getrennt. Es gibt eine Familie für
den Weg vom Client zum Server, eine für den Weg vom Modell über den Server zum
Client, und eine für Modellausgaben, die den Server nie verlassen.

Die Trennung ist nicht nur Ordnung. Sie macht **Reflexion unmöglich**: Ein Token
der einen Richtung ist auf dem Pfad der anderen kein gültiges Token, sondern
Text. Wege zurück gäbe es genug - ein wiedergespielter Gesprächsverlauf, ein
Client der echot, ein Protokoll das wieder in den Prompt wandert. Mit getrennten
Polen fällt eine ganze Klasse von Verwechslungen weg, ohne dass irgendwo eine
Prüfung dafür stehen müsste.

## Das Modell schlägt vor, der Server entscheidet

Kein Token führt sich selbst aus. Was das Sprachmodell erzeugt, ist ein
**Vorschlag in Textform** - nicht mehr. Der Server liest ihn, prüft ihn gegen
Regeln, die dem Modell nie gezeigt wurden, und führt ihn dann aus oder eben
nicht.

Dieselbe Trennung gilt eine Ebene tiefer. Ein optionales kleines Sprachmodell
pflegt im Hintergrund die Nutzerdaten - auch dieses schreibt nichts. Es liefert
strukturierte Operationsvorschläge, das eigentliche Schreiben passiert
deterministisch im Server-Code.

Der Unterschied klingt akademisch, ist aber der ganze Punkt. Ein Modell, das
Werkzeuge direkt aufruft, ist eine ausführende Instanz. Ein Modell, das Text
erzeugt, den ein Programm bewertet, ist eine vorschlagende Instanz. Nur bei der
zweiten Variante lässt sich sagen, was schlimmstenfalls passieren kann - nämlich
höchstens das, was der Server ohnehin zulässt.

## Die Rückkopplung

Bisher klang das nach einer Einbahnstraße: Das Modell schlägt eine Aktion vor,
der Server führt sie aus, fertig. Der interessantere Fall ist der, in dem der
Vorschlag nicht nach außen wirkt, sondern nach innen - auf die Daten, aus denen
die nächste Antwort entsteht.

Der Agent kann am Ende einer Antwort einen **Zustandsvermerk** anhängen. Er ist
für den Chat unsichtbar; der Server schneidet ihn heraus, bevor irgendetwas
versendet wird. Zwei Arten von Angaben stehen darin:

| Was vermerkt wird | Wirkung |
| --- | --- |
| Der eigene Zustand - Stimmung, Energie, Anspannung, aktuelle Tätigkeit | Der Agent bleibt nicht auf seinen Startwerten stehen, sondern entwickelt sich über einen Stream hinweg |
| Die Einschätzung der Person, mit der gerade gesprochen wird, samt einer freien Notiz | Wiederkehrende Zuschauer werden nicht jedes Mal wie Fremde behandelt |

Der Kreis schließt sich beim nächsten Durchlauf: Was der Server geschrieben hat,
stellt er beim Aufbau des nächsten Prompts wieder bereit. Aus „Modell schreibt
Vermerk, Server speichert, Server liest beim nächsten Mal vor" entsteht von
außen das, was wie Erinnerung und wie ein Verhältnis zu einzelnen Leuten
aussieht. Ohne diese Schleife bliebe die Persona statisch - dieselbe Stimmung,
dieselbe Distanz zu jedem, unabhängig davon, was vorher passiert ist.

Entscheidend ist, dass die Regel dabei unverändert gilt. Der Vermerk ist ein
**Vorschlag in Textform**, kein Schreibzugriff. Welche Felder überhaupt
beschreibbar sind, welche Werte zulässig sind, wessen Datensatz betroffen ist -
das entscheidet der Server. Einige Angaben werden nach dem ersten Setzen
gesperrt, damit das Modell sie später nicht überschreiben kann.

Das ist das schärfere Beispiel für die Trennung als jede Moderationsaktion. Bei
einem Timeout ist offensichtlich, dass ein Programm prüfen sollte. Hier wirkt
das Modell auf die Daten, aus denen sein eigener nächster Prompt gebaut wird -
und selbst dort schreibt es nicht selbst.

Was daraus für die gespeicherten Daten folgt, steht auf der
[Datenseite](daten.md).

## Warum keine Tool-Calls?

Die naheliegende Frage: Es gibt Function- und Tool-Calling, warum werden hier
Zeichenketten geparst? Vier Gründe.

### Tool-Calling setzt *einen* Prinzipal voraus

Die Semantik von Tool-Calling lautet sinngemäß: „Der Nutzer hat es angefragt,
also führe es aus." Das trägt bei einem Assistenten mit einem Nutzer.

Hier schreiben **N wechselseitig fremde Leute in ein Kontextfenster**, und
öffentlicher Chat ist per Definition feindlicher Input. In diesem Setting
verwechselt Tool-Calling „das Modell hat es angefordert" mit „ein berechtigter
Prinzipal hat es angefordert". Bei einem einzelnen Nutzer ist das dasselbe; bei
vielen fremden Sprechern in einem gemeinsamen Kontext sind es zwei verschiedene
Dinge.

### Keine Schema-Offenlegung

Eine Tool-Definition liefert dem Modell die Angriffsfläche als Landkarte mit:
Namen, Parameter, erlaubte Werte. Eine Injection hat damit ein wohlgeformtes
Ziel - sie muss nicht raten, sie liest ab.

Bei Token weiß das Modell, *dass* eine Fähigkeit existiert und wofür sie da ist
- aber nicht, wie daraus eine Aktion wird. Ob der Typ im aktuellen Kontext
überhaupt zulässig ist, wie das Ziel aufgelöst wird, welche Grenzen greifen:
Diese Semantik lebt vollständig im Server und ist dem Modell nie exponiert.

### Fail-Closed

Ein unbekannter Tool-Call ist in den meisten Frameworks ein Fehler. Er wird
gemeldet, oft mit Wiederholungsversuch, und erzeugt Signal - Signal, an dem sich
ein Angreifer entlanghangeln kann.

Ein unbekanntes Token dagegen passt auf kein Muster und bleibt Text. Ein
bekanntes Token mit unbekanntem Typ läuft ins Leere. In beiden Fällen ist das
Ergebnis dasselbe: es passiert nichts, und es gibt keine Rückmeldung, aus der
sich etwas lernen ließe. Bei feindlichem Input ist das der einzig richtige
Standardfall.

### Modell-Portabilität und Tempo

Tool-Calling ist keine Basisfähigkeit eines Sprachmodells, sondern antrainiertes
Verhalten plus ein anbieterspezifisches Format. Token emittieren kann dagegen
jedes Modell, das einer Anweisung folgt - auch ein kleines, lokal laufendes.
Emittiert ein schwächeres Modell Unsinn, greift Fail-Closed, und es bleibt eine
normale Chat-Antwort. Der Ausfallmodus ist „weniger fähig", nicht „defekt".

Das ist zugleich der Grund, warum sich der Betrieb ohne Cloud-Anbieter überhaupt
anbietet: Die Steuerungsebene stellt keine Anforderung, die nur große Modelle
erfüllen.

Dazu spart es einen Durchlauf. Kanonisches Tool-Calling ist Anfrage → Tool-Call
→ Ausführung → Ergebnis zurück → finale Generierung, also zwei Inferenzen. Hier
entstehen Aktion und Antworttext in **einer**. Da der Server Anfragen global
serialisiert, zählt das doppelt: halbierte Aufrufe pro Befehl heißen auch
Durchsatz für alle, die gerade warten.

> Ehrlicherweise gilt dieser letzte Vorteil nur, solange die Aktion nicht
> *zurück* in die Antwort fließen muss. Ein Moderationsbefehl ist
> fire-and-forget. Eine künftige Erweiterung, die ein Ergebnis liefert, über das
> der Agent anschließend reden soll, braucht zwangsläufig eine zweite Inferenz -
> unabhängig davon, welcher Mechanismus sie auslöst.

## Weiter

- [Sicherheit](sicherheit.md) - das Bedrohungsmodell und wie die Zusagen
  durchgesetzt werden
- [Daten](daten.md) - was gespeichert wird und was die Maschine verlässt
