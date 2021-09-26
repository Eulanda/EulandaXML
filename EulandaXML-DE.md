# EULANDA XML-Schnittstelle

Dieses Dokument ist auch in englischer Sprache verfügbar - bei Fragen wenden Sie sich bitte an: 

```
Christian Niedergesäß
cn@eulanda.de
 
EULANDA Software GmbH
Beuerbacher Weg 20
65510 Hünstetten
Germany
```

Die XML-Schnittstelle von und zur EULANDA Warenwirtschaft erlaubt den Austausch der wesentlichen Daten zwischen zwei EULANDA-ERP-Systemen oder einer EULANDA-ERP und einem beliebigen Shop- oder Auftragserfassungs-System. 

Die Schnittstelle wurde im Jahr 2000 erstmalig als reiner Artikel-Datenaustausch zwischen zwei EULANDA-Systemen eingeführt und seitdem kontinuierlich an weitere Aufgaben Funktionen angepasst. 

Historisch bedingt, sind alle Feldnamen in der XML-Datei in Deutsch. Lediglich einige Erweiterungs-Knoten, die für Altsysteme nicht relevant sind, wurden in Englisch bezeichnet.

Die Erweiterungen wurden so gestaltet, dass alle alten Anwendungen weiterhin mit dem aktuellen Stand zusammenarbeiten können.

Zur Schnittstelle gibt es ein Benutzer bzw. Administrator-Handbuch, jedoch wird dort auf den Software-Entwickler, der eigene Anbindungen realisieren möchte, wenige eingegangen. Diese Dokumentation richtet sich nun ausschließlich an den Software-Entwickler.

Die Schnittstellensoftware kommt bis auf ganz wenige Ausnahmen mit einer reinen SQL-Verbindung zur EULANDA-Datenbank aus, um Daten zu im- oder exportieren. Ausnahmen sind Funktionen wie zum Beispiel das Rendern von PDF-Dokumenten, diese Funktion steht nur dann zur Verfügung, wenn lokal zur Schnittstellensoftware auch eine EULANDA als Scripting-Host zur Verfügung steht.

Eine Kommunikation besteht jedoch immer aus zwei Teilen, einem der mit EULANDA kommuniziert und ein Teil, der mit dem Fremdsystem kommuniziert. Das kann zum Beispiel ein Shop-System wie nopCommerce sein. Es ist aber auch möglich den zweiten Teil durch einen FTP-Server und eine Datenstruktur zu ersetzen.

> Die Kommunikation erfolgt in jedem Fall über eine oder mehrere XML-Dateien, deren Aufbau in diesem Dokument beschrieben wird.

Die Schnittstellen-Module sind so konzipiert, dass sich diese auf einem beliebigen Windows Arbeitsplatz oder Windows Server (ab Windows 10 bzw. ab Windows-Server 2019) installieren lassen. Das Windows-System muss zur Ausführung der Module nicht angemeldet sein.

Über Steuerdateien, die in der Windows-Aufgabenplanung zeitgesteuert aufgerufen werden, lassen sich autonome Import- und Exportsysteme realisieren.

Hierbei kann ein eingehender Auftrag je nach Konfiguration automatisch gebucht werden. Bei vorhandenem Lagerbestand kann ein Lieferschein erstellt, und daraus eine Rechnung erzeugt werden. Einstellbar ist der anschließende Versand per E-Mail einer Rechnung im XRechnung-, ZUGFeERD- oder PDF-Format.

Das Schnittstellenpaket kann einen FTP-Server überwachen und bei eintreffenden Daten tätig werden oder zeitgesteuert Daten auf einem FTP-Server abliefern. Außerdem kann es mit nopCommerce als Fremdsystem direkt über ODATA kommunizieren.

Weiter Kommunikationen sind aufgrund des flexiblen Systems durch Customizing möglich.

# Szenarien der Anbindung

Grundsätzlich gibt zwei Szenarien zum Einsatz einer EULANDA-ERP mit einem Shopsystem. In einem werden die Produkte durch einen dritten gepflegt und in ein Shop-System übermittelt. Hier übernimmt der Datenpfleger in der Regel auch den Versand (Drop-Shipping). Wir werden lediglich Bestellungen und Statusmeldungen an die EULANDA-ERP gesendet. Die Bestellung enthält in dem Fall immer die kompletten Artikel-Stammdaten, die im Auftrag vorkommen. Im zweiten Szenario werden alle Artikel in der Warenwirtschaft verwaltet. Diese sendet den Artikelstamm bei Änderungen an das Shop-System und erhält Bestellungen von diesem. Hier müssen in der Bestellung nicht nochmal die Artikel-Stammdaten enthalten sein. Nachdem die Ware verschickt wurde, sendet die Warenwirtschaft die Tracking-Informationen als Statusmeldungen an das Shop-System.

Die Adresse-Stammdaten werden normalerweise im Auftrag immer als ADRESSELISTE mit angegeben. Lediglich wenn die Warenwirtschaft die Stammdatenpflege der Adressen durchführt und der Shop keine Neu-Kundenregistrierung durchführt, wäre die Auftragsübergabe ohne den Knoten ADRESSELISTE denkbar. Dieses Szenario wird derzeit jedoch nicht genutzt.

Es lassen sich weitere Szenarien abbilden, die im Rahmen des Customizings erstellt werden können.

# Einstellungen

Das Schnittstellen-Paket kann über eine Konfigurationsdatei mit weit über einhundert Einstellungen für die meisten Szenarien konfiguriert werden. Durch die flexible Jobverwaltung lassen sich zukünftige Anbindungen leicht hinzufügen. Die Optionen der Einstellungen und deren Bedeutung kann im Benutzerhandbuch nachgelesen werden.

# Datenaustausch

## Ordnerstruktur

Die Kommunikation erfolgt über XML-Dateien, die über eine Ordner-Struktur ausgetauscht werden. Die Ordner haben folgende Struktur:

```
/inbox
/inbox/finished
/inbox/pending
/inbox/running
/inbox/error

/outbox
/outbox/finished
/outbox/pending
/outbox/running
/outbox/error
```

Der Austausch kann über eine lokales Speichermedium oder einen FTP-Server erfolgen. Alle Ordnernamen sind in **Kleinbuchstaben** zu schreiben, um kompatibel mit Windows und Linux-Servern zu sein.

Im Ordner **pending** liegen die zu verarbeitenden Dateien mit der Endung **.xml**. Beim Hochladen ist die Dateiendung **.temp** zu verwenden. Erst wenn diese vollständig hochgeladen wurde, ist diese in **.xml** umzubenennen.

Der Prozess, der die Datei anschließend verarbeitet, schiebt diese während der Verarbeitung in den Ordner **running**. Wurde die Datei vollständig bearbeitet, ist diese nach **finished** zu verschieben. Bei Verarbeitungsfehlern ist diese in den Ordner **error** zu verschieben und eine Nachricht per E-Mail an eine vereinbarte Adresse zu senden. Normalerweise ist dies eine der Art [**process@domain.de**](mailto:process@domain.de).

Die **root** der Struktur kann eine beliebige sein und wird vom Kunden-System vorgegeben.

Die Bezeichnung **inbox** bzw. **outbox** ist immer von der Warenwirtschaft aus zu sehen. Sendet die Warenwirtschaft Artikelstammdaten an den Shop, so wird die fertige Datei product\*.xml in den **pending**-Ordner der **outbox** abgelegt. Die Anbindungs-Software, zum Beispiel für ein Shop-System, holt diese dort ab und verarbeitet die Daten. Sendet der Shop hingegen Aufträge an die Warenwirtschaft, so sind diese in den **pending**-Ordner der **inbox** abzulegen.

Das Schnittstellenpaket besteht mindestens aus dem Verarbeitungs-Programm. Die Basissoftware, die die XML-Dateien in die Warenwirtschaft im- oder exportiert und einem Programm, dass diese XM-Dateien dann für das Zielsystem aufbereitet und übergibt bzw. von diesem übernehmen kann.

Für eine Anbindung an das Shop-System nopCommerce ist eine solche Anbindung verfügbar. Hier werden die Daten über das ODATA-Verfahren direkt mit dem Shop-System ausgetauscht.

Eine andere Möglichkeit ist es, dass ein externer Prozess die XML-Dateien von einem FTP-Server, der beim Anwender steht, abholt bzw. Daten dort ablegt. Immer in der angegebenen Ordner-Struktur und über das hier beschriebene XML-Format.

## XML-Dateinamen

Die Dateinamen der XML-Dateien müssen einer bestimmten Namensgebung folgen. Der Aufbau ist **objektname-kennung-uid.xml** wobei der Objektname und die **Dateiextension** immer in **Kleinbuchstaben** sind.

> Ein Dateiname darf für eine Übertragung nur 1x verwendet werden. 

Erlaubte Zeichen sind:

```
a-z	(Kleinbuchstaben)
A-Z	(Großbuchstaben)
0-9	(Ziffern)
.	(Punkt)
-	(Bindestrich)
_	(Unterstrich)
```

Ein gültiger Dateiname für eine reine Produkt-Übertragung könnte wie folgt aussehen:

```
product-1294B00B-50C4-402D-BA2E-61140C301099.xml
```

Wird ein Einzelauftrag übermittelt, so kann optional nach dem Objektnamen die Auftragsnummer als Kennung übergeben werden. Ein gültiger Name einer Auftrags-Übertragung mit der Auftragsnummer **4711** könnte wie folgt aussehen:

```
order-4711-1224B00B-50C4-402D-BA2E-61140C301099.xml
```

### Objektname

Es gibt folgende Objektnamen:

- **stock** für Lagerzahlen von Produkten
- **price** für Preisänderungen zu Produkten
- **product** für Produktdatensätze
- **order** für Aufträge
- **status** für Statusmeldungen wie zum Beispiel Tracking

### Kennung

Die Kennung ist optional. Zu besseren Wiederfindens empfiehlt XML-Dateien, die nur einen Datensatz enthalten, diesen in der Kennung sinnvoll zu kennzeichnen. Dies könnte bei einem Einzelauftrag die Auftragsnummer sein.

### UID

Die UID ist im nächsten Abschnitt beschrieben. Sie wird im Dateinamen ohne die geschweiften Klammern verwendet. Lässt sich die UID vom Fremdsystem nicht erzeugen, so kann eine beliebig andere Bezeichnung verwendet werden, solange diese immer eindeutig ist und nur die erlaubten Zeichen verwendet.

# Datenformat in der XML-Datei

## Kodierung

Es sind nur UTF-8 Kodierungen zulässig.

## Float

Kommazahlen enthalten als Komma den Dezimalpunkt. Ein Minuszeichen ist zulässig, eine Tausender-Trennung ist unzulässig. Ein gültiger Wert ist: **1236.12**

## Integer

Ganze Zahlen je nach breite entweder 8, 16, 32 oder 64-bittig.

## Boolean

Das Feld kann nur zwei Zustände halten, entweder 0 oder 1. Hierbei steht 0 für **false** und 1 für **true**.

## Date

Datumswerte sind umgekehrt mit Bindestrich, vierstelliger Jahreszahl und zweistelligen Monats und Tageszahlen anzugeben. Ein gültiger Wert ist **2021-02-28** für den 28. Februar 2021.

## DateTime

Die Uhrzeit wird über den Buchstaben T mit dem Datumswert verbunden. Die einzelnen Werte Stunde, Minute und Sekunde werden zweistellig angegeben und mit einem Doppelpunkt getrennt. Ein gültiger Wert ist **2021-02-28T14:05:12** für den 28. Februar 2021 vierzehn Uhr fünf Minuten und 12 Sekunden.

## Text

Texte dürfen Umlaute und Sonderzeichen und Zeilensprung (CRLF) enthalten. Werden Sprachen im Langtext angegeben, so sind diese durch einen ISO-Separator zu trennen. Dieser steht in jeweils einer Zeile und in eckigen Klammern. Eine Erklärung zu den Abschnitt-Tags erfolgt im Abschnitt **Sprachen**.

Ein Beispiel für einen Text wäre:

```
[DE]
Das ist ein Text in Deutsch.
[EN]
This is a text in German.
[IT]
Questo è un testo in tedesco.
```

## GUID

Eine GUID (= **G**lobally **U**nique **Id**entifier) oder auch UID (**U**nique **Id**entifier) ist eine 16-Byte-Zahl  und hat somit 128 Bits zur Verfügung. Ein Beispiel einer GUID ist {1884B00B-50C4-402D-BA2E-61140C301099}. Die Zahl wird hexadezimal in Gruppen dargestellt. Innerhalb von XML-Dateien wird diese in geschweifte Klammern gesetzt, in Dateinamen hingegen ohne, Die Zahl ist von der Definition her weltweit eindeutig.

### Erzeugen einer UID in T-SQL

```
DECLARE @uid UNIQUEIDENTIFIER = NEWID();
SELECT CONVERT(CHAR(255), @uid) [UID];
```

### Erzeugen einer UID in Powershell 

```
(Get-WmiObject -Class Win32_ComputerSystemProduct).UUID
```

### Erzeugen einer UID in VbScript

```
Function UID
  Dim TypeLib
  Set TypeLib = CreateObject("Scriptlet.TypeLib")
  UID = Mid(TypeLib.Guid, 2, 36)
End Function
```

## MatchSet

Eine historische Besonderheit sind Textfelder mit dem Attribut MatchSet. Dazu gehören unter anderem die Artikelnummer und Matchcode bei Artikeln und Adressen. Diese Felder müssen in Großbuchstaben angegeben werden. Umlaute sind entsprechend umzusetzen.

