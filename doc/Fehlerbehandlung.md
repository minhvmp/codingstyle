# Fehlerbehandlung mit Exceptions 

In einem Java-Programm können zur Laufzeit verschiedene Fehler auftreten:
- logische Fehler (z.B. Programmierfehler)
- Problem im Java-Laufzeitsystem (z.B. Speichermangel)
- Probleme mit der Peripherie (z.B. mit Internet, Datenbank, Dateisystem)
- fehlerhafte Bedienung (z.B. Benutzereingaben)

Zum Melden eines solchen Fehlers benutzen wir i.A. Exceptions. D.h. bei Auftritt eines dieser Fehler wird das 
Programm an der aktuellen Stelle abgebrochen und eine Exception wird geworfen. Dies hat den Vorteil, dass diese
Laufzeitfehler behandelt werden müssen, und somit nicht ignoriert werden können. 

Auf diese Fehler kann anschließend an geeigneter Stelle im Programm reagiert werden. Je nach Fehler kann
- die zum Fehler führende Handlung wiederholt werden (Internet ist wieder verfügbar, Benutzereingabe verbessert)
- der Fehler in einem Dialog angezeigt werden 
- der Fehler ignoriert werden 
- das Programm abgebrochen werden

Wird keine Fehlerbehandlung umgesetzt, wird das Programm mit einem Stacktrace beendet.

## Validieren von Eingabeparametern

Sichere und robuste Software vertraut niemals Eingabewerten. Es gilt der Grundsatz: „all input is evil“. D.h. in 
öffentlichen Methoden und Konstruktoren müssen Parameter immer validiert werden. 
Genügen diese Parameter nicht dem erwarteten Vertrag, muss eine Exception geworfen werfen. I.A. ist dafür die
`IllegalArgumentExeption` geeignet. Ggf. kann davon abgewichen werden, um z.B. mit der `IndexOutOfBoundsException` oder
der `IllegalStateException` eine genauere Fehlerursache aufzuzeigen. Die `NullPointerException` hat einen Sonderstatus,
sie wird i.A. automatisch geworfen bei Zugriff auf `null`. Ein Werfen dieser Exception ist nur nötig, falls ein
Parameter ohne direkte Nutzung in einer Objektvariable gespeichert wird.

## Exception-Typen

Im JDK ist eine Vielzahl von Exception Klassen vordefiniert, diese haben alle die Endung `Exception` im Klassennamen. 
Es macht selten Sinn, eigene weitere Exceptions zu definieren. Wird eine Exception benötigt, ist i.A. im JDK 
immer eine passende dabei.

Java bietet als einzige Programmiersprache zwei verschiedene Exception Kategorien an:
- checked Exceptions: müssen deklariert und gefangen werden
- unchecked Exceptions: können deklariert und gefangen werden

Das Konzept hat sich in der Praxis nicht bewährt (Details gibt es in Artikeln wie [Checked Exceptions are Evil](https://phauer.com/2015/checked-exceptions-are-evil/)
oder [The Trouble with Checked Exceptions](https://www.artima.com/intv/handcuffs.html)), 
daher nutzen wir möglichst immer unchecked Exceptions. Werden Bibliotheken genutzt, die mit checked Exceptions arbeiten,
bietet es sich an, diese an der Aufrufstelle zu fangen und in eine äquivalente unchecked Exception umzuwandeln. 

## Dokumentation von Exceptions

Wenn Methoden oder Konstruktoren eine Exception werfen können, sollte dies im Methodenkopf mit einer `throws` Klausel 
und im JavaDoc mit einem `@throws` Tag dokumentiert werden. Dort sollte auch immer der Grund beschrieben sein. Dies ist 
nicht kaskadierend erforderlich, d.h. eine Methode die mehrere Methoden aufruft, die Exceptions werfen muss diese nicht
mehr aufführen, sondern nur die selbst geworfenen Exceptions. 

## Fehlerursache (Kontext)

Wird eine Exception geworfen, muss die Fehlerursache (d.h. der Kontext) genau lokalisiert werden, und als Text im 
Konstruktor der Exception übergeben werden. D.h. was ist das Problem? Wie konnte das Problem auftreten? 
Welche Parameterwerte sind Ursache? Diese Meldung ist i.A. nur sichtbar für das Entwicklungsteam und kann z.B. auch dafür
passend formuliert werden. Wird darüber hinaus eine andere Exception gefangen und umgewandelt,
so ist diese auch im Konstruktor der neuen Exception zu übergeben. Generell gilt: Der Default-Konstruktor einer 
Exception darf **nie** verwendet werden. 

## Testen von Exceptions

Das korrekte Werfen von Exceptions sollte generell getestet werden, siehe dazu den passenden Abschnitt im Kapitel zum 
[Testen](Testen.md#testen-von-exceptions).

## Best practice 

Exceptions dürfen nur für außergewöhnliche Ereignisse verwendet werden, d.h. die Programmflusssteuerung darf 
niemals über Exceptions durchgeführt werden. Dies lässt sich umso leichter erreichen, wenn es zu jeder Methode **x** eine
zweite Methode **y** gibt, die prüft, ob die Methode **x** mit den gegebenen Eingabeparametern 
eine Exception werfen würde.

Beispiele: Eine `IndexOutOfBoundsException` lässt sich bei `list.get(0)` vermeiden, wenn vorab die Anzahl der Elemente 
geprüft wird (mit `isEmpty()` oder `size()`). Analoge Methodenpaare finden sich z.B. bei `get(object)` und `contains(object)`,
oder `new FileInputStream(file)` und `file.exists()`, oder `iterator.next()` und `iterator.hasNext()`.
 
Exceptions können mit try/catch/finally Blöcken gefangen werden. Dadurch wird der Code recht schnell 
unübersichtlich, da das **Single Responsibility Principle** (siehe [4, S. 138-139]) verletzt wird. 
Sinnvoll ist daher das Aufteilen des Programmstücks in die folgenden Teile:
- im try Block: Aufruf einer Untermethode (keine Fehlerbehandlung)
- im catch Block: Fehlerbehandlung
- im finally Block: ggf. Aufräumen
- in der Untermethode: Implementierung der Anforderungen ohne Rücksicht auf Exceptions

In einem finally Block sollte niemals eine Exception geworfen werden. Außerdem sollte im finally Block niemals die 
Methode mit return beendet werden.

Ebenso sollten Exceptions niemals ignoriert werden. Sollte es erforderlich sein, einen leeren catch Block zu verwenden, so muss
dies mit einem Kommentar versehen werden!
