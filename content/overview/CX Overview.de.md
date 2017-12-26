+++
title = "CX Overview"
tags = [
    "CX",
]
bounty = 20
date = "2017-09-06"
categories = [
    "Overview",
]
+++

<!-- MarkdownTOC autolink="true" bracket="round" depth="2" -->

- [CX Einführung](#cx-einführung)
- [Das Projekt Repository](#das-projekt-repository)
- [Syntax](#syntax)
- [Angebote](#angebote)
    - [Aritätseinschränkungen](#aritätseinschränkungen)
    - [Typeneinschränkungen](#typeneinschränkungen)
    - [Existenzielle Einschränkungen](#existenzielle-einschränkungen)
    - [Bezeichnereinschränkungen](#bezeichnereinschränkungen)
    - [Grenzeinschränkungen](#grenzeinschränkungen)
    - [Benutzerdefinierte Einschränkungen](#benutzerdefinierte-einschränkungen)
- [Striktes Typensystem](#striktes-typensystem)
- [Kompiliert und Interpretiert](#kompiliert-und-interpretiert)
    - [REPL - (Read-Eval-Print Loop)](#read-eval-print-loop)
    - [Befehle zur Metaprogrammierung](#befehle-zur-metaprogrammierung)
    - [Schrittweise Ausführung](#schrittweise-ausführung)
    - [Interaktive Fehlersuche](#interaktive-fehlersuche)
- [Integrierter evolutionärer Algorithmus](#integrierter-evolutionärer-algorithmus)
- [Serialisierung](#serialisierung)

<!-- /MarkdownTOC -->

# CX Einführung

CX ist sowohl als Spezifikation als auch als Programmiersprache so gefertigt worden um einem neuen Programmiermodell zu folgen das auf gewissen Angeboten basiert. Angebote sind in diesem Dokument so definiert dass es Aktionen sind die möglich sind wenn Einschränkungen oder Bedingungen eingehalten worden sind. Diese Angebote erlauben einem Programm genau zu wissen welche Aktionen machbar und erlaubt sind und was nicht erlaubt ist. Zum Beispiel können wir das Programm fragen welche Argumente eine Funktion empfangen kann und das Programm wird uns eine Liste an möglichen Aktionen zurückliefern. Wenn wir uns für eine der Optionen entscheiden wird das Programm die Aktion ausführen. Als Konsequenz zum CX Regelsystem wurde ein genetischer Algorithmus gebaut und wird als eingebaute Funktion mitgeliefert die benutzt werden kann um die Programstruktur zur Laufzeit zu optimieren.

Die CX Spezifikation besagt dass sowohl der Kompiler als auch der Interpreter dem Programmierer zugänglich sein müssen. Der Interpreter kann in der REPL (auf deutsch übersetzt: lese-ausführe-drucke Schleife) verwendet werden, wo der Programmierer interaktiv nach Belieben Elemente zum Programm hinzufügen oder entfernen kann. Sobald das Programm fertig ist kann es auch kompiliert werden um die Performance noch zu erhöhen.

Das Typsystem in CX ist sehr strikt. Das einzige implizite Konvertieren zwischen Typen passiert wenn der Parser selbst entscheidet welcher Typ es ist, ansonsten wenn zb. eine Funktion explizit einen 64-bit Integer will dann muss auch tatsächlich zu diesem Typ konvertiert werden.

Zuletzt kann ein CX Programm komplett in ein Bytearray serialisiert werden und dabei behält es den gesamten Ausführungszustand und seine Struktur. Die serialisierte Version des Programms kann später wieder deserialisiert werden und man kann die Ausführung auf jeglichem Gerät fortsetzen das einen CX Interpreter oder Kompiler hat.

In den folgenden Sektionen werden CX Merkmale tiefer diskutiert die wir oben nur kurz angesprochen haben.

# Das Projekt Repository

Der Quellcode des Projekts kann von Github runtergeladen werden:
[https://github.com/skycoin/cx](https://github.com/skycoin/cx).
Das Repository beinhaltet die Spezifikationsdatei, die Dokumentation, die Beispiele und den Code selbst.

# Syntax

So wie wir eben schon in der Einführung beschrieben haben ist CX sowohl eine Spezifikation als auch eine Programmiersprache. Die CX Spezifikation verlangt keine bestimmte Syntax sondern stattdessen eine Struktur und Prozesse die ein CX Dialekt implementieren muss um als CX zu gelten. Deswegen könnte man zwei CX Dialekte implementieren, einer mit einer Lisp artigen Syntax und einer mit einer C artigen Syntax. Die zugrundeliegende Sprache heißt CX Basis, oder auch "die Basissprache". In diesem Dokument wird eine Implementierung verwendet die die Möglichkeiten der Spezifikation zeigt, allerdings ist diese Implementierung nicht nur als akademisches Werkzeug zu sehen sondern als eine komplette und robuste Sprache die man allgemein verwenden kann.

Das CX das hier in diesem Dokument verwendet wird hat als Ziel dass die Syntax so nah wie möglich an der Syntax von Go liegt.

# Angebote

Ein Programmierer muss eine Unzahl an Entscheidungen treffen während er ein Programm baut, zb. wieviele Parameter eine Funktion empfangen können soll, wieviele Parameter sie zurückgeben soll, welche Anweisungen sie besitzen muss um die gewollte Funktionalität zu gewährleisten und welche Argumente müssen zu den Aufrufen von Funktionen innerhalb einer Funktion bereitgestellt werden und noch mehr. Das Angebotssystem in CX kann nach einer Liste an möglichen Aktionen befragt werden welche auf ein Element funktionieren. In diesem Kontext kann man als Element zum Beispiel Funktionen, Strukturen, Module und Ausdrücke verstehen.

Ohne einen Satz an Regeln und Fakten welche vorgeben was die Logik und der Sinn des Programms ist kann man einige Angebote feststellen welche zumindest garantieren dass das Program semantisch korrekt ist. Das Angebotssystem von CX bietet solche Einschränkungen als erste Filterschicht, wie unten beschrieben wird.

### Aritätseinschränkungen

In CX können Ausdrücke mehrere Werte zurückgeben. Das erzeugt eine Herausforderung für das Angebotssystem weil die Anzahl der Variablen die Werte erhalten muss mit der Anzahl der Rückgabewerte eines Ausdrucks übereinstimmen.

```
out1, out2, ..., outN := op(inp1, inp2, ..., inpM)
```

Wenn das Beispiel oben korrekt sein soll dann muss *op* *N* Argumente zurückliefern. Dieses Problem kann noch herausfordernder werden wenn wir daran denken dass die Definition von *op* vom Angebotssystem oder vom Benutzer selbst verändert werden kann. Sobald sich die Definition von *op* verändert können neue Einschränkungen an jeden Ausdruck angewandt werden die den *op* Operator benutzen, weil die Anzahl der Variablen die zurückgegeben werden nicht mehr mit den empfangenden Variablen zusammenpasst.

Mit der Logik vorher bedeutet das auch gleichzeitig dass wenn sich die empfangenden Variablen ändern, dann stimmt es natürlich nicht mehr mit dem Operator zusammen und die Aktion neue Variablen hinzufügen kann nicht mehr ausgeführt werden.

Aritätseinschränkungen werden auch auf Eingangsargumente in Ausdrücken angewandt, konkreter wenn ein Funktionsaufruf bereits alle seine Eingangsargumente definiert hat dann soll das Angebotssystem nicht noch mehr Eingangsargumente als Aktion anbieten.
Genauso falls ein Ausdruck einen Operator mit zuwenig Argumenten aufrufen will, dann soll das Angebotssystem dem Programmierer anbieten mehr Argumente zum Funktionsaufruf hinzufügen, falls das Angebotssystem gefragt wird.

**Beispiel:**

*Notiz: Stringkonkatenation wurde noch nicht implementiert. Übrigens fügen die print Funktionen immer einen neuen Zeilenumbruch zum Ende des Strings hinzu. In zukünftigen Versionen von CX Implementierungen werden diese Dinge addressiert.*

```
var age i32 = 18
var steps i32 = 23

func advance (direction str, numberSteps i32) () {
    printStr("Advancing:")
    printStr(direction)
    printStr("Number of steps:")
    printI32(numberSteps)
}

func main () () {
    advance("North")
}
```

Im Beispiel oben fehlt dem Aufruf 'advance' in der *main* Funktion ein Argument. Wenn man das Angebotssystem befragt, sollte es zusätzlich zu anderen Dingen eine Aktion anführen die so ähnlich aussieht wie diese:

```
...
(k)       AddArgument advance age
(k+1)     AddArgument advance steps
...
```

wo k für einen beliebigen Index steht. Man sieht also, das Angebotssystem sagt dem Programmierer dass ein Argument hinzugefügt werden kann und die zwei Möglichkeiten für Argumente sind in diesem Fall die globalen Variablen *age* und *steps*.

Man sollte hier dazusagen dass die Angebote immer aufgelistet werden sollten und ihre Anordnung in der Liste sollte über mehrere Anfragen konstant gleich bleiben. Der Grund dafür ist dass der Programmierer in der Lage sein sollte dem System anzugeben welche Angebote angewandt werden soll nachdem er einen Blick darauf geworfen hat.

### Typeneinschränkungen

Üblich ist es in Programmiersprachen ein Typensystem zu haben welches den Programmier davor bewahrt Argumente an Funktionen zu senden die einen unerwarteten Typ haben. Sogar in schwach typisierten Programmiersprachen sollte eine Operation wie `true / "hello world"` einen Fehler verursachen (außer in sogenannten [esoterischen Sprachen](https://de.wikipedia.org/wiki/Esoterische_Programmiersprache) natürlich.
CX folgt einem sehr [strikten Typensystem](#striktes-typensystem) und Argumente die nicht exakt den erwarteten Typ haben sollten nicht  Kandidaten für Aktionen sein die das Angebotssystem anbietet (obwohl es gibt eine Alternative, nämlich dass man die Argumente zuerst im anderen Typen verpackt bevor sie im Angebotssystem gezeigt werden).

Typeinschränkungen müssen auch berücksichtigt werden wenn man einen neuen Wert an eine existierende Variable zuweist. In CX muss eine Variable die mit einem bestimmten Typ deklariert wurde in ihrer gesamten Lebenszeit diesen Typ beibehalten. (außer er wird durch Metaprogrammierungsbefehle entfernt und neu erzeugt). Eine Variable die einen 32 bit Integer beinhaltet sollte also nicht als Kandidat für einen 64 bit float Rückgabewert dienen, als Beispiel.

### Existenzielle Einschränkungen

Diese Art der Einschränkung kann auf erstem Blick trivial erscheinen: Wenn das Element nicht existiert dann soll auch ein Angebot dazu nicht existieren. Trotzdem wird diese Einschränkung zu einer Herausforderung wenn wir eine Situation betrachten wo eine Funktion umbenannt wurde und sie wurde bereits als Operator in Ausdrücken durch das ganze Programm. Wenn sich das Programm in seiner Quellcodeform befindet dann ist das Problem zu einem "suchen & ersetzen" Prozess reduziert worden, aber falls es zur Laufzeit passiert dann wird das Angebotssystem nützlich: Eine Angebot um den Bezeichner zu einem Bezeichner zu ändern der zu diesem Operator verbunden ist.

Sogar wenn das Element nicht umbenannt wurde ist es nicht so einfach zu entscheiden ob ein Element existiert oder nicht. Die Elemente die in dem Angebot benutzt werden müssen im Aufrufstapel des aktuellen Kontexts, des globalen Kontexts und den anderen Modulkontexten gesucht werden.

### Bezeichnereinschränkungen

Neue Elemente hinzuzufügen sind übliche Kandidaten für Aktionen im Angebotssystem. Eine Einschränkung die vorkommt wenn man solch ein Angebot anwenden will ist die Sicherstellung eines einzigartigen Bezeichners für das neue Element um doppelte Definitionen von Bezeichnern zu vermeiden. Das Angebotssystem kann entweder einen einzigartigen Bezeichner im Kontext des Elements generieren oder kann den Programmierer nach einem geeigneten Bezeichner fragen.

### Grenzeinschränkungen

CX bietet eingebaute, systemeigene Funktionen an für das Lesen und Schreiben von Elementen in Arrays.
Beispiel eines Arraylesers und Arrayschreibers:


```
readI32([]i32{0, 10, 20, 30}, 3)
writeF32([]f32{0.0, 10.10, 20.20}, 1, 5.5)
```

Im ersten Ausdruck wird ein Element aus vier 32 bit Integern auf Index 3 gelesen, was das letzte Element zurückgibt. Im zweiten Ausdruck wird das zweite Element des Arrays von drei 32 bit floats zu 5.5 geändert. Sollte jegliches Array entweder per negativem Index oder einem Index der über das Array hinausgeht gelesen werden wird ein "out of boundaries" (=außerhalb den Grenzen) Fehler verursacht werden.

Durch das Gehorchen dieser Typeneinschränkungen wird das Angebotssystem dem Programmierer sagen dass jegliches 32 bit integer Argument als Index für das Array benutzt werden kann. Obwohl solche Programme erfolgreich kompilieren würden, "out of boundaries" Fehler könnten passieren wenn der Programmierer hier nicht aufpasst was tatsächlich angewandt wird.

Das Angebotssystem muss die Angebote nach den folgenden Kriterien filtern: Verwerfen jeglicher negativer 32 bit integer Zahl und verwerfen jeglicher 32 bit integer die die Länge des Arrays übersteigt welches benutzt wird.

### Benutzerdefinierte Einschränkungen
*Notiz: Die benutzerdefinierten Einschränkungen sind noch auf einem experimentellen Status.*

Die einfachen Einschränkungen oben sollten zumindest garantiert dass das Programm keine Laufzeitfehler mehr hat. Diese Einschränkungen sollten genug sein um ein interessantes System zu bauen, zum Beispiel der in CX integrierte [evolutionäre Algorithmus](#integrierter-evolutionärer-algorithmus). Nichtsdestotrotz, in einigen Situationen braucht man ein robusteres System. Für diesen Zweck werden Klauseln, Anfragen und Objekte benutzt um die Umgebung eines Moduls zu beschreiben. Diese Elemente sind durch die Hilfe eines integrierten Prolog Interpreters definiert sowohl als auch durch die CX integrierten Funktionen *setClauses*, *setQuery*, and
*addObject*.

Die generellste Beschreibung für dieses Einschränkungssystem ist dass der Programmierer eine Serie an Prolog Klauseln (Fakten und Regeln) definiert welche mithilfe der definierten Prolog Klausel für jedes hinzugefügte Objekt angefragt werden. Das wird kaum Sinn machen wenn man es das erste Mal liest. Ein Beispiel sollte diese Konzepte und den Prozess klarer machen:

```
setClauses("move(robot, north, X, R) :- X = northWall, R = false.")

setQuery("move(robot, %s, %s, R).")
```

In diesem Beispiel wird eine einzige Regel definiert. Die Regel kann grob interpretiert werden als "wenn der Roboter sich nach Norden bewegt, frag was X ist. Wenn X northWall ist, dann kann ich mich nicht bewegen." Die Anfrage ist einfach ein formatierter String welcher als Anfrage für die *move* Aktion dient und für das *robot* Element das zwei Argumente annimmt: Eine Richtung und ein objekt.

In this example, only one rule is defined. The rule can roughly be
interpreted as "if the robot wants to move north, ask what is X. If X
is northWall, then it can't move." The query is just a format string
that will serve as a query for the *move* action, and for the *robot*
element that will receive two more arguments: a direction, and an
object.

Objekte können definiert werden mit Hilfe einer *addObject* Funktion:

```
addObject("southWall")
addObject("northWall")
```

Das Einschränkungssystem wird das System für jedes Objekt im Modul befragen. In diesem Beispiel wird das System zuerst die Anfrage "move(robot, north, southWall)," ausführen und dann wird das System antworten "nil," was heißt dass es noch keine Regel gibt die diese Situation behandelt und die Standardregel besagt dass man die Einschränkung nicht fallen lassen soll. Die zweite Anfrage wird "move(robot, north, northWall)," sein und das System wird mit "false." antworten. In diesem Fall hat das Angebot den Test nicht bestanden und wird verworfen.

Das Beispiel oben demonstrierte wie diese Regeln ein Angebot mithilfe einer Bedingung ablehnen können. Aber Regeln können auch benutzt werden um Angebote anzunehmen, sogar wenn sie vorher schon von anderen Regeln abgelehnt wurden.

```
setClauses("move(robot, north, X, R) :- X = northWall, R = false.
    move(robot, north, X, R) :- X = northWormhole, R = true.")

setQuery("move(robot, %s, %s, R).")
```

Die hinzugefügte Regel im Code oben sagt dem System dass sie die Roboterbewegung zum Norden akzeptieren soll sofern das Wurmloch da ist. Wenn das Objektarray links ist so wie vorher definiert dann wird das Bewegungsangebot trotzdem verworfen aber wenn `addObject("northWormhole")` evaluiert wird dann wird "northWormhole" hinzugefügt und der Roboter wird mithilfe des Wurmlochs durch die Wand passieren können.

# Striktes Typensystem

So wie schon in der Einführung erwähnt gibt es kein implizites Konvertieren von Typen in CX. Deswegen sind im Kernmodul mehrere Versionen der primitiven Typen definiert. Zum Beispiel existieren vier integrierte Funktionen für Addition: addI32, addI64, addF32, and addF64. 

Der Parser weist Daten die er im Quellcode findet einen Standardtyp zu: Wenn ein Integer gelesen wird, ist der Standardtyp *i32* also ein 32 bit integer; und wenn ein float gelesen wird, dann ist der Standardtyp *f32* oder: 32 bit float. Es gibt keine Zweideutigkeit mit anderen Daten die vom Parser gelesen werden: *true* und *false* sind immer boolean; Eine Serie von Charakteren die mit doppelten Anführungszeichen umrandet sind ist immer ein String; und ein Array muss immer seinen Typ anführen bevor es die Liste der Elemente aufzählt, zum Beispiel: `[]i64{1, 2, 3}`.

Für die Fälle wo der Programmierer explizit einen Wert von einem Typ auf den anderen konvertieren will bietet das Kernmodul eine Reihe an Konvertierungsfunktionen die mit primitiven Typen funktionieren. Zum Beispiel, `byteAToStr` konvertiert ein Bytearray zu einem String und `i32ToF32` konvertiert einen 32 bit Integer zu einem 32 bit Float.

# Kompiliert und Interpretiert

Die CX Spezifikation zwingt CX Dialekte dem Entwickler sowohl einen Interpreter als auch einen Compiler zu geben. Ein interpretiertes Programm ist weit langsamer als sein kompiliertes Gegenstück, wie man erwarten kann aber es erlaubt ein flexibleres Programm. Die Flexibilität kommt von den Metaprogrammierungsfunktionen und den Angeboten die die Programmstruktur zur Laufzeit ändern können.

Ein kompiliertes Programm braucht eine starrere Struktur als ein interpretiertes Programm weil sich viele Optimierungen auf diese Starrheit verlassen können müssen. Als Konsequenz sind das Angebotssystem und jegliche Funktion die über die Programmstruktur operieren in der kompilierten Version limiert.

Wenn Performance die größte Sorge ist sollte der Kompiler benutzt werden und wenn Flexibilität gebraucht wird dann sollte der Programmierer beim Interpreter bleiben. In den folgenden Unterkapiteln werden einige dieser Konzepte einführungsartig erklärt ohne dass es als Tutorial gesehen werden sollte.

### Read-Eval-Print Loop

Der read-eval-print loop (REPL) wird praktisch nie ins deutsche übersetzt, es bedeutet aber soviel wie "lese-ausführe-drucke Schleife", deswegen belasse ich es auch hier im Text auf der englischen Kürzung "REPL".
Er ist ein interaktives Werkzeug welches Programmierer benutzen können um neue Programmelemente einzugeben und ausführen, bzw. "zu evaluieren". Wenn eine REPL Sitzung gestartet wird, folgt diese Ausgabe auf der Konsole:

```
CX REPL
More information about CX is available at https://github.com/skycoin/cx

*
```

Das "*" zeigt dem Programmierer dass die REPL bereit ist Codelinien zu empfangen. Die REPL wird weiter Eingaben vom Benutzer lesen bis der Benutzer ein Semikolon eingibt und einen Zeilenumbruch also einmal Enter drückt.

Wenn kein Programm in die REPL geladen wurde startet CX mit einem leeren Programm. Das kann man sehen wenn man diesen Metaprogrammierungsbefehl `:dProgram true;` als Eingabe wählt:

```
* :dProgram true;
Program

*
```

Die REPL druckt nur das Wort "Program", gefolgt von einer leeren Zeile. Als erster Schritt definiert man ein neues Modul und eine Funktion.

Als erste Schritte sollten ein neues *main* Modul und eine neue *main* Funktion deklariert werden:

```
* package main;
Program
0.- Module: main

* func main () () {};
Program
0.- Module: main
	Functions
		0.- Function: main () ()

*
```

Wie man sehen kann wird die Programmstruktur jedes Mal wenn ein neues Element hinzugefügt wird auf die Konsole ausgegeben.

### Befehle zur Metaprogrammierung

`:dProgram` wurde im Unterkapitel oben verwendet. Ein beliebiger Ausdruck der mit einem Doppelpunkt (:) beginnt ist Teil einer Kategorie von Anweisungen die hier definiert sind als "Metaprogrammierungsbefehle".

Elemente in der REPL zu deklarieren gibt CX die Anweisung diese der Programmstruktur hinzufügen. Aber so wie in vielen anderen Programmiersprachen benutzen sind diese Deklarationen dahingehend limitiert dass sie in der REPL nur hinzugefügt oder ersetzt werden können. Metaprogrammierungsbefehle erlauben dem Programmierer mehr Kontrolle darüber wie die Programmstruktur modifiziert wird.

`:dProgram`, `:dState`, und `:dStack` werden nur für Fehlersuchezwecke benutzt um dem User eine Programmstruktur, den momentan Aufrufstapel und den vollen Aufrufstapel auszugeben. `:step` weist den Interpreter darauf hin vorwärts oder rückwärts in der Programmausführung zu gehen. `:package`, `:func`, und `:struct`, bekannt als *selectors* werden benutzt um den Kontext zu ändern. `:rem` gibt dem Programmierer Zugang zu *removers*, welche selektiv Elemente aus der Programmstruktur entfernen können. `:aff` wird benutzt um das CX Angebotssystem benutzen zu können; die Metaprogrammierungsbefehle werden benutzt um sowohl Angebote auf gewisse Elemente im Programm anzufragen als auch anzuwenden. Zuletzt, `:clauses` werden benutzt um die Klauseln eines Moduls zu definieren die im [benutzerdefinierten Einschränkungssystem](#benutzerdefinierte-einschränkungen) benutzt werden können;
`:object` and `:objects` werden benutzt um Objekte hinzuzufügen oder auszudrucken, jeweils; und die letzten zwei Metaprogrammierungsbefehle sind: `:query`, welcher benutzt wird um die Anfrage des Moduls zu setzen und `:dQuery` welche ein Helfer für die Fehlersuche von benutzerdefinierten Einschränkungen ist.

### Schrittweise Ausführung

Ein Programm das im REPL Modus gestartet wurde kann mit einer Programmstruktur initialisiert werden die in einer Quelldatei definiert wurde. Zum Beispiel:

```
$ ./cx --load examples/looping.cx

```

Das ladet `looping.cx` vom Beispielverzeichnis (die ganze Liste an Beispielen kann man hier finden: 
[project's repository](https://github.com/skycoin/cx)). Obwohl das Programm geladen wurde, wurde es noch nicht ausgeführt. Um es auszuführen muss in der REPL der Metaprogrammierungsbefehl `:step` ausgeführt werden. Um das Programm bis zum Ende durchzuführen muss `:step 0;` benutzt werden. Aber `:step` ist interessant weil es kann Integerzahlen als Argument nutzen. Zum Beispiel:

```
CX REPL
More information about CX is available at https://github.com/skycoin/cx

* :dStack false;

* :step 5;
0

* :step 5;
1

* :step 5;
2

*
```

Das *examples/looping.cx* Programm wird 5 Schritte auf einmal durchlaufen. Wir können sehen dass 5 Schritte notwendig sind um die *while* Bedingung zu reevaluieren, die Laufvariable auszudrucken und 1 der Laufvariable hinzuzufügen.

Genauso können wir "in der Zeit zurücklaufen" wenn wir der REPL diesen Befehl geben: `:step -5`.

```
...

* :step 5;
2

* :step -5;

* :step 5;
2

*
```

Nachdem wir CX gesagt haben nochmal 5 Schritte vorzugehen wird die 2 wieder auf der Konsole ausgegeben. Es ist zu erwähnen dass der Laufvariable nicht einfach neu zugewiesen wird, sondern der Stapel wird zum alten Status zurückgesetzt.

### Interaktive Fehlersuche

Ein CX Programm geht in den REPL Modus sobald ein Fehler gefunden wird. Dieses Verhalten gibt dem Programmierer die Möglichkeit den Fehler zu finden bevor das Programm weiter ausgeführt werden soll.

Im Beispiel unten wird ein Divisions durch 0 Fehler verursacht und die REPL weist den Programmierer darauf hin, der letzte Aufruf im Aufrufstapel wird ausgegeben und die REPL führt die Ausführung weiter.

```
CX REPL
More information about CX is available at https://github.com/skycoin/cx

* package main;

* func main () () {};

* :func main;
main
:func main {...
	* foo := divI32(5, 3);
main
:func main {...
	* bar := divI32(10, 0);
main
:func main {...
	* :step 0;
fn:main ln:0, 	locals:
>> 1
fn:main ln:1, 	locals: foo: 1

Call's State:
foo:		1

divI32() Arguments:
0: 10
1: 0

0: divI32: Division by 0
main
:func main {...
	*
```

Gleichweg, falls dem CX Interpreter ein Programm gegeben wird ohne die REPL auszuführen aber ein Fehler passiert, dann wird die REPL für den Programmierer oder Systemadministrator ausgeführt um den Fehler zu diagnostizieren:

```
$ ./cx examples/program-halt.cx
1

Call's State:
nonAssign_0:		1
nonAssign_1:		1

divI32() Arguments:
0: 5
1: 0

5: divI32: Division by 0
CX REPL
More information about CX is available at https://github.com/skycoin/cx

*
```

# Integrierter evolutionärer Algorithmus

Das Angebotssystem und die Metaprogrammierungsfunktionen in CX erlauben die Flexibilität die Programmstruktur in einer kontrollierten Weise zu verändern. Aber Angebote können auch automatisiert werden indem man eine Funktion hat die den Index eines Angebots auswählt welches angewandt wird. 

`evolve` ist eine integrierte Funktion die durch zufällige Angebote benutzerdefinierte Funktionen erzeugt. Ein stückweiser Prozess wird verwendet um zu testen.

`evolve` folgt den Prinzipien der evolutionären Berechnung. Im Speziellen benutzt evolve eine Technik namens genetischer Programmierung. Genetische Programmierung versucht eine Kombination aus Operatoren und Argumenten zu finden die das Problem löst. Zum Beispiel, man kann `evolve` anweisen eine Kombination an Operatoren zu finden dass wenn man 10 als Argument sendet, 20 rauskommt. Das mag trivial klingen, aber genetische Programmierung und andere evolutionäre Algorithmen können viele ganz komplizierte Probleme lösen.

Im Beispielverzeichnis *examples* vom Repository findet man ein Beispiel (*examples/evolving-a-function.cx*) das den Prozess zu einer [Anpassungsfunktion](https://de.wikipedia.org/wiki/Ausgleichungsrechnung) für eine Kurve beschreibt.

# Serialisierung

Ein Programm in CX kann teils oder vollständig zu einem Byte Array serialisiert werden. Diese Serialisierungsmöglichkeit erlaubt es dem Program ein Speicherabbild zu erzeugen (ähnlich zu: [Speicherabbild](#https://de.wikipedia.org/wiki/Speicherabbild)), wo der exakte Status des Programms welches serialisiert wurde festgehalten wird. Das bedeutet dass das serialisierte Programm wieder deserialisiert werden kann und die Ausführung später fortgesetzt werden kann. Serialisierung kann auch für Sicherungen benutzt werden.

Ein CX Programm kann diese integrierten Möglichkeiten nutzen um einige interessante Szenarien zu bauen. Zum Beispiel kann ein Programm so gebaut werden dass es eine Sicherung von sich selbst macht und an einer Funktion einen [evolutionären Algorithmus](#integrierter-evolutionärer-algorithmus) startet. Wenn der evolutionäre Algorithmus eine Funktion findet die besser funktioniert als die alte Definition, dann kann man diese neue Version des Programms behalten. Sollte der evolutionäre Algorithmus schlecht entscheiden, dann kann das Programm in seiner alten Form vom gesicherten Speicherabbild wiederhergestellt werden. All diese Dinge können automatisiert werden.