```
ä -> AE
ü -> UE
ö -> OE 
ß -> SS
```

Felder mit dem MatchSet-Attribut werden in Fremdsystemen benutzt und oft als Rückwärtsreferenz verwendet. Aus diesem Grunde sind nur wenige Sonderzeichen erlaubt. 

Erlaubte Zeichen sind:

```
A-Z	(Groß-Buchstaben)
0-9	(Ziffern)
.	(Punkt)
-	(Bindestrich)
_	(Unterstrich)
=	(Gleichheitszeichen)
@	(AT-Zeichen wie in E-Mail-Adressen)
```

# Sprachen

In der XML-Datei ist aus historischen Gründen kein gesonderter XML-Knoten oder ein Sprachattribut vorgesehen. Werden verschiedene Sprachen verwendet, so sind diese alle zusammen im selben Text-Feld anzugeben. Als Sprachtrennung dient ein zweistelliger ISO-Buchstaben-Code, der in einer eigenen Zeile in eckigen Klammern anzugeben ist. Also zum Beispiel **[EN]** für einen englischen Abschnitt.

```
[DE]
Das ist ein Text in der Sprache Deutsch
[EN]
This is a text in the language German
[IT]
Questo è un testo in lingua tedesca
```

Beginnt ein Textfeld ohne einen solchen Sprach-Trenner, kann hierdurch ein Standard-Text angeben werden. Dieser wird verwendet, wenn die Übersetzung in der gewünschten Sprache nicht zur Verfügung steht. Im Folgenden ist ein englischer Text der Standardtext. Würde die Sprache Französisch mit **[FR]** angefordert, so würde zumindest ein international lesbarer englischer Text verwendet werden.

```
This is a text in the language English
[DE]
Das ist ein Text in der Sprache Deutsch
[IT]
Questo è un testo in lingua italiano
```

In der EULANDA-Datenbank gibt es Felder, die keine ausreichende Länge haben, um mehrere Sprachen darstellen zu können. Beispielsweise das Feld **Name** eines Merkmals. Dieser ist in der Datenbank ein einzeiliges einfaches Textfeld. Hier bedient sich das System eines Tricks, indem man im normalen Text weitere Unterabschnitte definieren kann.

Handelt es sich bei dem einzeiligen Feld zum Beispiel um das Feld **NAME**, dann kann im Feld BESCHREIBUNG eines Merkmals auch dieser einzeilige Text als Übersetzung angegeben werden. Das Sub-Feld wird mit seinem Namen und einem vorangestellten Doppelpunkt angegeben. Es muss in Großbuchstaben angegeben werden.

Auch diese Übersetzung kann einen Standard haben, in einem solchen Fall wird das ISO-Kürzel weggelassen und der Tag beginnt direkt mit dem Doppelpunkt.

```
This is a text in the language English
[DE]
Das ist ein Text in der Sprache Deutsch
[IT]
Questo è un testo in lingua italiano
[:NAME]
Englisch
[DE:NAME]
Deutsch
[IT:NAME]
Italiano
```

Welche Unterfelder an welcher Stelle erlaubt sind wird an den entsprechenden Stellen des Dokuments angegeben.

# Referenzen und eindeutige Schlüssel

Innerhalb eines Datensatzes, z.B. eines Artikels wird als erstes Feld die ID-des Artikels angegeben. Der Feldinhalt ist identisch mit dem eindeutigen Schlüssel der Tabelle. Beim **Artikel** ist dies die **Artikel-Nummer**. 

```
<ARTIKELLISTE>
   <ARTIKEL>
      <ID.ALIAS>S0221265</ID.ALIAS>
      <ARTNUMMER>S0221265</ARTNUMMER>
      <BARCODE>4023125026003</BARCODE>
      <ARTMATCH />
      <MWSTSATZ>19.00</MWSTSATZ>
      ...
   </ARTIKEL>
</ARTIKELLISTE>
```

Werden in einer Dateien Referenzen zu einem anderen Objekt-Knoten benötigt, so erfolgt dies über ein **ID-Feld**. 

```
<AUFTRAGLISTE>
   <AUFTRAG>
      <DATUM>19-09-2021T21:16:05</DATUM>
      <BESTELLDATUM>2021-09-19T21:16:01</BESTELLDATUM>
      <BESTELLNUMMER>NW-3930</BESTELLNUMMER>
      <OBJEKT>Onlineshop</OBJEKT>
      ...
      <SHOPLTEL />
      <SHOP>
         <SHIPPINGINFO>
            <COST>4.99</COST>
         </SHIPPINGINFO>
      </SHOP>
      <AUFTRAGPOSLISTE>
         <AUFTRAGPOS>
            <ARTIKELID.ALIAS>S0221265</ARTIKELID.ALIAS>
            <MENGE>1</MENGE>
            <VKRAB>33.44</VKRAB>
            <VKVRAB>33.44</VKVRAB>
            <BASIS>30.08</BASIS>
         </AUFTRAGPOS>
      </AUFTRAGPOSLISTE>
   </AUFTRAG>
</AUFTRAGLISTE>
```

Der Knoten AUFTRAGPOS enthält das Feld <ARTIKELID.ALIAS>S0221265</ARTIKELID.ALIAS> und verweist damit auf den Knoten ARTIKEL und dort das Feld <ID.ALIAS>S0221265</ID.ALIAS>.

## Artikel-Referenzen

Die Angabe des Knotens ARTIKELLISTE in einem Auftrag ist nur notwendig, wenn die Stammdaten in der Warenwirtschaft - bzw. dem Zielsystem - nicht vorhanden sind. Dies ist dann der Fall, wenn die EULANDA-Warenwirtschaft nicht die Artikelpflege übernimmt. Der Shop muss dann nicht mehr die kompletten Artikel-Informationen mit jeder Bestellung an die Warenwirtschaft senden.

## Adressen-Referenzen

Das eindeutige Schlüsselfeld im Adressbereich ist das Feld **MATCH**. Es wird dringend empfohlen, die E-Mail-Adresse des Bestellers als **MATCH** zu verwendet. Werden mehrere Auftragserfassungssysteme gleichzeitig genutzt, kann es sinnvoll sein ein Präfix voranzustellen. 

Im folgenden Beispiel wird "SHOPIFY" als Präfix verwendet. Das Feld **MATCH** hat hier den Inhalt `SHOPIFY=FACEMONTY@TOOLHEROS.DE`. Aus verarbeitungstechnischen Gründen **muss** ebenfalls das Feld **ID-ALIAS** mit demselben Inhalt angegeben werden. Das Präfix sollte kurz sein, da das Feld **MATCH** in der Datenbank auf 80 Zeichen limitiert ist.

In der Regel werden in der Auftragsdatei auch die Adressen-Informationen mit den kompletten Stammdaten übergeben. Shop-Systeme erlauben gerade im B2C-Bereich das selbst registrieren als Neukunde. Man kann also nicht davon ausgehen, dass der Adress-Datensatz sich bereits in der Warenwirtschaft befindet.

Insgesamt werden zwei Adressen pro Auftrag unterstützt - die Rechnung- und die Lieferadresse. Die Lieferadresse hat in den Feldnamen des Auftrags immer ein vorangestelltes "L". Das Feld **NAME1** der Rechnungsadresse wäre dann in der Lieferadresse **LNAME1**. 

Da eine Rechnung-Adresse beliebig viele Lieferadressen haben kann und diese sich nicht ohne weiteres als EULANDA-Kontakt abbilden lassen, wird die Lieferadresse immer als Platzhalter in den Stammdaten angelegt und nur referenziert. Die vollständige Lieferadresse steht dann zwingend im Auftrags-Knoten. Sie ist aber nur erforderlich, wenn diese von der Rechnungsadresse abweicht. Schaden tut es jedoch nicht wenn die Lieferadress-Felder immer befüllt werden.

```
<ADRESSELISTE>
   <ADRESSE>
      <ID.ALIAS>SHOPIFY=FACEMONTY@TOOLHEROS.DE</ID.ALIAS>
      <MATCH>SHOPIFY=FACEMONTY@TOOLHEROS.DE</MATCH>
      <NAME1>Harald</NAME1>
      <NAME2>Schmitz</NAME2>
      <NAME3 />
      <STRASSE>Langgasse 15</STRASSE>
      <PLZ>65510</PLZ>
      <ORT>Idstein</ORT>
      <LAND>DE</LAND>
      <EMAIL>facemonty@toolheros.de</EMAIL>
      <TEL />
      <ZIELID.ALIAS>SHOP.PAID</ZIELID.ALIAS>
   </ADRESSE>
   <ADRESSE>
      <ID.ALIAS>SHOPIFY=SHIPPING</ID.ALIAS>
      <MATCH>SHOPIFY=SHIPPING</MATCH>
      <ZIELID.ALIAS>SHOP.PAID</ZIELID.ALIAS>
   </ADRESSE>
</ADRESSELISTE>
```

Wird eine Lieferadresse verwendet muss diese in der Warenwirtschaft eine zu den Stammdaten haben. In der XML-Datei sollte im Knoten ADRESSE immer ein Dummy-Datensatz angegeben werden. Im einfachsten Fall sieht diese dann wie folgt aus:

```
<ADRESSE>
	<ID.ALIAS>SHOPIFY=SHIPPING</ID.ALIAS>
	<MATCH>SHOPIFY=SHIPPING</MATCH>
	<ZIELID.ALIAS>SHOP.PAID</ZIELID.ALIAS>
</ADRESSE>
```

# Reihenfolge und Wiederholungen

Die Reihenfolge der XML-Knoten und Wiederholungen sind hier schematisch dargestellt:

```
<?xml version="1.0" encoding="utf-8" standalone="yes"?>
<EULANDA>
	<METADATA>
		< ... Metainformationen ... >
	</METADATA>
	<MERKMALBAUM>
		<ARTIKEL>
			<PFAD>\Shop (Root des Merkmalbaums in der Zielanwendung)</PFAD>
			<MERKMAL>
				< ... Merkmalfelder ... >
				<MERKMAL>
					< ... geschachteltes Merkmal ... >
				</MERKMAL>
			</MERKMAL>
		</ARTIKEL>
	</MERKMALBAUM>
	<RABATTLISTE/>
	<ARTIKELLISTE>
		<ARTIKEL>
        	< ... Artikelfelder ... >
		</ARTIKEL>
		<ARTIKEL>			
			< ... weiterer Artikel ... >
		</ARTIKEL>		
	</ARTIKELLISTE>
	<ADRESSELISTE>
		<ADRESSE>
			< ... Adressenfelder ... >
		</ADRESSE>
		<ADRESSE>
			< ... weitere Adressen ... >
		</ADRESSE>
	</ADRESSELISTE>
	<AUFTRAGLISTE>
		<AUFTRAG>
			<... Kopfdatenfelder ...>
			<SHOP>
				<SHIPPINGINFO>
				< ... Informationen zum Versand ... >
				</SHIPPINGINFO>
			</SHOP>
			<AUFTRAGPOSLISTE>
				<AUFTRAGPOS>
				<... Positionen zum Auftrag ...>
				</AUFTRAGPOS>
			</AUFTRAGPOSLISTE>
		</AUFTRAG>
		<AUFTRAG>
			<... weitere Aufträge ... >
		</AUFTRAG>		
	</AUFTRAGLISTE>
</EULANDA>
```

Wenn es um eine reine Artikel-Daten geht, brauchen natürlich die Knoten ADRESSLISTE und AUFTRAGLISTE nicht aufgeführt zu werden. In diesem Fall ist der Dateiname product\*.xml zu wählen. Werden Aufträge übermittelt, dann sind alle Knoten zu übergeben und der Dateiname order\*.xml ist zu verwenden. Sollen keine Artikel-Stammdaten angegeben werden, genügt es einen leeren Knoten ARTIKELLISTE zu übergeben. 

Werden Artikeldaten übergeben so ist zumindest ein leerer Knoten MERKMALBAUM mit eingeschlossenem leeren ARTIKEL-Knoten mit anzugeben.

```
<MERKMALBAUM>
		<ARTIKEL/>
</MERKMALBAUM>
```

# Metadaten

Die Metadaten stehen am Dateianfang und beschreiben verschiedene Standards, die in der XML-Datei verwendet werden. Dort kann ebenfalls eingestellt werden, welche Datenbankversion von EULANDA mindestens vorausgesetzt wird und wer die Datei erzeugt hat.

## XML-Darstellung

```
<METADATA>
	<VERSION>1.1</VERSION>
	<GENERATOR>SHOPIFY</GENERATOR>
	<DATEFORMAT>ISO8601</DATEFORMAT>
	<FLOATFORMAT>US</FLOATFORMAT>
	<COUNTRYFORMAT>ISO2</COUNTRYFORMAT>
	<FIELDNAMES>NATIVE</FIELDNAMES>
	<DATE>2021-05-19T21:16:05</DATE>
	<PCNAME/>
	<USERNAME/>
	<DATABASEVERSION>5.63</DATABASEVERSION>
	<APPLICATION>C:\shop\test\Eul.exe</APPLICATION>
	<ALIAS>Mustermann KG</ALIAS>
	<CLIENT>Mustermann</CLIENT>
	<PARTNER>nopCommerce</PARTNER>
</METADATA>
```

## Feldnamen METADATA


