# Sicherheit

[Deutsch (Hier)] | [English](../en/security.md) | [Übersicht](../index.md)

---

Diese Seite setzt die [Technikseite](technik.md) voraus - insbesondere die
Trennung zwischen Vorschlag und Ausführung.

## Das Bedrohungsmodell

Die meisten Absicherungen von KI-Assistenten gehen von einem Nutzer aus, der dem
Assistenten wohlgesonnen ist. Der Nutzer hat ein Ziel, der Assistent hilft, und
das Risiko liegt darin, dass das Modell etwas Unsinniges tut.

Hier ist die Lage eine andere. In einem öffentlichen Chat schreiben **viele
einander fremde Menschen gleichzeitig in ein gemeinsames Kontextfenster**. Sie
kennen sich nicht, sie verfolgen unterschiedliche Absichten, und ein Teil von
ihnen probiert aus, was passiert. Öffentlicher Chat ist damit nicht
„gelegentlich problematischer Input", sondern per Definition feindlicher Input.

Daraus folgt die Frage, die dieses Papier beantworten will: Was kann eine
einzelne Person, die beliebigen Text schreiben darf, schlimmstenfalls auslösen?

## Die Invarianten

Drei Regeln tragen die Sicherheit des Verfahrens. Sie sind bewusst so knapp
gehalten, dass man ihre Verletzung beim Lesen bemerkt.

**1. Das Modell schlägt vor, der Server entscheidet.** Kein Token führt sich
selbst aus. Ausführlich auf der [Technikseite](technik.md).

**2. Der Handelnde kommt aus dem Transport, nie aus der Modellausgabe.** Wer
eine Aktion auslöst, ergibt sich aus dem Anfragekontext - also daraus, wer die
Nachricht nachweislich geschickt hat. Das *Ziel* einer Moderationsaktion darf
aus den Parametern des Vorschlags kommen; der *Akteur* nie. Ohne diese Trennung
könnte jeder, der Text schreiben darf, im Namen eines anderen handeln.

**3. Systemereignisse kommen nur von einem reservierten Absender.** Ereignisse
wie Raids oder Werbepausen werden nur akzeptiert, wenn sie als von diesem
Absender stammend gekennzeichnet sind. Aus dem Chat heraus ist das nicht
fälschbar, weil der Client jeder Zuschauernachricht den echten Namen voranstellt

- die Kennzeichnung sitzt dann nicht mehr an der Stelle, an der sie zählt.

Zusätzlich ist der Name reserviert und kann nie als echter Chatter existieren,
der Namensraum ist also kollisionsfrei.

Ergänzend: Steuerblöcke werden entfernt, bevor eine Antwort in den
Gesprächsverlauf geschrieben wird. Steuerungstext kann sich damit nicht über den
Verlauf selbst reinjizieren.

## Struktur statt Filter

Das Verfahren steht und fällt mit einer Zusage: **Nutzertext darf nicht als
Steuerung gelesen werden.** Andernfalls tippt jemand einen Befehl in den Chat
und der Server nimmt ihn ernst.

Bemerkenswert ist, wie das gelöst ist - nämlich **nicht** durch Filtern.

Steuertoken werden aus eingehendem Text nicht entfernt. Sie müssen es auch
nicht. Der Client stellt jeder Zuschauernachricht den echten Namen voran; die
Nachricht beginnt also nie mit einem Token, sondern immer mit dem Namen. Die
Prüfung des Servers ist eine Prüfung auf den Anfang der Zeichenkette - sie
greift schlicht nicht. Nicht, weil etwas herausgefiltert wurde, sondern weil das
Token nie an der Stelle steht, an der es zählen würde.

Der Unterschied zu einer Filterliste ist wesentlich:

| | Filterliste | Struktur |
| --- | --- | --- |
| Deckt ab | jedes Token, an das jemand gedacht hat | auch Token, die es noch nicht gibt |
| Bei neuem Token | muss erweitert werden | unverändert gültig |
| Fehlerfall | ein vergessener Eintrag reißt ein Loch | kein Einzeleintrag, der fehlen könnte |
| Sichtbarkeit des Fehlers | still | - |

