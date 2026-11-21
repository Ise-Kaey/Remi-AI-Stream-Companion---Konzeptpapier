# Daten

[Deutsch (Hier)] | [English](../en/data.md) | [Übersicht](../index.md)

---

Ein Agent, der sich an Leute erinnert, speichert Daten über Leute. Diese Seite
sagt, welche das sind, wie lange sie bleiben, und was davon die Maschine
verlässt.

## Was gespeichert wird

Acht Kategorien, alle lokal als Dateien auf der Maschine des Betreibers:

| Kategorie | Inhalt |
| --- | --- |
| **Gesprächsverlauf** | Was gesagt und geantwortet wurde, mit Absender und Zeitpunkt. Sprache und Chat teilen sich einen Verlauf |
| **Tagebuch** | Die verdichtete Fassung älterer Gesprächsabschnitte: eine kurze Erzählung aus Sicht des Agenten, dazu Stichpunkte - Themen, Stimmung, was offen blieb und was davon seither erledigt ist. **Stark verdichtet:** ein Abschnitt von bis zu rund 50 000 Wortbestandteilen wird zu einigen hundert, Einzelheiten wie Vorlieben gehen dabei unter. Wer im Gespräch vorkam, kann darin unter seinem Anzeigenamen genannt sein |
| **Zuschauerprofile** | Anzeigename, bevorzugte Anrede, hinterlegtes Geschlecht, optional Geburtstag, die gewünschte Sprache der Anrede, der Zeitpunkt der letzten Nachricht, dazu kategorisierte Merkmale („mag Katzen"), ein Freitextfeld und eine Einstufung der Person durch den Agenten (etwa *vertraut*, *fremd*, *skeptisch*). Dazu drei **Zeitstempel**: wann jemand zum ersten Mal auftrat, wann er zuletzt begrüßt wurde und wann zuletzt zum Geburtstag gratuliert wurde |
| **Moderationsvorgänge** | Verstöße mit Zeitpunkt, Grund und ausgelöster Maßnahme, dazu eine Eskalationsstufe und der Zeitpunkt, zu dem sie zuletzt geändert wurde |
| **Gemeinschaftswissen** | Fakten über die Community als Ganzes, nicht einer Person zugeordnet |
| **Erfundene Vergangenheit** | Die Hintergrundgeschichte der Persona: Anekdoten, die der Agent selbst erfunden und als solche ausgezeichnet hat, damit er sich später nicht widerspricht. Sie betreffen die Figur, nicht die Zuschauer - als Person kommt darin in der Regel nur der Streamer vor, der im Systemprompt als ihr Gegenüber hinterlegt ist (etwa: wie sich beide kennengelernt haben) |
| **Stimmprofil** (optional) | Zahlenreihen, die die Stimme des **Streamers** beschreiben - errechnet aus sechs Aufnahmen, die er beim Einrichten selbst spricht. **Die Aufnahmen selbst werden nicht gespeichert**, nur diese Zahlen. Sie sind biometrische Daten (DSGVO Art. 9). Zuschauer betrifft das Profil nie; ihre Stimmen werden geprüft, aber nicht festgehalten, siehe [unten](#die-stimmprüfung-bleibt-auf-dem-gerät). Zur Gegenprobe kommen Zahlenreihen fremder Stimmen aus einem offenen Korpus dazu |
| **Betriebsprotokolle** | Ablauf des Betriebs - was fehlschlug, was ausgelöst wurde. Gesprächsinhalte stehen dort nicht, siehe [unten](#betriebsprotokolle) |

Bezeichner sind die Anzeigenamen aus dem Chat. Es werden keine Klarnamen,
E-Mail-Adressen, Zahlungsdaten oder Plattform-Identifikatoren erhoben - der
Server hat zu alldem gar keinen Zugang, siehe [Datensparsamkeit](#datensparsamkeit-im-entwurf).

Daneben liegt ein weiterer Datenbestand, der hier nur der Vollständigkeit halber
steht: der **Zustand des Agenten selbst** - seine Stimmung, sein Energieniveau,
seine aktuelle Tätigkeit, dazu Kennzahlen des laufenden Streams. Er enthält
keine personenbezogenen Daten und taucht deshalb in den folgenden Abschnitten
nicht mehr auf. Wie er entsteht und warum es ihn gibt, steht bei der
[Rückkopplung](technik.md#die-rückkopplung).

## Wie lange

| Kategorie | Aufbewahrung |
| --- | --- |
| Gesprächsverlauf | bis zu einem einstellbaren Umfang. Ist er erreicht, wird der ältere Teil zu einem Tagebucheintrag verdichtet und aus dem Verlauf genommen; der Wortlaut liegt danach noch eine Frist im Archiv (Vorgabe 14 Tage) und wird dann gelöscht |
| Tagebuch | unbefristet. Einträge, die älter als eine Woche sind (einstellbar), werden zu einem Rückblick zusammengefasst; die ersetzten Einträge liegen eine Frist im Archiv und werden dann gelöscht, der Rückblick bleibt |
| Zuschauerprofile | unbefristet, bis der Betreiber löscht oder die betroffene Person widerspricht |
| Moderationsvorgänge | die Eskalationsstufe sinkt zeitgesteuert von selbst; die Vorgänge selbst bleiben |
| Gemeinschaftswissen | unbefristet |
| Erfundene Vergangenheit | unbefristet |
| Stimmprofil | bis der Streamer es im Menü löscht. Es gibt **keine Sicherungskopien**: Löschen entfernt es vollständig |
| Betriebsprotokolle | feste Frist, ältere Dateien werden beim Start gelöscht |

Der **Wortlaut** eines Gesprächs bleibt damit nur befristet - was bleibt, ist
seine verdichtete Fassung im Tagebuch. Bis 0.6.5.15a stand hier, ältere
Einträge fielen einfach heraus; seither werden sie zusammengefasst, und die
Zusammenfassung geht mit jeder Anfrage an das Sprachmodell mit. Profile,
Tagebuch und erfundene Vergangenheit bleiben - das ist ihr Zweck, und es ist
der Punkt, an dem ein Betreiber eine bewusste Entscheidung treffen muss.

**Zur Größenordnung.** Im Wortlaut hält der Agent einen begrenzten Ausschnitt
vor, dazu für die Frist des Archivs die zuletzt verdichteten Abschnitte. Zum
Vergleich: Die Streaming-Plattform selbst hält den vollständigen Chatverlauf
üblicherweise über Wochen vor. **Das Tagebuch dagegen reicht weiter zurück**
als beides - nicht im Wortlaut, aber als Zusammenfassung über die ganze
Betriebszeit.

Das ist keine Rechtfertigung - der Betreiber ist für seine eigene Verarbeitung
verantwortlich, unabhängig davon, was die Plattform tut, und beide sind
getrennte Verantwortliche mit getrennten Zwecken. Aber es ordnet die
Größenordnung ein, um die es hier geht.

### Betriebsprotokolle

Ein Dienst, der läuft, schreibt mit, was er tut. Diese Protokolle sind die
einzige Kategorie, die weder Zweck noch Nutzen im Gedächtnis des Agenten hat -
sie existieren für Fehlersuche und Nachvollziehbarkeit. Genau deshalb sind sie
der Ort, an dem personenbezogene Daten am ehesten unbemerkt liegen bleiben.

Drei Maßnahmen greifen dort:

| | |
| --- | --- |
| **Nicht schreiben** | Zeilen, die Chat-Nachrichten, Sprachtranskripte, Antworten oder Profildaten führen würden, sind an ihrer Entstehungsstelle als personenbezogen gekennzeichnet und werden verworfen, bevor sie in die Datei gelangen |
| **Pseudonymisieren** | Was übrig bleibt - vor allem Moderationsvorgänge, die aus Nachvollziehbarkeitsgründen dokumentiert bleiben sollen - trägt statt des Anzeigenamens eine gesalzene, stabile Kennung |
| **Befristen** | Ältere Dateien werden gelöscht. Die Voreinstellung orientiert sich an der Aufbewahrung der Streaming-Plattform; sie lässt sich verkürzen oder ganz abschalten |

Bemerkenswert an der ersten Maßnahme ist, dass sie **nicht** über Mustererkennung
läuft. Es wird nicht versucht, aus einer fertigen Logzeile herauszufiltern, was
darin nach einem Namen aussieht - sondern beim Schreiben der Zeile wird
entschieden, ob ihr Inhalt personenbezogen ist. Wer später eine neue Logmeldung
ergänzt, trifft diese Entscheidung an genau der Stelle, an der er den Inhalt
kennt. Dieselbe Überlegung wie beim Verzicht auf Filterlisten in der
[Sicherheit](sicherheit.md#struktur-statt-filter): eine Erkennung, die raten
muss, versagt still.

Nicht erfasst ist die Protokolldatei, die nur der Entwicklung dient - sie wird
bei jedem Start überschrieben und existiert damit ohnehin nur für die Dauer
einer Sitzung.

## Was die Maschine verlässt

Der Agent besteht aus vier Verarbeitungsstufen. Sie unterscheiden sich darin,
ob Daten das Gerät verlassen:

| Stufe | Wo sie läuft | Verlässt Daten die Maschine? |
| --- | --- | --- |
| **Spracherkennung** | frei konfigurierbar, **Vorgabe lokal** | **Kommt darauf an.** In der Vorgabe wird Audio auf der CPU in Text gewandelt und verlässt das Gerät nie. Stellt der Betreiber einen Cloud-Erkenner ein, geht der aufgenommene Ton dorthin |
| **Gedächtnispflege** | frei konfigurierbar, **Vorgabe lokal**, standardmäßig aus | **Kommt darauf an.** Sie muss vom Betreiber ausdrücklich eingeschaltet werden, und in der Vorgabe läuft sie auf dem Gerät. Stellt der Betreiber einen Endpunkt ein, geht dorthin, was die jeweilige Aufgabe braucht — bei der Faktenpflege die Äußerung und ein Auszug des Verlaufs, bei der Verdichtung **ein ganzer Sitzungsabschnitt** (siehe unten) |
| **Haupt-Sprachmodell** | frei konfigurierbar | **Kommt darauf an.** Zeigt die Konfiguration auf einen Cloud-Anbieter, gehen Daten dorthin. Zeigt sie auf ein selbst betriebenes Modell, nicht |
| **Sprachausgabe** | frei konfigurierbar | **Kommt darauf an.** Mit einem Cloud-Adapter geht der gesprochene Satz dorthin - und nur er. Mit einem lokalen Adapter wird auf der CPU vertont, und nichts verlässt das Gerät |

Für den Standardfall - ein Cloud-Anbieter für Sprachmodell und Sprachausgabe,
Erkennung lokal - heißt das: Audio bleibt lokal, Text geht raus.

**Die Spracherkennung hat diese Wahl erst seit dieser Version, und sie geht in
die unbequeme Richtung.** Bis hierher stand an dieser Stelle, aufgenommenes
Audio verlasse das Gerät nie - es gab schlicht keine andere Möglichkeit. Jetzt
gibt es eine: die Erkennung läuft, wie die Sprachausgabe, über eine
austauschbare Schicht, und einer der wählbaren Erkenner ist ein Cloud-Dienst.
Wer ihn einstellt, überträgt **den Ton selbst**, nicht nur abgeleiteten Text -
und damit die Stimme des Sprechenden und alles, was im Raum mitgesprochen wird.

Drei Dinge begrenzen das, und alle drei sind Bauart, nicht Versprechen:

- **Die Vorgabe bleibt lokal.** Wer nichts umstellt, ändert nichts.
- **Übertragen wird nur, was der Sprachaktivitätserkenner als Sprache erkannt
  hat**, und erst ab dem Beginn einer Äußerung. Ein Mikrofon, an dem niemand
  spricht, sendet nichts.
- **Das Weckwort wird immer lokal erkannt.** Die Schleife, die dauernd
  mithört, bekommt bauartbedingt einen Erkenner auf dem Gerät - auch dann,
  wenn für alles andere ein Cloud-Dienst eingestellt ist. Andernfalls ginge
  jede Sprachregung im Raum hinaus.

**Die Sprachausgabe muss aber nicht in die Cloud.** Sie läuft über eine
austauschbare Schicht, und es stehen lokale Stimmen zur Wahl, die auf der CPU
vertonen - ohne Konto und ohne Netzverbindung. Wer eine davon einstellt,
überträgt für die Sprachausgabe **gar nichts**. Das ist eine Einstellung im
Menü, kein anderer Programmstand.

Solange ein Cloud-Adapter eingestellt ist, gelten zwei Einschränkungen, die in
beide Richtungen gehen. Er bekommt **deutlich weniger** als das Sprachmodell:
keinen Gesprächsverlauf, kein Profil, keine Moderationsdaten - nur den einen
Satz, der gesprochen werden soll. Und er hängt ausschließlich am Sprachpfad;
Chat-Antworten werden nicht vorgelesen und erreichen ihn nie.

Trotzdem ist „keine personenbezogenen Daten" zu weit gegriffen. Der Sprachpfad
greift auf denselben Verlauf zu wie der Chat. Spricht der Agent über das
Geschehen im Chat, kann der gesprochene Satz Namen und Einzelheiten über Dritte
enthalten - und genau dieser Satz wird übertragen.

### Ein Sonderfall, der eigens genannt gehört

Seit 0.6.5.9a kann die Gedächtnispflege eine Aufgabe übernehmen, die sich von
den übrigen unterscheidet: sie formt den **Anfang der Antwort**, während der
Sprecher noch spricht. Dafür liest sie den wachsenden Zwischenstand der
Erkennung mit — mehrfach je Äußerung, nicht einmal danach.

Läuft sie lokal, bleibt das auf dem Gerät wie alles andere. Zeigt der Betreiber
sie auf einen Endpunkt, ist der Unterschied zu den übrigen Stufen jedoch
wesentlich: es gehen **Bruchstücke aus dem laufenden Sprechen** hinaus, nicht
die fertige Äußerung. Wer eine Vertraulichkeitsabwägung trifft, sollte das
getrennt bewerten.

**Die Vorgabe ist aus.** Die Funktion muss ausdrücklich eingeschaltet werden,
und sie setzt eine eingeschaltete Gedächtnispflege voraus. Welcher Endpunkt
gilt, lässt sich je Aufgabe getrennt einstellen — eine Aufgabe darf lokal
bleiben, während eine andere über einen Dienst läuft.

Die Statusanzeige der Software führt dafür eine eigene Farbe: **blau heißt,
dass kein Dritter die Inhalte sieht.** Maßgeblich ist dabei der Zugriff, nicht
der Besitz — ein gemieteter Server in einem Rechenzentrum gilt als nicht
vertraulich, weil der Betreiber an die Daten kommt.

### Weitere Aufgaben, die je Endpunkt getrennt schaltbar sind

Die Gedächtnispflege ist inzwischen keine einzelne Tätigkeit mehr, sondern ein
Bündel. Diese Aufgaben senden anderes als die Faktenpflege, und wer eine
Vertraulichkeitsabwägung trifft, sollte sie einzeln ansehen:

| Aufgabe | Was hinausgeht, wenn sie auf einem Endpunkt läuft | Vorgabe |
| --- | --- | --- |
| **Verdichtung des Verlaufs** | ein **ganzer Sitzungsabschnitt** statt einer einzelnen Äußerung — in Stücken, damit er in das Kontextfenster passt. Das ist die größte Menge, die an dieser Stelle je übertragen wurde | an, sobald die Gedächtnispflege läuft |
| **Anfang der Antwort** | zusätzlich zum Zwischenstand der Erkennung auch **die Beschreibung der Figur selbst** — Persönlichkeit, Sprechweise, Regeln. Rund 6400 Wortbestandteile je Aufruf | aus |
| **Erfundene Vergangenheit** | eine **Anekdote, die der Agent selbst erfunden und als solche ausgezeichnet hat** - aus der Hintergrundgeschichte der Persona. Als Person kommt darin in der Regel nur der Streamer vor | aus |
| **Erledigt-Prüfung** | nach **jeder Antwort**, solange im jüngsten Tagebucheintrag etwas offen ist: die offenen Punkte und der letzte Wortwechsel - die Nachricht samt Absender und die Antwort darauf | an, sobald die Gedächtnispflege läuft |
| **Unterbrechen** | während der Agent spricht und jemand dazwischenredet: das Gesagte des Agenten, der Zwischenstand der Erkennung und ein Auszug des Verlaufs | aus |

**Die Figurenbeschreibung ist keine Personendatei**, und sie steht hier trotzdem:
sie ist der einzige Teil, den der Betreiber selbst geschrieben hat und über den
er womöglich nicht nachdenkt, wenn er einen Endpunkt einstellt. Was darin an
Dritten erwähnt wird, geht mit hinaus.

**Die erfundene Vergangenheit betrifft die Figur, nicht das Publikum.** Sie ist
die Hintergrundgeschichte der Persona - Herkunft, Erlebnisse, Eigenheiten -,
und der einzige Mensch, der darin regelmäßig vorkommt, ist der Streamer: er
steht im Systemprompt als ihr Gegenüber. Sie entsteht allerdings im Gespräch,
und ein Zuschauer ist darin nicht ausgeschlossen. Der Agent hält sie fest,
damit er sich beim nächsten Mal nicht widerspricht; dieselbe Datei kann also
über Wochen wachsen.

### Die Vorentscheidung im Chat

Seit 0.6.6.40000-alpha steht vor dem Haupt-Sprachmodell eine kleine
Vorentscheidung: Soll der Agent auf eine Chatnachricht überhaupt antworten?
Dafür gehen **die Chat-Regeln der Persona, die letzten 15 Einträge des Verlaufs
und die neue Nachricht** hinaus - an dasselbe Sprachmodell wie die Antwort, oder
an einen eigenen Endpunkt, wenn der Betreiber einen einstellt.

Sie ist **immer aktiv** und läuft in der Vorgabe über das Haupt-Sprachmodell.
Damit kommt kein neuer Empfänger hinzu: was dorthin geht, ist ein Ausschnitt
dessen, was die Antwort ohnehin bekäme. Ein **eigener Endpunkt** dagegen ist
ein weiterer Anbieter mit eigenem Ausschnitt - wer einen einstellt, sollte ihn
in der Auftragsverarbeitung mitzählen.

### Die Stimmprüfung bleibt auf dem Gerät

Seit 0.6.6.50000-alpha kann der Agent die Stimme des Streamers kennenlernen.
Der Streamer spricht dafür beim Einrichten sechs kurze Aufnahmen; daraus
entsteht das **Stimmprofil** - Zahlenreihen, keine Aufnahmen. Danach wird jede
Äußerung, die das Mikrofon aufnimmt, mit diesem Profil verglichen. In der
ersten Ausbaustufe nur gemessen: das Ergebnis steht als Prozentwert im
Betriebsprotokoll, verworfen wird nichts.

Drei Dinge gelten dabei, und alle drei sind Bauart:

- **Nichts davon verlässt das Gerät.** Das Modell, das die Stimme beschreibt,
  wird einmal heruntergeladen und rechnet danach auf der CPU. Es gibt keinen
  Cloud-Weg für diese Prüfung - auch dann nicht, wenn für die Erkennung ein
  Cloud-Dienst eingestellt ist.
- **Geprüft wird jede Stimme, festgehalten nur eine.** Auch eine fremde Stimme
  im Raum wird mit dem Profil verglichen, sonst könnte die Prüfung sie nicht
  erkennen. Für sie entsteht aber nichts: kein Profil, keine Zahlenreihe, nur
  eine Zeile mit dem Prozentwert im Protokoll, ohne Namen und ohne Inhalt.
- **Das Profil gehört dem Streamer und ist abschaltbar.** Die Funktion ist in
  der Vorgabe aus, das Profil entsteht nur, wenn er es selbst einrichtet, und
  „Stimmprofil löschen“ entfernt es ohne Sicherungskopie.

Der Widerspruchsweg der Zuschauer berührt das Profil nicht - es enthält keine
Zuschauerdaten.

Nach dem Einrichten zeigt eine **Gegenprobe**, wie gut das Profil trennt: Der
Agent vergleicht es mit Sätzen in seiner eigenen Stimme und mit 20 fremden
Stimmen. Dafür entstehen zwei lokale Bestände - die **Tonablage** mit den
einmal erzeugten Sätzen (ist die Stimme des Agenten der Klon einer echten
Person, ist es deren Stimme) und die **Gegenprobe** mit den Zahlenreihen der
fremden Stimmen. Diese stammen aus LibriSpeech, einem offenen Hörbuchkorpus
unter CC BY 4.0; abgeholt wird beim Einrichten **einmal** über Hugging Face,
der Ton wird verworfen, nur die Zahlen bleiben. Die Datensatzkarte verlangt,
die Sprecher nicht zu identifizieren - die Gegenprobe fragt nur, ob jemand
**nicht** der Streamer ist. Ist die Sprachausgabe ein Cloud-Dienst, gehen
außerdem **einmal** drei feste Sätze dorthin, nach Rückfrage.

### Vollständig lokaler Betrieb

Seit dieser Version lassen sich **alle vier Stufen auf dem Gerät betreiben**:

| Stufe | Lokal? |
| --- | --- |
| Spracherkennung | **wählbar** - die Vorgabe ist lokal, und das Weckwort wird immer lokal erkannt |
| Gedächtnispflege | **wählbar** - die Vorgabe ist lokal, und standardmäßig läuft sie gar nicht |
| Sprachausgabe | **wählbar** - lokale Stimmen stehen zur Verfügung |
| Haupt-Sprachmodell | **wählbar** - die Schnittstelle ist ein offener Standard, jedes selbst betriebene Modell mit dieser Schnittstelle genügt |

Damit ist „nichts verlässt das Gerät" keine theoretische Randstellung der
Skala mehr, sondern eine Konfiguration, die man tatsächlich einstellen kann.
Bis zur vorigen Version war sie unerreichbar: der gesprochene Satz musste immer
hinaus.

Bei der Spracherkennung ist „wählbar" allerdings anders gemeint als bei den
übrigen drei Zeilen. Dort ist die Vorgabe ein Cloud-Dienst und lokal die
Alternative; hier ist es umgekehrt. Die Zeile steht in der Tabelle, weil das
Papier auch die Wege benennen muss, die **aus** dem lokalen Betrieb
herausführen - nicht nur die hinein.

Zwei Einschränkungen, damit die Aussage trägt:

- **Modelle müssen einmal geladen werden.** Erkennung, Sprachausgabe und ein
  lokales Sprachmodell brauchen bei der Einrichtung eine Internetverbindung.
  Dabei gehen **keine Nutzerdaten** hinaus - es wird nur heruntergeladen. Im
  Betrieb danach ist keine Verbindung nötig.
- **Der Client ist eine eigene Frage.** Der Server hat keine Plattform-Anbindung
  (siehe [Datensparsamkeit](#datensparsamkeit-im-entwurf)); die Verbindung zu
  Twitch, YouTube oder Discord besteht beim Frontend. „Lokal" bezieht sich auf
  die Verarbeitung im Agenten, nicht auf die Chat-Plattform dahinter.

### Und der Gegenpol: vollständig fremder Betrieb

Dieselbe Wahlfreiheit gilt in die andere Richtung. Wer alle vier Stufen auf
Dienste stellt, betreibt den Agenten vollständig fremd — und das ist keine
theoretische Randstellung, sondern für manche der naheliegende Aufbau: ein
Rechner ohne Grafikkarte, alles über Endpunkte.

Dann verlässt **praktisch alles** das Gerät, und zwar mehr, als die Summe der
Einzelzeilen vermuten lässt:

- **der aufgenommene Ton** an den Erkenner — nicht abgeleiteter Text, sondern
  die Stimme selbst und alles, was im Raum mitgesprochen wurde
- **der gesamte Gesprächsverlauf** mit jedem Zug an das Sprachmodell
- **der gesprochene Satz** an die Sprachausgabe
- **Abschnitte, Figurenbeschreibung und erfundene Vergangenheit** an die
  Gedächtnispflege, je nachdem welche ihrer Aufgaben eingeschaltet sind

Das kann an **vier verschiedene Anbieter** gehen. Jeder sieht einen anderen
Ausschnitt, keiner das Ganze — aber jeder sieht genug, um Personen
wiederzuerkennen.

**Eines bleibt bauartbedingt lokal, auch in dieser Einstellung: das Weckwort.**
Die Schleife, die dauernd mithört, bekommt immer einen Erkenner auf dem Gerät.
Andernfalls ginge jede Sprachregung im Raum hinaus, auch die, die niemandem
galt. „Vollständig fremd" heißt also: alles, was der Agent verarbeitet — nicht
alles, was das Mikrofon hört.

**Zwischen den beiden Polen liegt der Normalfall.** Die vier Stufen sind
einzeln einstellbar, und die Gedächtnispflege noch einmal je Aufgabe. Die
üblichen Aufbauten mischen: Erkennung lokal, Sprachmodell und Stimme über
Dienste, Gedächtnispflege aus. Das Papier nennt die Pole, weil sie die Grenzen
des Möglichen abstecken — nicht, weil jemand sie einstellen müsste.

## Was in den Prompt geht

Das ist die wichtigste Aussage dieser Seite, und sie wird oft übersehen: **An
das Haupt-Sprachmodell geht nicht nur die aktuelle Nachricht.**

Vor jeder Anfrage stellt der Server einen Kontext zusammen. Darin stehen unter
anderem:

- der jüngste Gesprächsverlauf, also auch Beiträge anderer Personen
- das Profil der sprechenden Person samt gespeicherter Merkmale
- das Profil einer zweiten Person, wenn sie in der Nachricht erwähnt wird
- ob für diese Person eine Moderationsstufe aktiv ist
- Zustandsdaten des Streams und der Persona

Wer also einen Cloud-Anbieter für das Sprachmodell nutzt, überträgt damit die
gespeicherten Zuschauerdaten an diesen Anbieter - nicht als Datenbank-Export,
aber laufend und in Ausschnitten. Das ist keine Nebenwirkung, sondern der
Mechanismus, durch den der Agent sich überhaupt an Leute erinnert.

Wer das nicht möchte, kann das Sprachmodell auf einen selbst betriebenen
Endpunkt zeigen. Zusammen mit einer lokalen Stimme und der lokalen Erkennung -
der Vorgabe - verlässt dann **nichts mehr** die Maschine: Erkennung, Modell und
Sprachausgabe laufen alle auf dem Gerät.
Bis dahin blieb der Satz für die Sprachausgabe als einziger Abfluss übrig; er
ist es nicht mehr. Die Steuerungsebene ist bewusst so entworfen, dass sie auch
mit kleinen lokalen Modellen funktioniert - siehe
[Modell-Portabilität](technik.md#modell-portabilität-und-tempo).

### Vier Abstufungen

Zwischen „alles beim Anbieter" und „nichts verlässt das Gerät" liegen zwei
mittlere Stufen; die zweite dürfte in der Praxis die häufigste sein. Die
unterste ist seit der lokalen Sprachausgabe **wörtlich** zu nehmen:

| Aufbau | Wohin Zuschauerdaten gehen |
| --- | --- |
| Sprachmodell bei einem Endkunden-Anbieter | An den Anbieter, zu dessen Bedingungen - die Aufbewahrung und eine mögliche Verwendung zum Training richten sich nach dem Tarif |
| Sprachmodell über einen Geschäftszugang mit Auftragsverarbeitungsvertrag, ohne Protokollierung und ohne Trainingsfreigabe | An den Anbieter, aber als weisungsgebundener Auftragsverarbeiter |
| Sprachmodell selbst betrieben, Sprachausgabe in der Cloud | Nirgendwohin - bis auf den gesprochenen Satz |
| Sprachmodell selbst betrieben, Sprachausgabe lokal | **Nirgendwohin.** Nichts verlässt das Gerät |

Die Tabelle setzt die Erkennung in ihrer Vorgabe voraus, also lokal. Wird
stattdessen ein Cloud-Erkenner eingestellt, verschiebt das **jede** Zeile: dann
geht Ton hinaus, und zwar unabhängig davon, wo Sprachmodell und Sprachausgabe
laufen. Die unterste Zeile gilt dann nicht mehr.

Die mittlere Stufe ist der Punkt, an dem sich die Frage nach der
Auftragsverarbeitung sauber lösen lässt, statt sie zu umgehen: Vertrag,
Zweckbindung, keine Weiterverwendung. Die untere Stufe lässt sie ganz entfallen.
Die Wahl liegt beim Betreiber, und sie ist eine Konfigurationsfrage - kein
anderer Programmstand.

## Widerspruch

Betroffene können der Speicherung ihrer personenbezogenen Daten über einen
Chat-Befehl widersprechen. Der Befehl läuft deterministisch im Server und geht
gar nicht erst durch das Sprachmodell.

**Er kommt allerdings vom Frontend, nicht vom Server.** Der Server erkennt ihn
als eigene Befehlsart, strikt getrennt von allem, was das Sprachmodell erzeugt -
gesendet werden muss er aber von der Anbindung an die Chat-Plattform. Die
mitgelieferte tut das. Wer eine eigene baut, muss den Befehl weiterreichen,
sonst läuft der Widerspruchsweg ins Leere, während dieses Papier ihn zusagt.

Er **leert und sperrt** die betroffenen Felder. Das Sperren ist dabei der
wichtigere Teil: Ein bloßes Löschen würde bedeuten, dass die Gedächtnispflege
dieselben Angaben beim nächsten Gespräch erneut sammelt. Ein gesperrtes Feld
kann nicht wieder befüllt werden, auch nicht vom Modell.

**Was dabei erhalten bleibt, und warum.** Der Widerspruch ist nicht vollständig.
Das ist die wichtigste Einschränkung dieser Seite, deshalb hier vollständig
aufgeschlüsselt:

| Feld | Beim Widerspruch |
| --- | --- |
| Geburtstag | geleert und gesperrt |
| Gesammelte Merkmale | geleert und gesperrt |
| Freitextfeld | geleert und gesperrt |
| Anzeigename | **bleibt** |
| Bevorzugte Anrede | **bleibt** |
| Hinterlegtes Geschlecht | **bleibt** |
| Einstufung durch den Agenten | **bleibt** |
| Zeitpunkt des ersten Auftretens | **bleibt** |
| Zeitpunkt der letzten Nachricht | **bleibt** |

Die Trennlinie folgt einem Prinzip: Geleert wird, was der Agent über eine Person
*angesammelt* hat. Erhalten bleibt, was der Betrieb braucht, um überhaupt noch
richtig zu funktionieren - Anzeigename und Anrede für die korrekte Ansprache,
das Geschlecht für die Pronomen, der Zeitpunkt des ersten Auftretens für die
Zuordnung. Ohne Namen keine Zuordnung, ohne Zuordnung keine Sperre.

**Der Zeitpunkt der letzten Begrüßung bleibt aus einem Grund, der gegen den
ersten Eindruck spricht: er verhindert Zudringlichkeit.** Er ist der Merker
dafür, dass jemand *heute schon* begrüßt wurde - ohne ihn würde der Agent bei
jeder Nachricht aufs Neue willkommen heißen. Er speichert keinen Inhalt,
sondern nur, dass etwas bereits geschehen ist. Wer ihn löschte, bekäme nicht
weniger Aufmerksamkeit, sondern mehr.

**Der Zeitpunkt der letzten Nachricht bleibt aus demselben Grund.** Er sagt dem
Agenten, ob jemand gerade im Gespräch ist oder lange weg war. Ohne ihn würde die
Person behandelt, als käme sie gerade neu dazu. Er speichert keinen Inhalt, nur
einen Zeitpunkt, und wird bei jeder Chatnachricht überschrieben; ein Verlauf
vergangener Besuche entsteht daraus nicht.

**Der Geburtstagsvermerk wird dagegen mitgelöscht**, und das ist die
Gegenprobe auf dasselbe Prinzip: Er hält fest, wann zuletzt gratuliert wurde,
und setzt einen bekannten Geburtstag voraus. Ist der geleert und gesperrt, hat
der Vermerk keinen Bezug mehr - er wäre eine Spur ohne die Sache, auf die sie
zeigt. Was seine Grundlage verliert, bleibt nicht stehen.

Ein Feld verdient eine eigene Begründung: die **Einstufung**. Sie unterscheidet
sich in der Herkunft von allen übrigen - sie ist keine Angabe der betroffenen
Person, sondern ein Urteil des Agenten über deren Verhalten. Genau deshalb
bleibt sie erhalten.

Würde der Widerspruch sie zurücksetzen, wäre er ein Reset-Knopf: Wer den Chat
stört, widerspricht und wird anschließend wieder wie ein unbeschriebenes Blatt
behandelt - herzlich, arglos, bis von vorn. Die Einstufung ist die Erinnerung
daran, wie sich jemand **anderen gegenüber** verhalten hat, und sie schützt
nicht den Agenten, sondern die übrigen Anwesenden. Das Recht, eigene Daten
löschen zu lassen, erstreckt sich nicht auf die Folgen des eigenen Verhaltens
für Dritte.

Wichtig dabei, und der Grund, warum das kein Stigma ist: **Die Einstufung ist
eine Momentaufnahme, keine Akte.** Sie wird laufend aus dem Verhalten gegenüber
dem Agenten und den übrigen Anwesenden fortgeschrieben und bewegt sich in beide
Richtungen. Wer sich anders verhält, wird anders eingestuft. Sie rastet nicht
ein, und sie kennt keinen Endzustand - dieselbe Beweglichkeit, mit der auch die
Moderationsstufe von selbst wieder sinkt.

Bleibt die ehrliche Restaussage: Entfernen lässt sich die Einstufung nicht auf
Zuruf. Ändern schon - nur eben über den Weg, über den sie entstanden ist. Das
ist gewollt und an dieser Stelle begründet, nicht übersehen.

**Dasselbe gilt für die Moderationsvorgänge**, und es steht hier, weil es im
Widerspruch sonst niemand vermutet: Festgehaltene Verstöße mit Zeitpunkt,
Grund und ausgelöster Maßnahme bleiben. Die Eskalationsstufe sinkt
zeitgesteuert von selbst - aber das ist Zeitablauf, nicht Widerspruch.

Der Grund ist derselbe wie bei der Einstufung, nur deutlicher: Eine
Moderationsakte, die sich auf Zuruf leeren ließe, wäre kein Datenschutz,
sondern ein Reset-Knopf für das eigene Verhalten gegenüber Dritten. Ein
Widerspruch ist kein Freibrief für die Missachtung der Chatregeln.

**Sie ist aber kein Endzustand, und das auf zwei Wegen.** Die Eskalationsstufe
**verblasst von selbst** - je Stufe rund einen Monat ohne neuen Vorgang, und
danach ist sie eine Stufe niedriger. Wer sich länger nichts zuschulden kommen
lässt, steht irgendwann wieder bei null, ohne dass jemand etwas tun müsste.

Und sie lässt sich **aktiv zurücknehmen**: eine Begnadigung senkt die Stufe
sofort um eine. Sie kann vom Agenten selbst ausgehen - er darf begnadigen wie
verwarnen - oder vom Betreiber. Der Vorgang bleibt dabei protokolliert; was
zurückgenommen wird, ist seine **Wirkung**, nicht der Eintrag.

**Dass der Agent selbst begnadigen darf, ist eine Entwurfsentscheidung und
keine Bequemlichkeit.** Ein Moderationssystem, das nur eskalieren kann, ist eine
Einbahnstraße: Jeder Vorgang schiebt die Stufe nach oben, und nur Zeit schiebt
sie zurück.

Der Alltagsfall dahinter ist klein. Jemand spammt Emotes und bekommt eine
Verwarnung - Stufe eins, ohne weitere Folge. Er entschuldigt sich, und der
Agent nimmt sie im selben Gespräch zurück; die Stufe fällt auf null. Ohne
diesen Weg stünde die Verwarnung einen Monat lang da, obwohl die Sache im
nächsten Satz geklärt war.

Die Absicht ist damit ausgesprochen: **Jeder hat eine zweite Chance**, und sie
soll dort stattfinden können, wo der Verstoß stattfand - im Gespräch, nicht
als Verwaltungshandlung.

Damit gilt für die Moderation dieselbe Beweglichkeit wie für die Einstufung:
sie rastet nicht ein. Was bleibt, ist das Protokoll - was sich ändert, ist die
Folge daraus.

### Verlauf, Tagebuch und erfundene Vergangenheit bleiben unberührt

Der Widerspruch wirkt auf das Profil, nicht auf den laufenden Verlauf. Das ist
kein Versehen, sondern folgt aus dem, was der Verlauf ist: ein
**Gesprächsprotokoll**, kein Datensatz über eine Person. Die Zeilen darin
beziehen sich aufeinander. Nimmt man eine Stimme heraus, bleiben Antworten ohne
Frage stehen und Bezüge laufen ins Leere - und genau diese Bezüge sind es, die
eine Reaktion passend machen. Ein bereinigter Verlauf wäre kein datensparsamerer
Verlauf, sondern ein defekter.

Bis 0.6.5.15a entschärfte das die Aufbewahrung selbst: der Verlauf rollte in
einem aktiven Chat binnen Stunden durch, und die Verdrängung erledigte
schneller, was ein Löschbegehren erst hätte anstoßen müssen. **Das gilt nicht
mehr.** Seither rollt der Verlauf nicht, er wird ab einem einstellbaren Umfang
geschnitten, und der ältere Teil wird zu einem Tagebucheintrag verdichtet. Der
Wortlaut verschwindet nach der Frist des Archivs (Vorgabe 14 Tage) - die
Zusammenfassung bleibt, und mit ihr, wer darin genannt ist.

**Der Widerspruch erreicht das Tagebuch nicht, und ebenso wenig die erfundene
Vergangenheit.** Er gilt dem, was der Agent im Chat über eine Person
*angesammelt* hat - Vorlieben, Merkmale, Freitext im Profil. Das Tagebuch
ist keine solche Sammlung, sondern eine Zusammenfassung des Geschehens, und
die Verdichtung selbst begrenzt, was darin über Einzelne stehen kann: aus
bis zu rund 50 000 Wortbestandteilen werden einige hundert, und Vorlieben
überleben das nicht. Was bleibt, ist allenfalls, dass jemand unter seinem
Anzeigenamen am Gespräch teilnahm.

**Und dieser Name ist kein fester Bezug.** Jeder kann seinen Anzeigenamen
jederzeit ändern; anders als die Streaming-Plattform erkennt der Agent eine
Person unter einem geänderten Namen nicht wieder. Ein alter Tagebucheintrag
lässt sich dann keinem aktuellen Profil mehr zuordnen.

Die erfundene Vergangenheit ist die Hintergrundgeschichte der Persona und
betrifft Zuschauer in der Regel gar nicht. Beide sind **nicht befristet**;
wer dort trotzdem einen Eintrag entfernt haben möchte, wendet sich an den
Betreiber, siehe unten.

Der dritte Grund wiegt schwerer als die beiden ersten, und er gilt für den
Verlauf unverändert: Ein Widerspruch, der den
Verlauf mitnähme, wäre ein Freischein. Der Verlauf ist das Fenster, in dem ein
Verstoß überhaupt sichtbar ist, bevor er als Moderationsvorgang festgehalten
wird. Wer ihn auf Zuruf leeren könnte, könnte den Beleg für sein Verhalten
entfernen, bevor jemand darauf reagiert - stören, widersprechen, unbelastet
weitermachen.

Es ist dasselbe Prinzip wie bei der Einstufung, nur an der Stelle, an der es am
deutlichsten wird: Das Recht, eigene Daten löschen zu lassen, erstreckt sich
nicht darauf, Spuren des eigenen Verhaltens gegenüber Dritten zu beseitigen.

Wer eine vollständige Löschung möchte, muss sich an den Betreiber wenden - die
Daten liegen als Dateien auf dessen Maschine, und ein vollständiges Entfernen
ist dort eine administrative Handlung, keine Funktion des Agenten.

## Datensparsamkeit im Entwurf

Einiges wird nicht deshalb nicht gespeichert, weil es gelöscht würde, sondern
weil es nie ankommt:

- **Keine Plattform-Anbindung.** Der Server hat keine Zugangsdaten zu Twitch,
  YouTube oder Discord, keine Kanäle, keine Plattform-Schnittstelle. Er sieht
  nur, was ein Client ihm schickt.
- **Keine Ausführungsrechte.** Moderationsaktionen werden vom Server
  vorgeschlagen und an den Client übergeben. Ausgeführt werden sie dort. Der
  Server könnte niemanden sperren, selbst wenn er wollte.
- **Keine Identitäten.** Gespeichert werden Chat-Anzeigenamen. Ob dahinter eine
  reale Person steht und welche, weiß der Server nicht.
- **Keine sensiblen Angaben im Chat.** Die Chatregeln, die vor dem ersten
  Schreiben zu bestätigen sind, untersagen ausdrücklich, sensible Daten wie
  eine Adresse zu nennen. Was der Agent nie zu lesen bekommt, kann er weder
  speichern noch verdichten.

Das ist kein Zufall, sondern folgt aus der Architektur: Der Client liefert
Ereignisse und führt Aktionen aus, der Server hält den Zustand. Wo keine
Verbindung besteht, entstehen auch keine Daten.

## Was der Betreiber entscheiden muss

Diese Software ist kein Dienst. Sie läuft auf der Maschine einer Person, und
diese Person ist es, die gegenüber den Zuschauern verantwortlich ist. Das Papier
kann deshalb nicht erklären, ein Betrieb sei „datenschutzkonform" - das hängt
davon ab, wie er aufgesetzt wird.

Was ein Betreiber vor dem Produktivbetrieb für sich klären sollte:

- **Zweck und Rechtsgrundlage.** Warum werden Zuschauerdaten gespeichert, und
  worauf stützt sich das?
- **Transparenz.** Wissen die Zuschauer, dass ein Agent mitliest und sich Dinge
  merkt? Kennen sie den Weg zum Widerspruch?
- **Erkennbarkeit der KI.** Ist erkennbar, dass die Antworten von einem
  Sprachmodell stammen? Das ist eine eigene Pflicht aus einem eigenen
  Rechtsakt - sie fällt nicht mit der Datenschutz-Transparenz zusammen und
  gilt auch dann, wenn gar nichts gespeichert würde. Siehe unten.
- **Auftragsverarbeitung.** Welche Anbieter sind eingebunden, was übertragen
  sie, und liegen die nötigen Vereinbarungen vor? Es können **vier** sein --
  Erkennung, Sprachmodell, Stimme und Gedächtnispflege sind getrennt
  einstellbar und müssen nicht demselben Haus gehören. Bekommt die
  [Vorentscheidung im Chat](#die-vorentscheidung-im-chat) einen eigenen
  Endpunkt, ist es ein fünfter. Der Betrieb mit einem
  lokalen Sprachmodell verkleinert diese Frage erheblich; der vollständig
  lokale Betrieb lässt sie entfallen.
- **Auskunft und Löschung.** Wie wird eine Anfrage beantwortet, und wer kommt an
  die Dateien?
- **Aufbewahrung.** Profile bleiben unbefristet. Ist das gewollt?

Die technischen Voraussetzungen - lokale Verarbeitung, wo möglich; Widerspruch
per Befehl; keine Plattform-Anbindung - sind vorhanden. Die Entscheidungen
darüber sind es nicht, und sie lassen sich nicht in Software auslagern.

## Erkennbarkeit der KI

Antworten, die ein Sprachmodell erzeugt hat, werden im Chat sichtbar
gekennzeichnet. Das ist die Voreinstellung; der Betreiber kann sie abschalten,
weil die zugrunde liegende Pflicht regional gilt und nicht überall besteht. Wo
sie besteht, bleibt sie seine Pflicht — die Software nimmt sie ihm nicht ab, sie
erfüllt sie nur, solange er sie eingeschaltet lässt.

Drei Entwurfsentscheidungen dahinter sind erwähnenswert:

**Die Kennzeichnung sitzt im Server, nicht im Client.** Die Schnittstelle ist
dokumentiert und wird von selbstgebauten Clients bedient - eine Lösung auf der
Clientseite könnte schlicht vergessen werden. Serverseitig gilt sie für jeden
angeschlossenen Client, auch für den, den es noch nicht gibt.

**Sie steht nicht im Prompt.** Eine Anweisung „kennzeichne dich als KI" hinge
davon ab, dass das Modell sie befolgt - und wäre damit ein Ziel für genau die
Angriffe, gegen die sich der Rest des Systems wehrt. Wer den Agenten dazu
brächte, seine Kennzeichnung wegzulassen, hätte keine Nettigkeit gewonnen,
sondern eine Rechtspflicht ausgehebelt. Sie wird deshalb im Code an die fertige
Antwort gesetzt, außerhalb der Reichweite des Modells. Dasselbe Prinzip wie
überall sonst: [der Server entscheidet, nicht das Modell](technik.md#das-modell-schlägt-vor-der-server-entscheidet).

**Gekennzeichnet wird, was das Modell erzeugt hat - nicht alles, was der Bot
sagt.** Befehlsbestätigungen, Fehlermeldungen und feste Texte bekommen die
Markierung nicht. Ein Zeichen, das auf jeder Ausgabe steht, sagt nichts mehr
darüber aus, wie sie entstanden ist; es würde das Signal nicht vereinheitlichen,
sondern entwerten.

Die Markierung deckt den Chat ab. Für die gesprochene Ausgabe gibt es kein
Gegenstück im Text - dort trägt die Erkennbarkeit der Kontext, in dem der Agent
auftritt, und die Hinweise, die der Betreiber außerhalb des Chats platziert.

## Weiter

- [Technik](technik.md) - wie das Protokoll funktioniert
- [Sicherheit](sicherheit.md) - das Bedrohungsmodell und die Durchsetzung