| Feldname        | Beschreibung                                                 | Standard | Typ           | Muss |
| --------------- | ------------------------------------------------------------ | -------- | ------------- | ---- |
| ALIAS           | Gibt den Mandantennamen der EULANDA-Datenbank an. Die Angabe ist optional und dient nur dem Fehlertracking. |          | Text max: 60  | nein |
| APPLICATION     | Name des Programms inkl. Pfad, welches diese XML-Datei erzeugt hat. Die Angabe ist optional und dient nur dem Fehlertracking. |          | Text max: 100 | nein |
| CLIENT          | Der Ordnername zur Bildung der internen Struktur der Datenaustauschpfade. Es ist eine verkürzte Variante von ALIAS und wird als Ordnername verwendet. Die Angabe ist optional und dient nur dem Fehlertracking. |          | Text max: 100 | nein |
| COUNTRYFORMAT   | Das Format für Länderkürzel. Aktuell wird nur **ISO2** unterstützt. Hier wäre dann **DE** für Deutschland zu verwenden. | ISO2     | Text          | ja   |
| DATABASEVERSION | Die Versionsnummer der SQL-Datenbank die für die XML-Datei benötigt wird. |          |               |      |
| DATE            | Datum mit Uhrzeit der Erstellung der XML-Datei. Die Angabe erfolgt mit dem Format, welches in DATEFORMAT angegeben ist. Ein solches Datum wäre beispielsweise: 2021-05-19T21:16:05 |          | DateTime      | ja   |
| DATEFORMAT      | Format des Datums. Dies ist aktuell nur **ISO8601**          | ISO8601  | Text          | ja   |
| FIELDNAMES      | Die Feldnamen in der XML-Datei sind aktuell nur NATIVE zulässig. Also mit den Namen, wie diese auch in der SQL-Datenbank vorhanden sind. | NATIVE   | Text          | ja   |
| FLOATFORMAT     | Format bei Fließkommazahlen. Aktuell ist nur **US** zulässig. Hier werden nur das führende **Minuszeichen** sowie ein **Dezimalpunkt** neben den **Ziffern** 0-9 erlaubt. | US       | Text          | ja   |
| GENERATOR       | Ersteller der XML-Datei ist ein Freier Text in Großbuchstaben ohne Umlaute |          | Text          | ja   |
| PARTNER         | Der Name ist eine Kurzbezeichnung für das Zielsystem. Die Angabe ist optional und dient nur dem Fehlertracking. |          | Text max: 100 | nein |
| PCNAME          | Der Feldname kann leer gelassen werden oder es wird der PC-Name des erzeugenden PCs verwendet; dies erleichtert eventuelle Fehlersuchen. |          | Text          | ja   |
| USERNAME        | Der Feldname kann leer übergeben werden, oder einen Ansprechpartner wie er zum Beispiel vom Betriebssystem verwendet wird; dies erleichtert eventuelle Fehlersuchen. |          | Text          | ja   |
| VERSION         | Versionsnummer des Dateiformats. Die aktuelle Version ist **1.1** | 1.1      | Text          | ja   |

# Merkmale

Die Einteilung von Produkten in Gruppen nennen Shop-Systeme **Kataloge** oder **Kategorien**. In EULANDA benutzen wir hierfür Merkmale. Diese sind mit jedem Datenobjekt verwendbar, also auch mit Kunden, Angeboten, Aufträgen usw. 

Merkmale sind in einer Baumstruktur organisiert. Hierbei sind alles Ordner und lediglich die Enden können mit einem Datensatz, wie zum Beispiel einem Artikel, verbunden werden. Jeder Datensatz kann in beliebig vielen Merkmalen referenziert werden.

Werden zum Beispiel Artikeldaten übertragen, so ist stets der komplette Merkmalbaum anzugeben, auch wenn nur Teile eines Artikelstamms übertragen werden und eventuell diese Merkmale dort gar nicht referenziert werden. Der Knotenname für die Merkmale lautet **MERKMALBAUM**. Dieser Knoten ist in jedem Fall anzugeben, wenn Artikeldaten in der XML-Datei übertragen werden, auch wenn Merkmale nicht explizit genutzt werden.

```
<MERKMALBAUM>
		<ARTIKEL/>
</MERKMALBAUM>
```

Zur Synchronisation liefert das initiierende System eine UID zu jedem Merkmal. Änderungen an den Namen oder in den Schachtelungen lassen sich effizient verarbeiten.

## XML-Darstellung

Im folgenden ist ein einfacher Merkmalbaum drei Ästen von der Root weg aufgeführt, Der zweite Ast ist ein Ordner (MERKMALTYP=0), der zwei End-Merkmalen enthält. Der erste und letzte Ast haben keine Unterelemente, und sind damit automatisch End-Merkmale.

```
<MERKMALBAUM>
	<ARTIKEL>
		<PFAD>\Shop</PFAD>
		<MERKMAL>
			<NAME>Bad- Armaturen</NAME>
			<MERKMALTYP>1</MERKMALTYP>
			<BESCHREIBUNG>Bad- Armaturen usw...</BESCHREIBUNG>				
			<UID>{1884B00B-50C4-402D-BA2E-61140C301099}</UID>
		</MERKMAL>
		<MERKMAL>
			<NAME>Hersteller</NAME>
			<MERKMALTYP>0</MERKMALTYP>
			<BESCHREIBUNG>Wir listen alle namenhaften Hersteller, sollte dennoch...</BESCHREIBUNG>			
			<UID>{1884B00B-60C4-402D-BA2E-61140C301099}</UID>
			<MERKMAL>
				<NAME>Grohe</NAME>
				<MERKMALTYP>1</MERKMALTYP>
				<BESCHREIBUNG>Grohe ist seit dem Jahr...</BESCHREIBUNG>			
				<UID>{2884B00B-60C4-402D-BA2E-61140C301099}</UID>
			</MERKMAL> 
			<MERKMAL>
				<NAME>Roco</NAME>
				<MERKMALTYP>1</MERKMALTYP>
				<BESCHREIBUNG>Roco ist seit dem Jahr...</BESCHREIBUNG>			
				<UID>{2984B00B-60C4-402D-BA2E-61140C301099}</UID>
			</MERKMAL> 				
		</MERKMAL>
			<MERKMAL>
				<NAME>Werkzeuge</NAME>
				<MERKMALTYP>1</MERKMALTYP>
				<BESCHREIBUNG>Werkzeuge für Haus und Garten...</BESCHREIBUNG>				
				<UID>{9884B00B-50C4-402D-BA2E-61140C301099}</UID>
		</MERKMAL>
	</ARTIKEL>
</MERKMALBAUM>					
```

Ein vollständig beschriebenes End-Merkmal inkl. Sprachen würde entsprechend wie folgt aussehen:

```
<MERKMALBAUM>
	<ARTIKEL>
		<PFAD>\Shop</PFAD>
		<MERKMAL>
			<NAME>Grohe</NAME>
			<MERKMALTYP>1</MERKMALTYP>
			<BESCHREIBUNG>[DE]
Die Grohe AG ist ein deutscher Hersteller von Armaturen und Sanitärprodukten mit Satzungssitz in Hemer und Verwaltungssitz in Düsseldorf. Seit 2014 gehört Grohe zur japanischen Lixil Group. Grohe ist nicht identisch mit dem Unternehmen Hansgrohe, das ebenfalls ein deutscher Armaturenhersteller ist.
[EN]
Grohe AG is a German manufacturer of fittings and sanitary products with its registered office in Hemer and administrative headquarters in Düsseldorf. Since 2014, Grohe has been part of the Japanese Lixil Group. Grohe is not identical with the company Hansgrohe, which is also a German faucet manufacturer.
[DE:NAME]
Grohe
[EN:NAME]
</BESCHREIBUNG>
			<BILD>Grohe.jpg</BILD>
			<PUBLISHED>1</PUBLISHED>
			<TOP>1</TOP>
			<DISPLAYORDER>0</DISPLAYORDER>
			<SQLBEDINGUNG/>
			<UID>{24640AFC-6E72-4283-89E4-3E37B8E05CB2}</UID>
			<COLOR>536870911</COLOR>
		</MERKMAL>
	</ARTIKEL>
</MERKMALBAUM>
```

## Feldnamen

## MERKMALBAUM.ARTIKEL.MERKMAL
| Feldname     | Beschreibung                                                 | Standard | Typ            | Muss   |
| ------------ | ------------------------------------------------------------ | -------- | -------------- | ------ |
| BESCHREIBUNG | Verschiedene Shop-Systeme bzw. Zielsystem erlauben es, einen Katalog mit einem beschreibenden Text oder Bild zu versehen. Der Text der Beschreibung kann, wie im Abschnitt **Sprachen** beschrieben, auch in Fremdsprachen angegeben werden. Zusätzlich kann er Übersetzungen für das Feld **NAME** enthalten. |          | Text           | nein   |
| BILD         | Jedem Merkmal kann ein Bild zugeordnet werden. Dieses wird inkl. der Beschreibung bei vielen Shop-Systemen in der Katalogübersicht angezeigt. Wenn EULANDA das initiierende System ist, wird hier nur der reine Bildname angegeben, zum Beispiel **meinbild.jpg**. Bilder liegen dann im lokalen DMS, im Normalfall in etwa **\DmsRoot\DMS\Artikel\MERKMALE**. Der Ordner kann im lokalen System oder auf einem FTP-Server liegen. Wird die Artikel-Übergabe jedoch von einem Fremdsystem initiiert und kann nicht auf den lokalen Ordner oder einen unterstützten FTP-Ordner zugreifen, so kann die Bildangabe als URL angegeben werden. Das Bild wird dann von der Schnittstelle automatisch heruntergeladen und in das lokale DMS bzw. die FTP-Struktur kopiert. Ein Beispiel für die Angabe einer URL wäre dann: "**https://www.meinserver.de/subfolder/meinbild.jpg"**. |          | FILE / URL     | nein   |
| COLOR        | Enthält einen RGB-Wert als Dezimalzahl. Hierdurch kann die Farbe eines Merkmals gesetzt werden. Dieses Feld wird innerhalb von EULANDA ausgewertet um Merkmale gesondert hervorzuheben. Es spricht jedoch nichts dagegen, dass ein Zielsystem diesen Wert auswertet. Für die Übergabe an das Shop-System nopCommerce gibt es hier keine Verwendung, da dies dort keine sinnvolle Abbildung dafür gibt. |          | Integer 32-Bit | nein   |
| DISPLAYORDER | Die Displayorder wird zur Sortierung der Anzeigereihenfolge von Merkmalen benutzt. Wird der Wert nicht angegeben, dann verwenden Shop-Systeme bzw. Zielsysteme oft die Erstellungsreihenfolge, aber auch eine alphabetische Ausgabe wird von einigen benutzt. Über DISPLAYORDER hat man nun Einfluss auf die Reihenfolge der Anzeige, sofern das Zielsystem diese unterstützt bei nopCommerce ist dies der Fall. | 0        | Integer 0-255  | nein   |
| MERKMALTYP   | Gibt an um welchen Merkmaltyp es sich handelt. 0=Ordner, 1=Endmerkmale, 2=SQL-Merkmale |          | Integer 8-Bit  | ja     |
| NAME         | Der Name des Ordners bzw. des End-Elements. Der Name wird normalerweise in der Landessprache angegeben z.B. in Deutsch. Werden Fremdsprachen benötigt, so werden die Übersetzungen im Feld BECHREIBUNG abgelegt. Das Vorgehen hierbei ist im Abschnitt **Sprachen** beschrieben. Das Namensfeld ist gleichzeitig das Standard-Feld, wenn dies vom Zielsystem unterstützt wird. Bei Anbindung von nopCommerce als Shop-System ist dies möglich. Fehlt dort eine notwendige Übersetzung, so wird der Inhalt des Standardfelds verwendet. Aus diesem Grund kann es sinnvoll sein, eben hier nicht in Landessprache den Text anzugeben, sondern zum Beispiel in Englisch. |          | Text           | ja     |
| PFAD         | Ist eine interne Angabe, ab welchem Pfad der Merkmalbaum für den Artikel-Datenaustausch benutzt werden soll. Beim Importieren in EULANDA wird der Baum relativ zu dieser Angabe gespeichert. Der Standardpfad ist "\Shop" und sollte aus Kompatibilität angegeben werden. | \Shop    | Text           | (nein) |
| PUBLISHED    | Hierüber kann angegeben werden, ob das Einzelmerkmal im Zielsystem angezeigt werden soll, oder nicht. Die Angabe wird normal von EULANDA erzeugt. Wird die Übergabe von einem Fremdsystem initiiert ist die Angabe normalerweise nicht notwendig. | 1        | Boolean 0/1    | nein   |
| SQLBEDINGUNG | Die EULANDA Warenwirtschaft unterstützt neben gesetzten Einzelmerkmalen auch **intelligente** Merkmale. Dies sind SQL-Befehle, die mit der Datenbank der Warenwirtschaft interagieren. Hier kann man zum Beispiel ein Merkmal "formulieren", dass alle Artikel auswählt, die in einem bestimmten Zeitraum eine Lagerbewegung hatten. Die Angabe spielt im Austausch mit Fremdsystem eigentlich keine Rolle, aber über die Schnittstelle lassen sich auch zwei EULANDA Warenwirtschaften koppeln, um ein Mehrmandaten- oder Dropshipping-System zu realisieren. |          | Text           | nein   |
| TOP          | Über diese Angabe kann gesteuert werden, ob eine Kategorie besonders präsentiert werden soll. Zum Beispiel auf der Homepage eines Shop-Systems. Die Angabe wird normal von EULANDA erzeugt. Wird die Übergabe von einem Fremdsystem initiiert ist die Angabe normalerweise nicht notwendig. | 0        | Boolean 0/1    | nein   |
| UID          | Die UID ist notwendig, um Strukturen mit einem Fremdsystem zu synchronisieren. Das initiierende System sollte eine UID ablegen. Ist diese nicht vorhanden, dann lässt sich **nur eine Erstanlage realisieren**. Anhand der UID werden Änderungen an den Namensbezeichnern erkannt und die Struktur des Merkmalbaums kann synchronisiert werden. Ein Beispiel einer UID ist **{1884B00B-50C4-402D-BA2E-61140C301099}**, es ist das unter Windows verwendete Standard-Verfahren, um weltweit eindeutige Schlüssel zu erzeugen. |          | GUID           | (ja)   |