Eine Liste muss mit jedem neuen Token wachsen, und wer eines vergisst, merkt es
nicht. Die strukturelle Variante gilt automatisch auch für Token, die zum
Zeitpunkt ihrer Formulierung noch nicht erfunden waren.

## Was der Server trotzdem bereinigt

Zwei Dinge sind nicht strukturell abgedeckt und werden deshalb aktiv behandelt.
Beides passiert **serverseitig** - es gilt damit auch für selbstgebaute Clients,
die es weggelassen haben oder falsch machen.

**Platzhaltersyntax.** Der Systemprompt arbeitet mit Platzhaltern, und
Nutzertext landet über den Gesprächsverlauf im selben Prompt. Zeichen, mit denen
sich Platzhalter nachbauen ließen, werden deshalb beim Eingang durch optisch
gleichwertige, aber funktionslose Varianten ersetzt. Die Ersetzung ist
idempotent: Hat ein Client sie bereits vorgenommen, passiert nichts weiter, alte
und neue Clientversionen lassen sich mischen.

**Das Format des Gesprächsverlaufs.** Der Verlauf ist ein zweiter In-Band-Kanal

- eine Zeile pro Gesprächszug, mit Absender und Zeitstempel. Genau das war eine

Lücke: Es genügte, in der eigenen Nachricht eine neue Zeile zu beginnen, um
weitere Gesprächszüge zu erfinden. Auch Antworten der Bot-Persona. Das
Sprachmodell ließ sich damit per In-Context-Learning auf eine Historie
konditionieren, die nie stattgefunden hat. Der Angriff brauchte keine
Sonderzeichen und keine Kenntnis interner Formate - er brauchte eine
Zeilenschaltung.

Behandelt wird das an der Stelle, an der das Verlaufsformat entsteht, und nicht
am Eingang. Damit gilt der Schutz auch für den Sprachpfad und für jeden
künftigen Aufrufer - dasselbe Prinzip wie „der Server entscheidet, nicht das
Modell".

## Auf dem Rückweg

Was das Sprachmodell erzeugt, darf umgekehrt nicht ungeprüft im Chat landen.
Interne Steuerblöcke werden vor dem Versand entfernt, einschließlich verwaister
Marker für den Fall, dass das Modell ein schließendes Tag vergisst. Alle
Ausgabewege laufen über dieselbe Funktion, damit kein Steuertoken über einen
Umweg sichtbar wird.

## Grenzen

Ein Sicherheitspapier ohne Grenzen ist Werbung. Drei Dinge gehören dazu:

**Die Zugriffsebene ist ein geteiltes Geheimnis.** Der Server prüft bei
Anfragen einen konfigurierbaren Schlüssel. Wird keiner gesetzt und der Dienst in
einem offenen Netz betrieben, ist er für jeden erreichbar, der ihn findet. Das
ist eine Betriebsentscheidung, die dieses Papier nicht abnehmen kann.

**Die Absicherung schützt den Server, nicht das Modell vor sich selbst.** Alles
oben Beschriebene stellt sicher, dass ein Zuschauer keine Aktion auslösen kann,
die ihm nicht zusteht. Es stellt nicht sicher, dass das Sprachmodell inhaltlich
sinnvoll antwortet oder sich nicht zu einer unpassenden Äußerung überreden
lässt. Das ist eine andere Problemklasse, und sie ist nicht gelöst.

**Der Client ist Teil der vertrauenswürdigen Zone.** Die tragende Zusage -
jeder Zuschauernachricht wird der echte Name vorangestellt - wird vom Client
eingelöst. Ein Client, der rohen Nutzertext ohne diese Kennzeichnung schickt,
hebelt die strukturelle Ebene aus. Der Server kann das nicht unterscheiden,
denn genau diese Unterscheidbarkeit ist das, was der Präfix herstellt.

## Weiter

- [Technik](technik.md) - wie das Protokoll funktioniert
- [Daten](daten.md) - was gespeichert wird und was die Maschine verlässt