# Artikel-Stammdaten

Artikeldaten, die für eine reine Stammdatenanlage verwendet werden, haben den Dateinamen **product\*.xml**. Neben den üblichen Feldern eines Artikelstamms, kann dieser auch shop-spezifische Angaben enthalten. Dies können Meta-Informationen für die Suchmaschinen sein, Bild-URLs, Cross- oder Up-Selling Angaben usw. 

Zusätzlich kann der Artikelstamm die Kataloge (= Merkmale) und deren Zuordnung enthalten. Artikeltexte lassen sich in verschiedenen Fremdsprachen angeben. Für geschlossene Systeme lassen sich auch Kundengruppen mit Preislisten im Stammsatz abbilden.

Sollen Artikel vom Shop an eine EULANDA übergeben werden, so ist die Katalogzuordnung i.d.R. nicht erforderlich. Umgekehrt aber, wenn EULANDA Artikel-Daten an einen Shop sendet, sind diese existenziell für die Darstellung im Shop-System.

## XML-Darstellung

```
<ARTIKELLISTE>
	<ARTIKEL>
		<ID.ALIAS>6000505054</ID.ALIAS>
		<CHANGEDATE>2021-09-22T08:09:34</CHANGEDATE>
		<ARTNUMMER>6000505054</ARTNUMMER>
		<MATCH>6000505054</MATCH>
		<BARCODE>4022009275957</BARCODE>
		<ARTNUMMERHERSTELLER>575210000</ARTNUMMERHERSTELLER>
		<MWSTSATZ>19.00</MWSTSATZ>
		<WAEHRUNG>EUR</WAEHRUNG>
		<GEWICHT>15.00</GEWICHT>
		<SHOPFREIGABEFLG>1</SHOPFREIGABEFLG>
		<AUSLAUFFLG>0</AUSLAUFFLG>
		<NEUFLG>0</NEUFLG>
		<SONDERFLG>0</SONDERFLG>
		<LOESCHFLG>0</LOESCHFLG>
		<VERPACKEH>1.00</VERPACKEH>
		<PREISEH>1.00</PREISEH>
		<EKNETTO>210.00</EKNETTO>
		<VK>280.00</VK>
		<BRUTTOFLG>1</BRUTTOFLG>
		<VKNETTO>235,29</VKNETTO>
		<VKBRUTTO>280.00</VKBRUTTO>
		<VOLUMEN>2.40</VOLUMEN>
		<KURZTEXT1>GEBERIT</KURZTEXT1>
		<KURZTEXT2 />
		<ULTRAKURZTEXT>Geberit Vertriebs GmbH Bereich Keramag</ULTRAKURZTEXT>
		<LANGTEXT>Keramag/Geberit Cassini WC-Sitz mit Absenkautomatik, abnehmbar - weiß (Alpin)</LANGTEXT>
		<INFO>Scharniere: Edelstahl Nur passend für das Keramag Cassini WC 203210000!</INFO>
		<SHOP>
			<ARTICLETYPE>1</ARTICLETYPE>
			<SALESSIZE>750.00</SALESSIZE>
			<SALESUNIT>ml</SALESUNIT>			
			<BASEDIVISOR>1</BASEDIVISOR>
			<BASEUNIT>l</BASEUNIT>			
			<CROSS1>4022009275957</CROSS1>
			<CROSS2>""</CROSS2>
			<CROSS3>""</CROSS3>
			<CROSS4>""</CROSS4>
			<CROSS5>""</CROSS5>
			<CROSS6>""</CROSS6>
			<CROSS7>""</CROSS7>
			<CROSS8>""</CROSS8>
			<IMAGE1>https://geberit.com/mera-packshot-geschlossen-1920x1358-16-9.jpg</IMAGE1>
			<IMAGE10>""</IMAGE10>
			<IMAGE11>""</IMAGE11>
			<IMAGE12>""</IMAGE12>
			<IMAGE13>""</IMAGE13>
			<IMAGE14>""</IMAGE14>
			<IMAGE15>""</IMAGE15>
			<IMAGE2>""</IMAGE2>
			<IMAGE3>""</IMAGE3>
			<IMAGE4>""</IMAGE4>
			<IMAGE5>""</IMAGE5>
			<IMAGE6>""</IMAGE6>
			<IMAGE7>""</IMAGE7>
			<IMAGE8>""</IMAGE8>
			<IMAGE9>""</IMAGE9>
			<METADESCRIPTION>Serie: Cassini | Ausstattungs-Merkmale: Mit Absenkautomatik...</METADESCRIPTION>
			<METAKEYWORDS>Serie Cassini, Gewicht 25, Absenkautomatik, Weiß, Keramag, WC</METAKEYWORDS>
			<METATITLE>Keramag Geberit Cassini WC-Sitz mit Absenkautomatik</METATITLE>
			<SHIPPINGWEIGHT>22.00</SHIPPINGWEIGHT>
			<SUGGESTEDLISTPRICE>320.00</SUGGESTEDLISTPRICE>
			<UP1>""</UP1>
			<UP2>""</UP2>
			<UP3>""</UP3>
			<UP4>""</UP4>
			<UP5>""</UP5>
			<UP6>""</UP6>
			<UP7>""</UP7>
			<UP8>""</UP8>
		</SHOP>
		<LAGER>
			<BESTANDVERFUEGBAR>1000.00</BESTANDVERFUEGBAR>
			<BESTANDVERFUEGBAR1>1000.00</BESTANDVERFUEGBAR1>
			<BESTANDVERFUEGBAR2>1000.00</BESTANDVERFUEGBAR2>
		</LAGER>
		<MERKMALLISTE>
			<MERKMAL>
				<PFAD>\WC und Zubehör\WC - Sitze\Geberit</PFAD>
				<PFAD>\Hersteller\Geberit</PFAD>
			</MERKMAL>
		</MERKMALLISTE>
	</ARTIKEL>
</ARTIKELLISTE>
```

## XML-Datei einer product\*.xml

```
<?xml version="1.0" encoding="utf-8" standalone="yes"?>
<EULANDA>
    <METADATA>
        <VERSION>1.1</VERSION>
        <GENERATOR>NOPCOMMERCE</GENERATOR>
        <DATEFORMAT>ISO8601</DATEFORMAT>
        <FLOATFORMAT>US</FLOATFORMAT>
        <COUNTRYFORMAT>ISO2</COUNTRYFORMAT>
        <FIELDNAMES>NATIVE</FIELDNAMES>
        <DATE>2021-09-22T08:10:00</DATE>
        <PCNAME>ASTRA7276</PCNAME>
        <USERNAME>ADMINISTRATOR</USERNAME>
        <DATABASEVERSION>5.63</DATABASEVERSION>
        <APPLICATION>C:\shop\test\Eul.exe</APPLICATION>
        <UDL>Eulanda_1 Test.udl</UDL>
        <ALIAS>Test</ALIAS>
        <CLIENT>test</CLIENT>
        <PARTNER>nopcommerce</PARTNER>
    </METADATA>
    <MERKMALBAUM>
        <ARTIKEL>
            <PFAD>\Shop</PFAD>
            <MERKMAL>
                <NAME>Bad- Armaturen</NAME>
                <MERKMALTYP>0</MERKMALTYP>
                <BESCHREIBUNG>Bad- Armaturen</BESCHREIBUNG>
                <BILD>Bad Armaturen.jpg</BILD>
                <PUBLISHED>1</PUBLISHED>
                <TOP>1</TOP>
                <DISPLAYORDER>0</DISPLAYORDER>
                <SQLBEDINGUNG />
                <UID>{1884B00B-50C4-402D-BA2E-61140C301099}</UID>
                <COLOR>536870911</COLOR>
                <MERKMAL>
                    <NAME>Grohe</NAME>
                    <MERKMALTYP>0</MERKMALTYP>
                    <BESCHREIBUNG>Grohe</BESCHREIBUNG>
                    <BILD>Grohe.jpg</BILD>
                    <PUBLISHED>1</PUBLISHED>
                    <TOP>1</TOP>
                    <DISPLAYORDER>0</DISPLAYORDER>
                    <SQLBEDINGUNG />
                    <UID>{24640AFC-6E72-4283-89E4-3E37B8E05CB2}</UID>
                    <COLOR>536870911</COLOR>
                    <MERKMAL>
                        <NAME>Eurosmart</NAME>
                        <MERKMALTYP>1</MERKMALTYP>
                        <BESCHREIBUNG>Eurosmart</BESCHREIBUNG>
                        <BILD>Eurosmart.jpg</BILD>
                        <PUBLISHED>1</PUBLISHED>
                        <TOP>1</TOP>
                        <DISPLAYORDER>0</DISPLAYORDER>
                        <SQLBEDINGUNG />
                        <UID>{5ED0468A-18A5-4B2D-B399-2723371A4B5D}</UID>
                        <COLOR>536870911</COLOR>
                    </MERKMAL>
                </MERKMAL>
            </MERKMAL>
            <MERKMAL>
                <NAME>WC und Zubehör</NAME>
                <MERKMALTYP>0</MERKMALTYP>
                <BESCHREIBUNG>WC und Zubehör</BESCHREIBUNG>
                <BILD>WC und Zubehör.jpg</BILD>
                <PUBLISHED>1</PUBLISHED>
                <TOP>1</TOP>
                <DISPLAYORDER>0</DISPLAYORDER>
                <SQLBEDINGUNG />
                <UID>{C42223C3-C91E-4C01-A0C4-B03930B53AAF}</UID>
                <COLOR>536870911</COLOR>
                <MERKMAL>
                    <NAME>WC - Sitze</NAME>
                    <MERKMALTYP>0</MERKMALTYP>
                    <BESCHREIBUNG>WC - Sitze</BESCHREIBUNG>
                    <BILD>WC- Sitze.jpg</BILD>
                    <PUBLISHED>1</PUBLISHED>
                    <TOP>1</TOP>
                    <DISPLAYORDER>0</DISPLAYORDER>
                    <SQLBEDINGUNG />
                    <UID>{664296C7-6D76-44E9-8914-1552CBD7BF6D}</UID>
                    <COLOR>536870911</COLOR>
                    <MERKMAL>
                        <NAME>Geberit</NAME>
                        <MERKMALTYP>1</MERKMALTYP>
                        <BESCHREIBUNG>Geberit</BESCHREIBUNG>
                        <BILD>Geberit.jpg</BILD>
                        <PUBLISHED>1</PUBLISHED>
                        <TOP>1</TOP>
                        <DISPLAYORDER>0</DISPLAYORDER>
                        <SQLBEDINGUNG />
                        <UID>{EC87775E-8F2E-4CAF-B4FC-5BAA79FA9C67}</UID>
                        <COLOR>536870911</COLOR>
                    </MERKMAL>
                    <MERKMAL>
                        <NAME>Laufen</NAME>
                        <MERKMALTYP>1</MERKMALTYP>
                        <BESCHREIBUNG>Laufen</BESCHREIBUNG>
                        <BILD>Laufen.jpg</BILD>
                        <PUBLISHED>1</PUBLISHED>
                        <TOP>1</TOP>
                        <DISPLAYORDER>0</DISPLAYORDER>
                        <SQLBEDINGUNG />
                        <UID>{B714BDC3-660F-4028-8BB4-0FB872FEFF89}</UID>
                        <COLOR>536870911</COLOR>
                    </MERKMAL>
                </MERKMAL>
            </MERKMAL>
            <MERKMAL>
                <NAME>Werkzeuge</NAME>
                <MERKMALTYP>0</MERKMALTYP>
                <BESCHREIBUNG>Werkzeuge</BESCHREIBUNG>
                <BILD>Werkzeuge.jpg</BILD>
                <PUBLISHED>1</PUBLISHED>
                <TOP>1</TOP>
                <DISPLAYORDER>0</DISPLAYORDER>
                <SQLBEDINGUNG />
                <UID>{143E9B77-6F41-440A-BAE1-EC9158E567AA}</UID>
                <COLOR>536870911</COLOR>
                <MERKMAL>
                    <NAME>Knipex</NAME>
                    <MERKMALTYP>0</MERKMALTYP>
                    <BESCHREIBUNG>Knipex</BESCHREIBUNG>
                    <BILD>Knipex.jpg</BILD>
                    <PUBLISHED>1</PUBLISHED>
                    <TOP>1</TOP>
                    <DISPLAYORDER>0</DISPLAYORDER>
                    <SQLBEDINGUNG />
                    <UID>{E1D2B85C-F1AA-4399-B088-602B8E144CFE}</UID>
                    <COLOR>536870911</COLOR>
                    <MERKMAL>
                        <NAME>Sortimente</NAME>
                        <MERKMALTYP>1</MERKMALTYP>
                        <BESCHREIBUNG>Werkzeug Sortimente</BESCHREIBUNG>
                        <BILD>Sortimente.jpg</BILD>
                        <PUBLISHED>1</PUBLISHED>
                        <TOP>1</TOP>
                        <DISPLAYORDER>0</DISPLAYORDER>
                        <SQLBEDINGUNG />
                        <UID>{DDF15636-E626-4DD1-BD4F-9D0F302982A5}</UID>
                        <COLOR>536870911</COLOR>
                    </MERKMAL>
                </MERKMAL>
            </MERKMAL>
        </ARTIKEL>
    </MERKMALBAUM>
    <RABATTLISTE />
    <ARTIKELLISTE>
        <ARTIKEL>
            <ID.ALIAS>6000505054</ID.ALIAS>
            <CHANGEDATE>2021-09-22T08:09:34</CHANGEDATE>
            <ARTNUMMER>6000505054</ARTNUMMER>
            <BARCODE>4022009275957</BARCODE>
            <ARTNUMMERHERSTELLER>575210000</ARTNUMMERHERSTELLER>
            <MWSTSATZ>19.00</MWSTSATZ>
            <WAEHRUNG>EUR</WAEHRUNG>
            <GEWICHT>0.00</GEWICHT>
            <SHOPFREIGABEFLG>1</SHOPFREIGABEFLG>
            <AUSLAUFFLG>0</AUSLAUFFLG>
            <NEUFLG>0</NEUFLG>
            <SONDERFLG>0</SONDERFLG>
            <LOESCHFLG>0</LOESCHFLG>
            <VERPACKEH>1.00</VERPACKEH>
            <PREISEH>1.00</PREISEH>
            <EKNETTO>210.00</EKNETTO>
            <VK>210.00</VK>
            <BRUTTOFLG>1</BRUTTOFLG>
            <VKNETTO>176.47</VKNETTO>
            <VKBRUTTO>210.00</VKBRUTTO>
            <VOLUMEN>0.00</VOLUMEN>
            <KURZTEXT2>GEBERIT</KURZTEXT2>
            <ULTRAKURZTEXT>Geberit Vertriebs GmbH Bereich Keramag</ULTRAKURZTEXT>
            <LANGTEXT>Keramag/Geberit Cassini WC-Sitz mit Absenkautomatik, abnehmbar - weiß (Alpin)</LANGTEXT>
            <INFO>Scharniere: Edelstahl Nur passend für das Keramag Cassini WC 203210000!</INFO>
            <SHOP>
                <ARTICLETYPE>1</ARTICLETYPE>
                <BASEDIVISOR>0.00</BASEDIVISOR>
                <BASEUNIT>""</BASEUNIT>
                <CROSS1>""</CROSS1>
                <CROSS2>""</CROSS2>
                <CROSS3>""</CROSS3>
                <CROSS4>""</CROSS4>
                <CROSS5>""</CROSS5>
                <CROSS6>""</CROSS6>
                <CROSS7>""</CROSS7>
                <CROSS8>""</CROSS8>
                <DIMENSIONDEPTH>0</DIMENSIONDEPTH>
                <DIMENSIONHEIGHT>0</DIMENSIONHEIGHT>
                <IMAGE1>""</IMAGE1>
                <IMAGE10>""</IMAGE10>
                <IMAGE11>""</IMAGE11>
                <IMAGE12>""</IMAGE12>
                <IMAGE13>""</IMAGE13>
                <IMAGE14>""</IMAGE14>
                <IMAGE15>""</IMAGE15>
                <IMAGE2>""</IMAGE2>
                <IMAGE3>""</IMAGE3>
                <IMAGE4>""</IMAGE4>
                <IMAGE5>""</IMAGE5>
                <IMAGE6>""</IMAGE6>
                <IMAGE7>""</IMAGE7>
                <IMAGE8>""</IMAGE8>
                <IMAGE9>""</IMAGE9>
                <INFOURL>""</INFOURL>
                <METADESCRIPTION>""</METADESCRIPTION>
                <METAKEYWORDS>""</METAKEYWORDS>
                <METATITLE>""</METATITLE>
                <SALESSIZE>0.00</SALESSIZE>
                <SALESUNIT>""</SALESUNIT>
                <SHIPPINGFREE>0</SHIPPINGFREE>
                <SHIPPINGWEIGHT>0.00</SHIPPINGWEIGHT>
                <SUGGESTEDLISTPRICE>0.00</SUGGESTEDLISTPRICE>
                <UP1>""</UP1>
                <UP2>""</UP2>
                <UP3>""</UP3>
                <UP4>""</UP4>
                <UP5>""</UP5>
                <UP6>""</UP6>
                <UP7>""</UP7>
                <UP8>""</UP8>
            </SHOP>
            <LAGER>
                <BESTANDVERFUEGBAR>1000.00</BESTANDVERFUEGBAR>
                <BESTANDVERFUEGBAR1>1000.00</BESTANDVERFUEGBAR1>
                <BESTANDVERFUEGBAR2>1000.00</BESTANDVERFUEGBAR2>
            </LAGER>
            <MERKMALLISTE>
                <MERKMAL>
                    <PFAD>\WC und Zubehör\WC - Sitze\Geberit</PFAD>
                </MERKMAL>
            </MERKMALLISTE>
        </ARTIKEL>
    </ARTIKELLISTE>
    <ADRESSELISTE />
    <AUFTRAGLISTE />
</EULANDA>
```

## Feldnamen

## ARTIKELLISTE.ARTIKEL

| Feldname            | Beschreibung                                                 | Standard | Typ                                  | Muss   |
| ------------------- | ------------------------------------------------------------ | -------- | ------------------------------------ | ------ |
| **ID.ALIAS**        | Das Schlüsselfeld **ID.ALIAS** ist der Platzhalter, für die in der Warenwirtschaft verwendete ID. Sie muss in jedem Fall übergeben werden und muss denselben Inhalt haben wie das Feld ARTNUMMER. |          | Text max: 80; SPECIAL-UPCASE; UNIQUE | ja     |
| ARTNUMMER           | Die Artikelnummer ist in Großbuchstaben anzugeben. Die Nummer selbst darf nur Buchstaben A-Z sowie Ziffern von 0-9 enthalten. Umlaute sind in z.B. ü in UE oder ß in SS zu übergeben. Zusätzlich sind Bindestrich, Punkt und der Unterstrich zulässig. Die Artikelnummer muss **eindeutig** sein. Wird von mehrere Datenlieferanten importiert und gibt es Artikelnummern-Überscheidungen, so sollte überlegt werden mit einem Präfix zu werden. Zum Beispiel **Unielektro** mit der Artikelnummer **568658997** könnte dann vom Datenlieferanten dann als **UE.568658997** übermittelt werden. Hierbei wäre es wichtig, dass diese Darstellung dann durchgängig genutzt wird. Also bei Bestellungen, Aufträgen, Preisübergaben usw. |          | Text max: 80; SPECIAL-UPCASE; UNIQUE | ja     |
| ARTNUMMERHERSTELLER | Die Hersteller-Nummer kann Groß- und Kleinbuchstaben enthalten. Das Feld selbst darf nur Buchstaben A-Z, a-z sowie Ziffern von 0-9 enthalten. Zusätzlich sind Bindestrich, Punkt und der Unterstrich zulässig. Das Feld ist in der Warenwirtschaft mehrdeutig. |          | Text max: 80;                        | nein   |
| AUSLAUFFLG          | Zeigt an, ob das Produkt demnächst auslaufen soll. Anhand des Wertes lässt sich in einem Shop-System das Produkt besonders bewerben, sofern die Schnittstelle dies zum Fremdsystem unterstützt. | 0        | Boolean                              | nein   |
| BARCODE             | Das Feld kann als GTIN (ehemals EAN) genutzt werden.  Das Feld selbst darf nur Buchstaben A-Z sowie Ziffern von 0-9 enthalten. Umlaute sind in z.B. ü in UE oder ß in SS zu übergeben. Zusätzlich sind Bindestrich, Punkt und der Unterstrich zulässig. Das Feld ist in der Warenwirtschaft mehrdeutig. |          | Text max: 30;                        | nein   |
| BRUTTOFLG           | Dieser Boolean gibt an, ob der übergebene Preis die MwSt. enthält. Hat BRUTTOFLG den Wert 1, so enthält der VK die MwSt. Der Prozentsatz der MwSt. wird in dem Fall im Feld MWST angegeben. Bei B2C Systemen ist es üblich den VK inkl. MwSt. anzugeben. In diesem Fall bekommt der Endverbraucher eine Rechnung mit Inklusive Preisen, bei der am Ende die gesamte MwSt. herausgerechnet und ausgewiesen wird. Im Gegensatz dazu ist ein VK ohne MwSt. im Bereich B2B üblich. Hier bekommt der Unternehmer eine Netto-Rechnung, bei der am Ende auf die Netto-Gesamtsumme die MwSt. heraufgeschlagen wird. Beide Systeme sind in EULANDA parallel möglich. Durch das BRUTTOFLG wird erreicht, dass Endkundenpreise Cent-genau sind. |          | BOOLEAN                              | ja     |
| CHANGEDATE          | Das Änderungsdatum des Datensatzes.                          |          | DateTime                             | nein   |
| EKNETTO             | Der Einkaufspreis wird ohne MwSt. angegeben. Der EKNETTO ist zweistellig (Cent-genau). |          | FLOAT 18.2                           | nein   |
| GEWICHT             | Das Gewicht ist in der Einheit kg anzugeben.                 | 0        | FLOAT 18.4                           | ja     |
| INFO                | Das Feld Info hat eine beliebige Länge. Er speichert nur ANSI also keine Unicode-Sonderzeichen. Die ANSI-Umsetzung erfolgt in Abhängigkeit der Betriebssystem-Sprache. Der Text kann Fremdsprachen enthalten. Das Feld Info sollte eine ergänzende Artikelbeschreibung enthalten. Diese im Shop normalerweise unter dem Artikelbild. Das Info-Feld benutzt Sprachtags für die Angabe von Fremdsprachen. Im Abschnitt **Sprachen** ist beschrieben, wie diese anzugeben sind. |          | Text                                 | ja     |
| KURZTEXT1           | Der Kurztext1 enthält eine Kurzbeschreibung des Produkts, in Kombination mit dem Kurztext 2. Diese beiden Texte haben heute weniger Bedeutung und sind vor allem beim Datenaustausch nach DATANORM/ELDANORM notwendig. |          | Text max: 100                        | nein   |
| KURZTEXT2           | Der Kurztext2 enthält eine Kurzbeschreibung des Produkts, in Kombination mit dem Kurztext 1. Diese beiden Texte haben heute weniger Bedeutung und sind vor allem beim Datenaustausch nach DATANORM/ELDANORM notwendig. |          | Text max: 100                        | nein   |
| LANGTEXT            | Der Langtext hat eine beliebige Länge. Er speichert nur ANSI also keine Unicode-Sonderzeichen. Die ANSI-Umsetzung erfolgt in Abhängigkeit der Betriebssystem-Sprache. Der Text kann Fremdsprachen enthalten, sowie einen mehrsprachigen Kurztext, wie er in Shop-Systemen genutzt wird, um Produktüberschriften zu erzeugen. Der reine Langtext sollte eine Mittellange Produktbeschreibung sein, die normalerweise im Shop neben dem Artikelbild steht. Innerhalb der Warenwirtschaft wird der Langtext in Rechnungen und Angeboten usw. als Produkttext verwendet. Je nach Szenario kann dort aber auch der Kurztext1/2 oder der Ultrakurztext dafür definiert werden. Aus diesem Grund ist es besser auch Kurztext1/2 und Ultrakurztext mit Daten zu befüllen. Der Langtext benutzt Sprachtags für die Angabe von Fremdsprachen. Im Abschnitt **Sprachen** ist beschrieben, wie diese anzugeben sind. Des Weiteren kann der Langtext die Übersetzungen für einen Kurztext enthalten. Hier ist der Tag **[DE:SHORT]** für den Deutschen Kurztext zu verwenden. |          | Text                                 | ja     |
| LOESCHFLG           | Gibt an, dass das Produkt im Zielsystem gelöscht werden kann. Das Shopsystem muss entsprechend hierauf reagieren und die Schnittstelle zum Shop muss das Feld auch in geeigneter Weise unterstützen. | 0        | Boolean                              | nein   |
| MATCH               | Das Feld ist in Großbuchstaben anzugeben. Das Feld dient dem schnellen Auffinden von Produkten und wird in Übersichtslisten benutzt. Das Feld selbst darf nur Buchstaben A-Z sowie Ziffern von 0-9 enthalten. Umlaute sind in z.B. ü in UE oder ß in SS zu übergeben. Zusätzlich sind Bindestrich, Punkt und der Unterstrich zulässig. Das Feld ist in der Warenwirtschaft mehrdeutig. |          | Text max: 80; UPCASE;                | nein   |
| MWSTSATZ            | Der Mehrwertsteuersatz ist in Prozent angegeben. Der Standardwert in Deutschland ist (Stand 09/2021 = 19.00). |          | Float                                | ja     |
| NEUFLG              | Gibt an, dass es sich um ein neues Produkt handelt. Im Zielsystem wie zum Beispiel einem Shop-System können dann Produkte mit dieser Eigenschaft gesondert gekennzeichnet werden. So eine Kennzeichnung könnte ein Rubberband (= Gummiband) oder ein Stempel auf dem Produktbild sein. Das Shopsystem muss entsprechend hierauf reagieren und die Schnittstelle zum Shop muss das Feld auch in geeigneter Weise unterstützen. | 0        | Boolean                              | nein   |
| PREISEH             | Die Preiseinheit gibt an, ob es sich beim VK, VKNETTO oder VKBRUTTO um einen Einzelpreis, oder einen Zehnerpreis oder einen beliebig anderen handelt. Es ist ein Divisor, welcher erst bei der Erstellung der Rechnung benutzt wird. Hierdurch ergibt sich eine sehr genaue Rundung bei Einzelpreisen, die kleiner als 1 Cent sind. Werden Preise mit Preiseinheit 100 angegeben, hat man zwei Nachkommastellen mehr an Genauigkeit. | 1        | FLOAT 10.2                           | ja     |
| SHOPEXPORTDATUM     | Das letzte Exportdatum des Artikels, wenn EULANDA das initiierende System ist. |          | DateTime                             | nein   |
| SHOPFREIGABEFLG     | Das ShopfreigabeFlg ist ein Feld innerhalb der Warenwirtschaft, das angibt, ob der Artikel publiziert werden darf. Beim Import hat das Feld weniger Relevanz und dient speziell bei EULANDA nach EULANDA Übertragungen dazu, den Artikelstamm in abhängigen Systemen zu synchronisieren. | 0        | Boolean                              | nein   |
| SONDERFLG           | Gibt an, dass es sich um ein Sonderangebot handelt. Im Zielsystem wie zum Beispiel einem Shop-System können dann Produkte mit dieser Eigenschaft gesondert gekennzeichnet werden. So eine Kennzeichnung könnte ein Rubber-Band oder ein Stempel auf dem Produktbild sein. Das Shopsystem muss entsprechend hierauf reagieren und die Schnittstelle zum Shop muss das Feld auch in geeigneter Weise unterstützen. | 0        | Boolean                              | nein   |
| ULTRAKURZTEXT       | Der Ultrakurtext enthält eine sehr kompakte Artikelbeschreibung, die speziell auf dem BON bei POS-Systemen verwendet wird. |          | Text max: 100                        | nein   |
| VERPACKEH           | Die Verpackungseinheit gibt an, wieviel gleiche Teile in einem Produkt enthalten sind. Diese Angabe wird lediglich beim Ausdruck von Angeboten oder Rechnungen verwendet. | 1        | FLOAT 10.2                           | ja     |
| VK                  | Der Verkaufspreis kann entweder mit oder ohne MwSt. angegeben werden. So wie der Preis angegeben wird, wird er in die Datenbank der Warenwirtschaft übernommen. Damit das System intern korrekt rechnen kann ist das Zusatzfeld BRUTTOFLG in jedem Fall erforderlich. Der VK ist zweistellig (Cent-genau). |          | FLOAT 18.2                           | ja     |
| VKNETTO             | Unabhängig vom in der Datenbank gespeicherten VK lässt sich auch ein reiner VK ohne MwSt. ausgeben. Dies ist kein Datenbankfeld, sondern ein berechnetes Feld. Es kann nicht in die Warenwirtschaft importiert werden, sondern dient der einfachen Weiterverarbeitung von Fremdsystemen. Es ist jedoch üblich, dieses Feld generell mit anzugeben. Der VKNETTO ist zweistellig (Cent-genau). |          | FLOAT 18.2                           | (nein) |
| VKBRUTTO            | Unabhängig vom in der Datenbank gespeicherten VK lässt sich auch ein reiner VK mit MwSt. ausgeben. Dies ist kein Datenbankfeld, sondern ein berechnetes Feld. Es kann nicht in die Warenwirtschaft importiert werden, sondern dient der einfachen Weiterverarbeitung von Fremdsystemen. Es ist jedoch üblich, dieses Feld generell mit anzugeben. Der VKBRUTTO ist zweistellig (Cent-genau). |          | FLOAT 18.2                           | (nein) |
| VOLUMEN             | Das Volumen ist in der Einheit Liter anzugeben.              | 0        | FLOAT 18.4                           | ja     |
| WAEHRUNG            | Die Währung enthält standardmäßig EUR für EURO. Um andere Währungen zu verarbeiten, benötigt die EULANDA-Warenwirtschaft ein Zusatzmodul **Multiwährung**. Diese sollten deswegen nur nach Rücksprache von Fremdsystemen anders als **EUR** gesetzt werden. | EUR      | Text max: 3                          | ja     |

## Feldnamen

## ARTIKELLISTE.ARTIKEL.SHOP

Diese Feldnamen sind nur im Knoten SHOP unterhalb des Knoten ARTIKEL zulässig. 

> HINWEIS: Sie erfordern die Installation der Shop-Erweiterung, die Bestandteil der Schnittstelle ist.

| Feldname           | Beschreibung                                                 | Standard | Typ                      | Muss |
| ------------------ | ------------------------------------------------------------ | -------- | ------------------------ | ---- |
| ARTICLETYPE        | Das Feld gibt an um welchen Typ eines Artikels es sich handelt. Es wird nur in Verbindung mit der XML-Schnittstelle genutzt. Typ=0: Der Artikel ist noch nicht spezifiziert. Typ=1: Es handelt sich um einen einfachen Verkaufsartikel. Typ=2: Es handelt sich um einen MASTER-Artikel. Typ=3: Es handelt sich um eine Variante. Typ=4: Es handelt sich um eine Variante mit Einzellistung. | 0        | Integer 8-Bit; 0,1,2,3,4 | nein |
| BASEDIVISOR        | Der BASEDIVISOR gibt an, wie das Produkt in einem Shop zusätzlich zum Produktpreis ausgewiesen werden soll. Laut Preisabgabeverordnung muss in Produktpreisnähe auch eine übliche Bezugseinheit mit umgerechnetem Preis angezeigt werden. Soll das in 250 ml angebotene Shampoo mit einem Literpreis ausgewiesen werden, so ist der BASEDIVISOR auf 1 zu setzen und die BASEUNIT entsprechend auf **l**, soll hingegen die Angabe in 100 ml angegeben werden, dann wäre als BASEDIVISOR 100 anzugeben und als BASEUNIT die **ml**. | 1        | Float 18.4               | nein |
| BASEUNIT           | Die Basiseinheit wird im Feld BASEUNIT angegeben. Die Logik hierzu wurde unter BASEDIVISOR bereits beschrieben. |          | Text max: 8              | nein |
| CROSS1 .. CROSS6   | Crosselling-Produkte zeigen kompatible Produkte oder Zubehöre, die üblicherweise bei einem Shop-System auf der Produkt-Detailseite unten aufgeführt werden. Hier wird eine Artikel-Nummer erwartet, die im System bekannt ist. |          | Text max: 32             | nein |
| DIMENSIONDEPTH     | Produkttiefe in mm                                           |          | Integer                  | nein |
| DIMENSIONHEIGHT    | Produkthöhe in mm                                            |          | Integer                  | nein |
| DIMENSIONWIDTH     | Produktbreite in mm                                          |          | Integer                  | nein |
| IMAGE1 .. IMAGE15  | Diese Felder können jeweils eine Bild-URL zu Produktbildern enthalten. Der Dateiname, der jpg- oder png-Datei, sollte so aufgebaut sein, dass er die Artikelnummer, gefolgt von einem Bindestrich und einer laufenden Bildnummer von 1-15 enthält. Wird ein anderer Dateiname gewählt, so kann das Bild nur zur Daten-Erstübernahme verwendet werden. Durch die Nummerierung wird gewährleistet, dass Bilder im System ergänzt, gelöscht oder ausgetauscht werden können. Für die Artikel-Nummer **4711** wäre dann das erste Bild eine URL wie: **`http://www.eulanda.eu/images/4711-1.jpg`** eine gute Wahl. Bilder sollten alle vom selben Bild-Typ sein. Die Bild-Extension sollte 3-stellig sein, also nicht **jpeg** sondern **jpg**. Die laufenden Nummern sollten keine Lücken in der Nummerierung haben. Die Schnittstelle lädt dann beim Import die Bilder über die URL herunter und speichert diese im lokalen DMS. Die Auflösung sollte wenn möglich 2000x2000 Pixel sein (nicht durch Vergrößerung), die Proportion nach Möglichkeit quadratisch. Bei Bildern mit Hintergrund sollte dieser rein weiß sein. Vorzugsweise sollten Bilder auf transparentem Hintergrund liegen und im png-Format sein. Das Hauptbild sollte die laufende Nummer 1 haben. Technische Zeichnungen sollten die letzten Bilder der Reihenfolge sein. |          | URL max: 64              | nein |
| INFOURL            | Eine URL zu weiterführender Information. Das Feld wird normalerweise nicht verwendet, da es heute bessere Methoden gibt. |          | Text max: 128            | nein |
| INFOURLTEXT        | Der Text zu der INFOURL um daraus einen HTML-Link zu erstellen. Das Feld wird normalerweise nicht verwendet, da es heute bessere Methoden gibt. |          | Text max: 64             | nein |
| METADESCRIPTION    | Die METADESCRIPTION wird als Meta-Informationen in HTML-Seiten benötigt, um suchmaschinenfreundliche Inhalte anzubieten. |          | Text                     | nein |
| METAKEYWORDS       | Die METAKEYWORDS wird als Meta-Informationen in HTML-Seiten benötigt, um suchmaschinenfreundliche Inhalte anzubieten. |          | Text                     | nein |
| METATITLE          | Der METATITLE wird als Meta-Informationen in HTML-Seiten benötigt, um suchmaschinenfreundliche Inhalte anzubieten. |          | Text max: 128            | nein |
| SALESSIZE          | Die Verkaufsmenge des Inhalts wie zum Beispiel eines Shampoos. Wird diese üblicherweise in ml angegeben, so könnte dort dann 250 stehen. | 0        | Float 18.4               | nein |
| SALESUNIT          | Die Verkaufseinheit in Bezug auf die Mengenangabe SALESSIZE des Inhalts. Ein Shampoo wird üblicherweise in ml angegeben. |          | Text max: 8              | nein |
| SHIPPINGFREE       | Frachtfrei                                                   | 0        | Boolean                  | nein |
| SHIPPINGWEIGHT     | Im Gegensatz zum Produktgewicht, was im Feld GEWICHT angegeben wird, kann über SHIPPINGWEIGHT das Gewicht inkl. der Produkt-Verpackung angegeben werden. Die Angabe erfolgt in kg. |          | Float 18.4               | nein |
| SUGGESTEDLISTPRICE | Enthält den empfohlenen Verkaufspreis inkl. MwSt.            |          | Float 18.2               | nein |
| UP1 .. UP6         | Up-Selling-Produkte zeigen höherwertige Produkte, die üblicherweise bei einem Shop-System beim Checkout angeboten werden. Hier wird eine Artikel-Nummer erwartet, die im System bekannt ist. |          | Text                     | nein |

## Feldnamen

## ARTIKELLISTE.ARTIKEL.LAGER

Die Lagerzahlen können mit dem Stammdatensatz zusammen übergeben werden. Zusätzlich gibt es noch eine optimierte XML-Datei stock\*.xml, die dann nur die Lagerzahlen in einem vereinfachten Knoten enthält.

| Feldname           | Beschreibung                                                 | Standard | Typ        | Muss |
| ------------------ | ------------------------------------------------------------ | -------- | ---------- | ---- |
| BESTANDVERFUEGBAR  | Gibt die absolute Lagerverfügbarkeit an. Dies ist das übliche Feld zur Übergabe der Lagerverfügbarkeit. |          | Float 10.2 | nein |
| BESTANDVERFUEGBAR1 | Gibt die Lagerverfügbarkeit aus BESTANDVERFUEGBAR an, jedoch abzüglich Reservierungen durch Verkaufsaufträge. Dieses Feld wird in der Regel nicht übermittelt. |          | Float 10.2 | nein |
| BESTANDVERFUEGBAR2 | Gibt die Lagerverfügbarkeit aus BESTANDVERFUEGBAR1 an, jedoch zuzüglich getätigter Einkaufsbestellungen. Dieses Feld wird in der Regel nicht übermittelt. |          | Float 10.2  | nein |

## Feldnamen

## ARTIKELLISTE.ARTIKEL.MERKMALLISTE.MERKMAL

| Feldname | Beschreibung                                                 | Standard | Typ           | Muss |
| -------- | ------------------------------------------------------------ | -------- | ------------- | ---- |
| PFAD     | Gibt die die absolute Lagerverfügbarkeit an. Die einzelnen Pfadabschnitte werden durch Backslash getrennt. Beispielsweise: **\Hersteller\Geberit**. Dieses Feld kann sich im Knoten MERKMAL beliebig wiederholen. Werden Pfade angegeben, so muss in jedem Fall der Komplette Merkmalbaum mit seiner Struktur mit angegeben werden. |          | Text max: 200 | nein |

# Adress-Stammdaten

Adressen können über eine XML-Datei angelegt werden, es ist aber auch möglich, dass Adressen über eine Auftrags-XML-Datei angelegt werden. Der Aufbau des Adressen-Knotens ist jedoch identisch. Werden lediglich Adressen übertragen so haben diese den Dateinamen address*.xml.

Innerhalb der Warenwirtschaft werden alle Adressen in einer Tabelle gespeichert und später zu Lieferanten, Kunden usw. qualifiziert. In der Adress-Verwaltung werden entsprechend Rechnungs- und Lieferadressen gespeichert. Aufgrund der zu unterschiedlichen Verfahren wie externe System Lieferadressen handhaben, wird lediglich eine Platzhalter-Adresse im Stammdatenbereich gespeichert. Diese dient zur internen Referenzierung.

## XML-Darstellung

Der folgende Abschnitt enthält zwei Adressen. Die erste ist eine vollständige Adresse, die zweite ist eine Dummy-Adresse um einen Link  als Lieferadresse zu unterstützen.

```
<ADRESSELISTE>
	<ADRESSE>
		<ID.ALIAS>SHOPIFY=FACEMONTY@TOOLHEROS.DE</ID.ALIAS>
		<MATCH>SHOPIFY=FACEMONTY@WTOOLHEROS.DE</MATCH>
		<NAME1>Harald</NAME1>
		<NAME2>Schmitz</NAME2>
		<NAME3/>
		<STRASSE>Langgasse 15</STRASSE>
		<PLZ>65510</PLZ>
		<ORT>Idstein</ORT>
		<LAND>DE</LAND>
		<EMAIL>facemonty@toolheros.de</EMAIL>
		<TEL/>
		<ZIELID.ALIAS>SHOP.PAID</ZIELID.ALIAS>
	</ADRESSE>
	<ADRESSE>
		<ID.ALIAS>SHOPIFY=SHIPPING</ID.ALIAS>
		<MATCH>SHOPIFY=SHIPPING</MATCH>
		<ZIELID.ALIAS>SHOP.PAID</ZIELID.ALIAS>
	</ADRESSE>
</ADRESSELISTE>
```

## Feldnamen

## ADRESSELISTE.ADRESSE

| Feldname         | Beschreibung                                                 | Standard | Typ                    | Muss |
| ---------------- | ------------------------------------------------------------ | -------- | ---------------------- | ---- |
| **ID.ALIAS**     | Das Schlüsselfeld **ID.ALIAS** ist der Platzhalter, für die in der Warenwirtschaft verwendete ID. Sie muss in jedem Fall übergeben werden und muss denselben Inhalt haben wie das Feld MATCH. |          | Text max: 80; MatchSet | ja   |
| **ZIELID.ALIAS** | Hier kann der intern von EULANDA benutzte Name der Zahlungsbedingung verwendet werden. Es kann aber hier auch ein neuer unbekannter Name verwendet werden. Der Import würde dann zunächst eine entsprechende Zahlungsbedingung in den Stammdaten anlegen und mit der Adresse referenzieren. Übliche Namen wären SHOP.PAID, SHOP.PREPAID, SHOP.PAYPAL usw. |          | Text max: 100          | ja   |
| EMAIL            | Die E-Mail-Adresse in der üblichen Schreibweise.             |          | Text max: 64           | ja   |
| LAND             | Das Land der Adresse wird mit dem zweistelligen ISO-Kürzel angegeben. Also FR für Frankreich, DE für Deutschland usw. Intern kann die Warenwirtschaft auch mit den Landeskürzel aus dem KFZ-Bereich arbeiten. Da wäre Deutschland dann anstatt DE ein D. In der Schnittstelle darf jedoch nur das zweistellige ISO-Kürzel genutzt werden. Die Umsetzung in das intern verwendete Format erfolgt automatisch. |          | Text max: 6            | ja   |
| MATCH            | Das Feld MATCH ist der Hauptschlüssel zur Adresse und muss eindeutig sein. Eine **Besonderheit** ist, dass mit demselben Inhalt das Feld **ID.ALIAS** gesetzt werden muss. Wird die Adresse in einem Fremdsystem, wie zum Beispiel einem Shop-System erzeugt, so wird dringend empfohlen die E-Mail-Adresse des Bestellers hierfür zu verwenden. Das Feld muss in Großbuchstaben umgewandelt werden. Wird ein Präfix verwendet, um das erzeugende System festzuhalten, so kann dies mit einem Gleichheitszeichen abgetrennt werden. Das Feld Match erlaubt nur Zeichen aus dem MatchSet, welche am Dokumentenanfang beschrieben sind. Speziell Umlaute wie **ä** sind in **AE** umzuwandeln. |          | Text max: 80; MatchSet | ja   |
| NAME1            | Der Namensbereich einer Anschrift besteht aus drei möglichen Namenszeilen. Sie können frei verwendet werden und so, wie diese normal bei der Briefadressierung verwendet werden. Üblicherweise steht in NAME1 der Firmenname. Bei Privatpersonen kann das Feld auch leer gelassen werden. |          | Text max: 40           | ja   |
| NAME2            | Das Feld NAME2 wird üblicherweise für Vorname und Nachname verwendet. Bei Formen ist dies dann der Ansprechpartner. |          | Text max: 40           | ja   |
| NAME3            | Das Feld NAME3 wird üblicherweise für besondere Bezeichnungen verwendet, das kann die Postbox sein, eine Abteilung oder ein ansonsten wichtiger Zusatz. |          | Text max: 40           | nein |
| ORT              | Enthält den Ortsnamen zur Adresse.                           |          | Text max: 40           | ja   |
| PLZ              | Das Feld enthält die Postleitzahl.                           |          | Text max: 15           | ja   |
| STRASSE          | Das Feld enthält die Straße zur Adresse inkl. der Hausnummer. Die Hausnummer kann mit den üblichen Sonderzeichen vom Straßen-Namen abgetrennt werden. Für Italien wäre das beispielsweise ein Komma. |          | Text max: 40           | ja   |
| TEL              | Die Telefonnummer zur Adresse. Bei nationalen Rufnummern kann die Auslandsvorwahl, wie zum Beispiel +49, weggelassen werden. Durchwahlen können mit einem Bindestrich abgetrennt werden. Dieser kann auch die Ortsvorwahl von der reinen Rufnummer abtrennen. |          | Text max: 30           | nein |

# Auftrag mit Artikel-Stammdaten

## XML-Datei einer order\*.xml

Die folgende Datei enthält einen vollständigen Auftrag inkl. der Stammdaten in einer einzigen XML-Datei. Innerhalb der XML-Datei werden die Stammdaten über ein ID-System referenziert.

```
<?xml version="1.0" encoding="utf-8" standalone="yes"?>
<EULANDA>
	<METADATA>
		<VERSION>1.1</VERSION>
		<GENERATOR>SHOPIFY</GENERATOR>
		<DATEFORMAT>ISO8601</DATEFORMAT>
		<FLOATFORMAT>US</FLOATFORMAT>
		<COUNTRYFORMAT>ISO2</COUNTRYFORMAT>
		<FIELDNAMES>NATIVE</FIELDNAMES>
		<DATE>19-09-2021T21:16:05</DATE>
		<PCNAME/>
		<USERNAME/>
		<DATABASEVERSION>5.63</DATABASEVERSION>
	</METADATA>
	<MERKMALBAUM>
		<ARTIKEL/>
	</MERKMALBAUM>
	<RABATTLISTE/>
	<ARTIKELLISTE>
		<ARTIKEL>
			<ID.ALIAS>S0221265</ID.ALIAS>
			<ARTNUMMER>S0221265</ARTNUMMER>
			<BARCODE>4023125026003</BARCODE>
			<ARTMATCH/>
			<MWSTSATZ>19.00</MWSTSATZ>
			<MWSTGR>3</MWSTGR>
			<WAEHRUNG>EUR</WAEHRUNG>
			<GEWICHT>0.16</GEWICHT>
			<ARTMASHOPFREIGABEFLGTCH>1</ARTMASHOPFREIGABEFLGTCH>
			<AUSLAUFFLG>0</AUSLAUFFLG>
			<NEUFLG>0</NEUFLG>
			<SONDERFLG>0</SONDERFLG>
			<LOESCHFLG>0</LOESCHFLG>
			<VERPACKEH>1</VERPACKEH>
			<PREISEH>1.00</PREISEH>
			<VK>33.44</VK>
			<BRUTTOFLG>1</BRUTTOFLG>
			<EKNETTO>25.28</EKNETTO>
			<VKNETTO>28.10</VKNETTO>
			<VKBRUTTO>33.44</VKBRUTTO>
			<VOLUMEN>0.00</VOLUMEN>
			<KURZTEXT1>Schnittstellen-Repeater Fritz! N300 300 Mbps WIFI WPS Weiß</KURZTEXT1>
			<ULTRAKURZTEXT>Schnittstellen-Repeater Fritz! N300 300 Mbps WIFI WPS Weiß</ULTRAKURZTEXT>
			<LANGTEXT>Schnittstellen-Repeater Fritz! N300 300 Mbps WIFI WPS Weiß</LANGTEXT>
			<INFO/>
			<SHOP/>
			<LAGER>
				<BESTANDVERFUEGBAR>858.00</BESTANDVERFUEGBAR>
				<BESTANDVERFUEGBAR1>858.00</BESTANDVERFUEGBAR1>
				<BESTANDVERFUEGBAR2>858.00</BESTANDVERFUEGBAR2>
			</LAGER>
			<MERKMALLISTE/>
		</ARTIKEL>
	</ARTIKELLISTE>
	<ADRESSELISTE>
		<ADRESSE>
			<ID.ALIAS>SHOPIFY=FACEMONTY@TOOLHEROS.DE</ID.ALIAS>
			<MATCH>SHOPIFY=FACEMONTY@WTOOLHEROS.DE</MATCH>
			<NAME1>Harald</NAME1>
			<NAME2>Schmitz</NAME2>
			<NAME3/>
			<STRASSE>Langgasse 15</STRASSE>
			<PLZ>65510</PLZ>
			<ORT>Idstein</ORT>
			<LAND>DE</LAND>
			<EMAIL>facemonty@toolheros.de</EMAIL>
			<TEL/>
			<ZIELID.ALIAS>SHOP.PAID</ZIELID.ALIAS>
		</ADRESSE>
		<ADRESSE>
			<ID.ALIAS>SHOPIFY=SHIPPING</ID.ALIAS>
			<MATCH>SHOPIFY=SHIPPING</MATCH>
			<ZIELID.ALIAS>SHOP.PAID</ZIELID.ALIAS>
		</ADRESSE>
	</ADRESSELISTE>
	<AUFTRAGLISTE>
		<AUFTRAG>
			<DATUM>19-09-2021T21:16:05</DATUM>
			<BESTELLDATUM>2021-09-19T21:16:01</BESTELLDATUM>
			<BESTELLNUMMER>NW-3930</BESTELLNUMMER>
			<OBJEKT>Onlineshop</OBJEKT>
			<BRUTTOFLG>1</BRUTTOFLG>
			<ADRESSEID.ALIAS>SHOPIFY=FACEMONTY@TOOLHEROS.DE</ADRESSEID.ALIAS>
			<NAME1>Harald</NAME1>
			<NAME2>Schmitz</NAME2>
			<NAME3/>
			<STRASSE>Langgasse 15</STRASSE>
			<PLZ>65510</PLZ>
			<ORT>Idstein</ORT>
			<LAND>DE</LAND>
			<SHOPEMAIL>facemonty@toolheros.de</SHOPEMAIL>
			<SHOPTEL/>
			<ZIELID.ALIAS>SHOP.PREPAID</ZIELID.ALIAS>
			<LADRESSEID.ALIAS>SHOPIFY=SHIPPING</LADRESSEID.ALIAS>
			<LNAME1>Harald</LNAME1>
			<LNAME2>Schmitz</LNAME2>
			<LNAME3/>
			<LSTRASSE>Marktplatz 15</LSTRASSE>
			<LPLZ>65510</LPLZ>
			<LORT>Hünstetten</LORT>
			<LLAND>DE</LLAND>
			<SHOPLEMAIL>facemonty@toolheros.de</SHOPLEMAIL>
			<SHOPLTEL/>
			<SHOP>
				<SHIPPINGINFO>
					<COST>4.99</COST>
				</SHIPPINGINFO>
			</SHOP>
			<AUFTRAGPOSLISTE>
				<AUFTRAGPOS>
					<ARTIKELID.ALIAS>S0221265</ARTIKELID.ALIAS>
					<MENGE>1</MENGE>
					<VKRAB>33.44</VKRAB>
					<VKVRAB>33.44</VKVRAB>
					<BASIS>30.08</BASIS>
				</AUFTRAGPOS>
			</AUFTRAGPOSLISTE>
		</AUFTRAG>
	</AUFTRAGLISTE>
</EULANDA>
```

# Auftrag ohne Artikel-Stammdaten

## XML-Datei einer order\*.xml

Der gleiche Auftrag, ohne Artikel-Stammdaten jedoch mit den Adresse-Stammdaten würde entsprechend wie folgt aussehen.

```
<?xml version="1.0" encoding="utf-8" standalone="yes"?>
<EULANDA>
	<METADATA>
		<VERSION>1.1</VERSION>
		<GENERATOR>SHOPIFY</GENERATOR>
		<DATEFORMAT>ISO8601</DATEFORMAT>
		<FLOATFORMAT>US</FLOATFORMAT>
		<COUNTRYFORMAT>ISO2</COUNTRYFORMAT>
		<FIELDNAMES>NATIVE</FIELDNAMES>
		<DATE>19-09-2021T21:16:05</DATE>
		<PCNAME/>
		<USERNAME/>
		<DATABASEVERSION>5.63</DATABASEVERSION>
	</METADATA>
	<MERKMALBAUM>
		<ARTIKEL/>
	</MERKMALBAUM>
	<RABATTLISTE/>
	<ARTIKELLISTE>
		<ARTIKEL/>			
	</ARTIKELLISTE>
	<ADRESSELISTE>
		<ADRESSE>
			<ID.ALIAS>SHOPIFY=FACEMONTY@TOOLHEROS.DE</ID.ALIAS>
			<MATCH>SHOPIFY=FACEMONTY@WTOOLHEROS.DE</MATCH>
			<NAME1>Harald</NAME1>
			<NAME2>Schmitz</NAME2>
			<NAME3/>
			<STRASSE>Langgasse 15</STRASSE>
			<PLZ>65510</PLZ>
			<ORT>Idstein</ORT>
			<LAND>DE</LAND>
			<EMAIL>facemonty@toolheros.de</EMAIL>
			<TEL/>
			<ZIELID.ALIAS>SHOP.PAID</ZIELID.ALIAS>
		</ADRESSE>
		<ADRESSE>
			<ID.ALIAS>SHOPIFY=SHIPPING</ID.ALIAS>
			<MATCH>SHOPIFY=SHIPPING</MATCH>
			<ZIELID.ALIAS>SHOP.PAID</ZIELID.ALIAS>
		</ADRESSE>
	</ADRESSELISTE>
	<AUFTRAGLISTE>
		<AUFTRAG>
			<DATUM>19-09-2021T21:16:05</DATUM>
			<BESTELLDATUM>2021-09-19T21:16:01</BESTELLDATUM>
			<BESTELLNUMMER>NW-3930</BESTELLNUMMER>
			<OBJEKT>Onlineshop</OBJEKT>
			<BRUTTOFLG>1</BRUTTOFLG>
			<ADRESSEID.ALIAS>SHOPIFY=FACEMONTY@TOOLHEROS.DE</ADRESSEID.ALIAS>
			<NAME1>Harald</NAME1>
			<NAME2>Schmitz</NAME2>
			<NAME3/>
			<STRASSE>Langgasse 15</STRASSE>
			<PLZ>65510</PLZ>
			<ORT>Idstein</ORT>
			<LAND>DE</LAND>
			<SHOPEMAIL>facemonty@toolheros.de</SHOPEMAIL>
			<SHOPTEL/>
			<ZIELID.ALIAS>SHOP.PREPAID</ZIELID.ALIAS>
			<LADRESSEID.ALIAS>SHOPIFY=SHIPPING</LADRESSEID.ALIAS>
			<LNAME1>Harald</LNAME1>
			<LNAME2>Schmitz</LNAME2>
			<LNAME3/>
			<LSTRASSE>Marktplatz 15</LSTRASSE>
			<LPLZ>65510</LPLZ>
			<LORT>Hünstetten</LORT>
			<LLAND>DE</LLAND>
			<SHOPLEMAIL>facemonty@toolheros.de</SHOPLEMAIL>
			<SHOPLTEL/>
			<SHOP>
				<SHIPPINGINFO>
					<COST>4.99</COST>
				</SHIPPINGINFO>
			</SHOP>
			<AUFTRAGPOSLISTE>
				<AUFTRAGPOS>
					<ARTIKELID.ALIAS>S0221265</ARTIKELID.ALIAS>
					<MENGE>1</MENGE>
					<VKRAB>33.44</VKRAB>
					<VKVRAB>33.44</VKVRAB>
					<BASIS>30.08</BASIS>
				</AUFTRAGPOS>
			</AUFTRAGPOSLISTE>
		</AUFTRAG>
	</AUFTRAGLISTE>
</EULANDA>
```

Zu beachten ist, dass der Knoten ARTIKELLISTE in jedem Fall mit anzugeben ist, auch wenn keine Artikel übertragen werden.

```
<ARTIKELLISTE>
	<ARTIKEL/>			
</ARTIKELLISTE>
```

## Feldnamen

## AUFTRAGLISTE.AUFTRAG

| Feldname             | Beschreibung                                                 | Standard   | Typ           | Muss   |
| -------------------- | ------------------------------------------------------------ | ---------- | ------------- | ------ |
| **ADRESSEID.ALIAS**  | Enthält den Schlüssel für die Adress-Referenz. Sie muss den selben Inhalt wie das dazugehörende Feld **ID.ALIAS** des Knoten **ADRESSE** haben. |            | Text          | ja     |
| **LADRESSEID.ALIAS** | Enthält den Schlüssel für die Lieferadressen-Referenz. Sie muss den selben Inhalt wie das dazugehörende Feld **ID.ALIAS** des Knoten **ADRESSE** haben. Die Lieferadresse muss nur angegeben werden, wenn diese von der Rechnungsadresse abweicht. Wird Sie angegeben, so muss die LADRESSEID.ALIAS auf den Dummy-Adresssatz der Lieferadresse im Knoten ADRESSE zeigen - also den selben Inhalt haben wie das dortige Feld ID.ALIAS der Lieferadresse. Da die Lieferadresse im Stamm leer sein muss, müssen bei Abweichender Lieferadresse in jedem Fall die Lieferadressen-Felder LNAME1, LSTRASSE usw. befüllt werden. |            | Text          | (ja)   |
| **ZIELID.ALIAS**     | Enthält das Schlüsselfeld zur Zahlungsbedingung. Hier kann der intern von EULANDA benutzte Name der Zahlungsbedingung verwendet werden. Es kann aber hier auch ein neuer unbekannter Name verwendet werden. Der Import würde dann zunächst eine entsprechende Zahlungsbedingung in den Stammdaten anlegen und mit der Adresse referenzieren. Übliche Namen wären SHOP.PAID, SHOP.PREPAID, SHOP.PAYPAL usw. |            | Text max: 100 | ja     |
| BESTELLDATUM         | Das Datum der Bestellung wird normalerweise in einem Shop-System durch den Anwender vergeben. Dies ist in der Regel dasselbe Datum wie das Feld DATUM. |            | DateTime      | ja     |
| BESTELLNUMMER        | Bestellnummer des Fremdsystems welches zur der Referenzierung dient. Diese wird in der Warenwirtschaft als Kunden-Bestellnummer geführt. |            | Text          | ja     |
| BRUTTOFLG            | Dieses Feld gibt an, ob der Verkaufspreis **VK** ein Preis inklusive MwSt. ist. Bei einem B2C-Auftrag ist dies meist der Fall. In einem B2C-Onlineshop werden in der Regel die Preise inkl. MwSt. angegeben. Der Warenkorb wird addiert und aus der Summe die MwSt. berechnet und angegeben. Es erfolgt nur eine Rundung. Um dies in der Warenwirtschaft abbilden zu können, müssen in so einem Fall diese identischen Preise übertragen werden und dies wird mit dem BRUTTOFLG angegeben. |            | Boolean       | ja     |
| DATUM                | Datum und Uhrzeit der Auftragsanlage im Fremdsystem.         |            | DateTime      | ja     |
| LAND                 | Das Land der Adresse wird mit dem zweistelligen ISO-Kürzel angegeben. Also FR für Frankreich, DE für Deutschland usw. Intern kann die Warenwirtschaft auch mit den Landeskürzel aus dem KFZ-Bereich arbeiten. Da wäre Deutschland dann anstatt DE ein D. In der Schnittstelle darf jedoch nur das zweistellige ISO-Kürzel genutzt werden. Die Umsetzung in das intern verwendete Format erfolgt automatisch. |            | Text max: 6   | ja     |
| LLAND                | Das Land der Lieferadresse wird mit dem zweistelligen ISO-Kürzel angegeben. Also FR für Frankreich, DE für Deutschland usw. Intern kann die Warenwirtschaft auch mit den Landeskürzel aus dem KFZ-Bereich arbeiten. Da wäre Deutschland dann anstatt DE ein D. In der Schnittstelle darf jedoch nur das zweistellige ISO-Kürzel genutzt werden. Die Umsetzung in das intern verwendete Format erfolgt automatisch. |            | Text max: 6   | ja     |
| LNAME1               | Der Namensbereich einer Lieferanschrift besteht aus drei möglichen Namenszeilen. Sie können frei verwendet werden. Üblicherweise steht in LNAME1 der Firmenname. Bei Privatpersonen kann das Feld auch leer gelassen werden. Die Lieferadresse braucht nur angegeben zu werden, wenn diese von der Rechnungsadresse abweicht. Wird diese jedoch angegeben, so muss auch LADRESSEID.ALIAS angegeben werden. |            | Text max: 40  | (nein) |
| LNAME2               | Das Feld LNAME2 der Lieferadresse wird üblicherweise für Vorname und Nachname verwendet. Bei Firmen ist dies dann der Ansprechpartner. |            | Text max: 40  | (nein) |
| LNAME3               | Das Feld LNAME3 der Lieferadresse wird üblicherweise für besondere Bezeichnungen verwendet; das kann die Postbox, eine Abteilung oder ein ansonsten wichtiger Zusatz sein. |            | Text max: 40  | (nein) |
| LSTRASSE             | Das Feld enthält die Straße zur Adresse inkl. der Hausnummer. Die Hausnummer kann mit den üblichen Sonderzeichen vom Straßen-Namen abgetrennt werden. Für Italien wäre das beispielsweise ein Komma. |            | Text max: 40  | ja     |
| LORT                 | Enthält den Ortsnamen zur Lieferadresse.                     |            | Text max: 40  | ja     |
| LPLZ                 | Das Feld enthält die Postleitzahl zur Lieferadresse.         |            | Text max: 15  | ja     |
| NAME1                | Der Namensbereich einer Anschrift besteht aus drei möglichen Namenszeilen. Sie können frei verwendet werden. Üblicherweise steht in NAME1 der Firmenname. Bei Privatpersonen kann das Feld auch leer gelassen werden. |            | Text max: 40  | ja     |
| NAME2                | Das Feld NAME2 wird üblicherweise für Vorname und Nachname verwendet. Bei Firmen ist dies dann der Ansprechpartner. |            | Text max: 40  | ja     |
| NAME3                | Das Feld NAME3 wird üblicherweise für besondere Bezeichnungen verwendet; das kann die Postbox, eine Abteilung oder ein ansonsten wichtiger Zusatz sein. |            | Text max: 40  | nein   |
| OBJEKT               | Eine Objektbezeichnung, dies kann eine Baustelle sein oder ein sonstiger Begriff, den der Kunde wählt. In der Regel wird dort einfach **Onlineshop** eingetragen. Es ist kein Muss-Feld, hilft aber bei der manuellen Suche in den Auftragsdaten. Wir empfehlen es zu verwenden. | Onlineshop | Text          | (nein) |
| ORT                  | Enthält den Ortsnamen zur Adresse.                           |            | Text max: 40  | ja     |
| PLZ                  | Das Feld enthält die Postleitzahl.                           |            | Text max: 15  | ja     |
| SHOPEMAIL            | Enthält die E-Mail-Adresse des Rechnungsempfängers. Diese E-Mail hat Vorrang vor einer eventuellen E-Mail aus den Adress-Stammdaten. Bei einem Versand des Rechnungsdokuments wird diese E-Mail verwendet. Lediglich wenn keine angegeben ist, wird die E-Mail aus dem Adress-Stamm verwendet. |            | Text max: 64  | (ja)   |
| SHOPLEMAIL           | Enthält die E-Mail-Adresse der Lieferadresse.                |            | Text max: 64  | (ja)   |
| SHOPLTEL             | Die Telefonnummer der Lieferanschrift.                       |            | Text max: 30  | nein   |
| SHOPTEL              | Die Telefonnummer des Rechnungsempfängers. Diese Telefonnummer hat Vorrang vor einer eventuellen Telefonnummer aus den Adress-Stammdaten. |            | Text max: 30  | nein   |
| STRASSE              | Das Feld enthält die Straße zur Adresse inkl. der Hausnummer. Die Hausnummer kann mit den üblichen Sonderzeichen vom Straßen-Namen abgetrennt werden. Für Italien wäre das beispielsweise ein Komma. |            | Text max: 40  | ja     |

## Feldname 

## AUFTRAGLISTE.AUFTRAG.SHOP.SHIPPINGINFO

| Feldname | Beschreibung                           | Standard | Typ        | Muss |
| -------- | -------------------------------------- | -------- | ---------- | ---- |
| COST     | Enthält die Versandkosten der Sendung. |          | Float 18.2 | nein |

## Feldname 

## AUFTRAGLISTE.AUFTRAG.AUFTRAGPOSLISTE.AUFTRAGPOS

Diese minimalen Felder der Auftragsposition genügen, wenn der Artikelstamm bekannt ist und an dieser Stelle über die ARTIKELID.ALIAS referenziert wird. Fehlende Felder werden dann automatisch durch die Stammdaten ergänzt.

| Feldname            | Beschreibung                                                 | Standard | Typ           | Muss   |
| ------------------- | ------------------------------------------------------------ | -------- | ------------- | ------ |
| **ARTIKELID.ALIAS** | Enthält den Schlüssel für die Artikel-Referenz. Sie muss den selben Inhalt wie das dazugehörende Feld **ID.ALIAS** des Knoten **ARTIKEL** haben. |          | Text max: 100 | ja     |
| MENGE               | Verkaufsmenge der Position.                                  |          |               | ja     |
| BASIS               | Basispreis der Position, dies ist der Einkaufspreis inkl. Sonderkonditionen die zu diesem Auftrag geführt haben. Oft stimmt dieser mit dem EKNETTO des Artikelstamms überein. Der Basispreis kann mit oder ohne MwSt. angegeben werden. Ist beim Auftrag das BRUTTOFLG auf 1 gesetzt, so sind alle Preise der Position inklusive MwSt. anzugeben, ansonsten ohne MwSt. |          | Float 18.2    | (nein) |
| VKRAB               | Der rabattierte Verkaufspreis dieser Position bezogen auf ein Stück. Der VKRAB kann mit oder ohne MwSt. angegeben werden. Ist beim Auftrag das BRUTTOFLG auf 1 gesetzt, so sind alle Preise der Position inklusive MwSt. anzugeben, ansonsten ohne MwSt. |          | Float 18.2    | ja     |
| VKVRAB              | Der Verkaufspreis der Position ohne Rabatt. Ist beim Auftrag das BRUTTOFLG auf 1 gesetzt, so sind alle Preise der Position inklusive MwSt. anzugeben, ansonsten ohne MwSt. Dieses Feld wird benötigt um den Rabatt in einer Rechnung später ausweisen zu können. |          | Float 18.2    | ja     |

