# EULANDA XML Interface

- [EULANDA XML Interface](#eulanda-xml-interface)
- [Scenarios of the connection](#scenarios-of-the-connection)
- [Settings](#settings)
- [Data exchange](#data-exchange)
  * [Folder structure](#folder-structure)
  * [XML file names](#xml-file-names)
    + [Object name](#object-name)
    + [Identifier](#identifier)
    + [UID](#uid)
- [Data format in the XML file](#data-format-in-the-xml-file)
  * [Encoding](#encoding)
  * [Float](#float)
  * [Integer](#integer)
  * [Boolean](#boolean)
  * [Date](#date)
  * [DateTime](#datetime)
  * [Text](#text)
  * [GUID](#guid)
    + [Generating a UID in T-SQL](#generating-a-uid-in-t-sql)
    + [Create a UID in Powershell](#create-a-uid-in-powershell)
    + [Create a UID in VbScript](#create-a-uid-in-vbscript)
  * [MatchSet](#matchset)
- [Languages](#languages)
- [References and unique keys](#references-and-unique-keys)
  * [Article references](#article-references)
  * [Address references](#address-references)
- [Order and repetitions](#order-and-repetitions)
- [Metadata](#metadata)
  * [XML representation](#xml-representation)
  * [Field names METADATA](#field-names-metadata)
- [Merkmale](#merkmale)
  * [XML representation](#xml-representation-1)
  * [Field names](#field-names)
  * [MERKMALBAUM.ARTIKEL.MERKMAL](#merkmalbaumartikelmerkmal)
- [Article master data](#article-master-data)
  * [XML representation](#xml-representation-2)
  * [XML file of a product\*.xml](#xml-file-of-a-product--xml)
  * [Field names](#field-names-1)
  * [ARTIKELLISTE.ARTIKEL](#artikellisteartikel)
  * [Field names](#field-names-2)
  * [ARTIKELLISTE.ARTIKEL.SHOP](#artikellisteartikelshop)
  * [Field names](#field-names-3)
  * [ARTIKELLISTE.ARTIKEL.LAGER](#artikellisteartikellager)
  * [Field names](#field-names-4)
  * [ARTIKELLISTE.ARTIKEL.MERKMALLISTE.MERKMAL](#artikellisteartikelmerkmallistemerkmal)
- [Price changes](#price-changes)
  * [XML representation](#xml-representation-3)
  * [Field names](#field-names-5)
  * [ARTIKELLISTE.ARTIKEL](#artikellisteartikel-1)
- [Address master data](#address-master-data)
  * [XML representation](#xml-representation-4)
  * [Field names](#field-names-6)
  * [ADRESSELISTE.ADRESSE](#adresselisteadresse)
- [Order with article master data](#order-with-article-master-data)
  * [XML file of an order\*.xml](#xml-file-of-an-order--xml)
- [Order without article master data](#order-without-article-master-data)
  * [XML file of an order\*.xml](#xml-file-of-an-order--xml-1)
  * [Field names](#field-names-7)
  * [AUFTRAGLISTE.AUFTRAG](#auftraglisteauftrag)
  * [Field name](#field-name)
  * [AUFTRAGLISTE.AUFTRAG.SHOP.SHIPPINGINFO](#auftraglisteauftragshopshippinginfo)
  * [Field name](#field-name-1)
  * [AUFTRAGLISTE.AUFTRAG.AUFTRAGPOSLISTE.AUFTRAGPOS](#auftraglisteauftragauftragposlisteauftragpos)
- [Status message](#status-message)
  * [XML representation](#xml-representation-5)
  * [Field names](#field-names-8)
  * [AUFTRAGLISTE.AUFTRAG](#auftraglisteauftrag-1)
  * [Field names](#field-names-9)
  * [AUFTRAGLISTE.AUFTRAG.SHOP](#auftraglisteauftragshop)


This document is also available in German - if you have any questions, please contact:

```
Christian Niedergesaess
cn@eulanda.de
 
EULANDA Software GmbH
Beuerbacher Weg 20
65510 Huenstetten
Germany
```

The XML interface from and to EULANDA ERP allows the exchange of essential data between two EULANDA ERP systems or an EULANDA ERP and any store or order entry system. 

The interface was first introduced in 2000 as a pure article data exchange between two EULANDA systems and has since been continuously adapted to further tasks functions. 

Historically, all field names in the XML file are in German. Only some extension nodes, which are not relevant for legacy systems, were named in English.

The extensions have been designed so that all legacy applications can continue to work with the current state.

To the interface there is a user and/or administrator manual, however there the software developer, who would like to realize own connections, few is dealt with. This documentation addresses itself now exclusively to the software developer.

The interface software gets along with very few exceptions with a pure SQL connection to the EULANDA database to import or export data. Exceptions are functions as for example the rendering of PDF documents, this function is available only if locally to the interface software also an EULANDA as Scripting host is available.

A communication consists however always of two parts, one which communicates with EULANDA and a part which communicates with the foreign system. This can be for example a store system like nopCommerce. But it is also possible to replace the second part by a FTP server and a data structure.

> In any case, the communication takes place via one or more XML files, the structure of which is described in this document.

The interface modules are designed in such a way that they can be installed on any Windows workstation or Windows server (from Windows 10 or from Windows Server 2019). The Windows system does not need to be logged in to run the modules.

Autonomous import and export systems can be implemented via control files that are called up time-controlled in Windows task scheduling.

Here, an incoming order can be booked automatically depending on the configuration. If stock is available, a delivery bill can be created and an invoice generated from it. The subsequent dispatch by e-mail of an invoice in XRechnung, ZUGFeERD or PDF format can be set.

The interface package can monitor an FTP server and take action when data arrives or deliver data to an FTP server on a time-controlled basis. It can also communicate with nopCommerce as a third-party system directly via ODATA.

Further communications are possible through customizing due to the flexible system.

# Scenarios of the connection

Basically there are two scenarios for the use of an EULANDA-ERP with a store system. In one, the products are maintained by a third party and transferred to a store system. Here the data maintainer usually also takes care of the shipping (drop shipping). We are only orders and status messages sent to the EULANDA-ERP. In this case the order always contains the complete article master data, which occur in the order. In the second scenario all articles are administered in the merchandise management. This sends the article master with changes to the Shop system and receives orders from this. Here the order does not have to contain the article master data again. After the goods have been shipped, the merchandise management system sends the tracking information as status messages to the store system.

The address master data is normally always included in the order as an ADRESSELISTE. Only if ERP performs the address master data maintenance and the store does not perform a new customer registration, the order transfer would be conceivable without the ADRESSELISTE node. However, this scenario is not currently used.

Other scenarios can be mapped, which can be created as part of customizing.

# Settings

The interface package can be configured via a configuration file with well over one hundred settings for most scenarios. The flexible job management makes it easy to add future connections. The options of the settings and their meaning can be read in the user manual.

# Data exchange

## Folder structure

Communication takes place via XML files, which are exchanged via a folder structure. The folders have the following structure:

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
The exchange can be done via a local storage medium or an FTP server. All folder names are to be written in **lowercase** to be compatible with Windows and Linux servers.

The **pending** folder contains the files to be processed with the extension **.xml**. When uploading, the file extension **.temp** must be used. Only when this has been completely uploaded is it to be renamed to **.xml**.

The process that subsequently processes the file moves it to the **running** folder during processing. If the file has been fully processed, it should be moved to **finished**. If there are processing errors, move it to the **error** folder and send a message by email to an agreed address. Usually this is one of the type [**process@domain.com**](mailto:process@domain.com).

The **root** of the structure can be any and is specified by the customer system.

The designation **inbox** or **outbox** is always to be seen from the merchandise management. If the merchandise management system sends article master data to the store, the finished product\*.xml file is placed in the **pending** folder of the **outbox**. The connection software, for example for a store system, picks it up there and processes the data. If, on the other hand, the store sends orders to the merchandise management system, these must be stored in the **pending** folder of the **inbox**.

The interface package consists at least of the processing program. The basic software that imports or exports the XML files to the merchandise management system and a program that then prepares these XM files for the target system and transfers them or can accept them from it.

Such a connection is available for a connection to the store system nopCommerce. Here, the data is exchanged directly with the store system via the ODATA procedure.

Another possibility is that an external process fetches the XML files from an FTP server located at the user's site or stores data there. Always in the specified folder structure and via the XML format described here.

## XML file names

The filenames of XML files must follow a specific naming convention. The structure is **objectname-identifier-uid.xml** where the object name and the **file extension** are always in **lowercase**. 

> A file name may only be used 1x for a transfer. 

Allowed characters are:

```
a-z (lower case letters)
A-Z (upper case letters)
0-9 (digits)
.	(period)
- (hyphen)
_ (underscore)
```

A valid file name for a product-only transfer might look like this:

```
product-1294B00B-50C4-402D-BA2E-61140C301099.xml
```

If an individual order is transferred, the order number can optionally be transferred as an identifier after the object name. A valid name of an order transmission with the order number **4711** could look like the following:

```
order-4711-1224B00B-50C4-402D-BA2E-61140C301099.xml
```

### Object name

There are the following object names:

- **stock** for stock numbers of products
- **price** for price changes of products
- **product** for product records
- **order** for orders
- **status** for status messages such as tracking

### Identifier

The identifier is optional. For better retrieval, it is recommended that XML files that contain only one data record are identified in the identifier in a meaningful way. In the case of a single order, this could be the order number.

### UID

The UID is described in the next section. It is used in the file name without the curly brackets. If the UID cannot be generated by the external system, any other designation can be used, as long as it is always unique and only uses the permitted characters.

# Data format in the XML file

## Encoding

Only UTF-8 encodings are allowed.

## Float

Comma numbers contain the decimal point as a decimal point. A minus sign is allowed, a thousands separator is not allowed. A valid value is: **1236.12**

## Integer

Integer depending on width either 8, 16, 32 or 64-bit.

## Boolean

The field can only hold two states, either 0 or 1, where 0 stands for **false** and 1 for **true**.

## Date

Date values must be entered in reverse with a hyphen, four-digit year, and two-digit month and day. A valid value is **2021-02-28** for February 28, 2021.

## DateTime

The time is linked to the date value by the letter T. The individual hour, minute, and second values are specified with two digits and separated by a colon. A valid value is **2021-02-28T14:05:12** for February 28, 2021 fourteen o'clock five minutes and 12 seconds.

## Text

Texts may contain umlauts and special characters and interlace (CRLF). If languages are specified in the long text, they must be separated by an ISO separator. This is in each case in a line and in square brackets. An explanation of the section tags is given in the **Languages** section.

An example of a text would be:

```
[DE]
Das ist ein Text in Deutsch.
[EN]
This is a text in German.
[IT]
Questo è un testo in tedesco.
```

## GUID

A GUID (= **G**lobally **U**nique **Id**entifier) or UID (**U**nique **Id**entifier) is a 16-byte number and thus has 128 bits available. An example of a GUID is {1884B00B-50C4-402D-BA2E-61140C301099}. The number is represented in hexadecimal in groups. Within XML files it is enclosed in curly brackets, but in filenames it is enclosed without, The number is globally unique by definition.

### Generating a UID in T-SQL

```
DECLARE @uid UNIQUEIDENTIFIER = NEWID();
SELECT CONVERT(CHAR(255), @uid) [UID];
```

### Create a UID in Powershell 

```
(Get-WmiObject -Class Win32_ComputerSystemProduct).UUID
```

### Create a UID in VbScript

```
Function UID
  Dim TypeLib
  Set TypeLib = CreateObject("Scriptlet.TypeLib")
  UID = Mid(TypeLib.Guid, 2, 36)
End Function
```

## MatchSet

Text fields with the MatchSet attribute are a historical feature. These include the article number and match code for articles and addresses. These fields must be specified in capital letters. Umlauts must be converted accordingly.

```
ä -> AE
ü -> UE
ö -> OE 
ß -> SS
```

Fields with the MatchSet attribute are used in foreign systems and are often used as backward references. For this reason, only a few special characters are allowed. 

Allowed characters are:

```
A-Z (capital letters)
0-9 (digits)
.	(period)
- (hyphen)
_ (underscore)
= (equal sign)
@ (AT character as in e-mail addresses)
```

# Languages

For historical reasons, no separate XML node or language attribute is provided in the XML file. If different languages are used, they must all be specified together in the same text field. The language separator is a two-character ISO letter code, which must be specified in a separate line in square brackets. For example **[EN]** for an English section.

```
[DE]
Das ist ein Text in der Sprache Deutsch
[EN]
This is a text in the language German
[IT]
Questo è un testo in lingua tedesca
```

If a text field starts without such a language separator, this can be used to specify a default text. This is used if the translation is not available in the desired language. In the following, an English text is the default text. If the language French was requested with **[FR]**, at least an internationally readable English text would be used.

```
This is a text in the language English
[DE]
Das ist ein Text in der Sprache Deutsch
[IT]
Questo è un testo in lingua italiano
```

In the EULANDA database there are fields which do not have a sufficient length to be able to represent several languages. For example the field **Name** of a feature. This is a single-line simple text field in the database. Here the system uses a trick by allowing you to define additional subsections in the normal text.

If the single-line field is for example the field **NAME**, then in the field DESCRIPTION of a characteristic also this single-line text can be indicated as translation. The sub-field is specified with its name and a preceding colon. It must be specified in capital letters.

This translation can also have a default, in such a case the ISO abbreviation is omitted and the tag starts directly with the colon.

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
Which subfields are allowed in which place is specified in the corresponding places of the document.

# References and unique keys

Within a record, e.g. an article, the ID of the article is specified as the first field. The field content is identical to the unique key of the table. For **article** this is the **article number**. 

```xml
<ARTIKELLISTE>
	<ARTIKEL>
		<ID.ALIAS>S0221265</ID.ALIAS>
		<ARTNUMMER>S0221265</ARTNUMMER>
		<BARCODE>4023125026003</BARCODE>
		<ARTMATCH/>
		<MWSTSATZ>19.00</MWSTSATZ>
		...
	</ARTIKEL>
</ARTIKELLISTE>
```

If references to another object node are required in a file, this is done via an **ID field**. 

```xml
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

The AUFTRAGPOS node contains the field <ARTIKELID.ALIAS>S0221265</ARTIKELID.ALIAS> and thus refers to the ARTIKEL node and there the field <ID.ALIAS>S0221265</ID.ALIAS>.

## Article references

The specification of the ARTIKELLISTE node in an order is only necessary if the master data does not exist in the merchandise management system - or the target system. This is the case if the EULANDA merchandise management does not take over the article maintenance. The store must then no longer send the complete article information with each order to the merchandise management.

## Address references

The unique key field in the address section is the **MATCH** field. It is strongly recommended to use the orderer's email address as **MATCH**. If multiple order entry systems are used at the same time, it may be useful to prepend a prefix. 

In the following example, "SHOPIFY" is used as the prefix. The **MATCH** field here has the content `SHOPIFY=FACEMONTY@TOOLHEROS.DE`. For processing reasons the field **ID-ALIAS** must also be specified with the same content. The prefix should be short, because the **MATCH** field is limited to 80 characters in the database.

As a rule, the address information is also transferred with the complete master data in the order file. Store systems allow especially in the B2C area the self-registration as a new customer. It can therefore not be assumed that the address data record is already in the merchandise management system.

A total of two addresses are supported per order - the billing address and the shipping address. The delivery address always has a preceding "L" in the field names of the order. The **NAME1** field of the invoice address would then be **LNAME1** in the delivery address. 

Since an invoice address can have any number of delivery addresses and these can not be mapped easily as an EULANDA contact, the delivery address is always created as a placeholder in the master data and only referenced. The complete delivery address stands then compellingly in the order node. However, it is necessary only if this differs from the invoice address. However, it does no harm if the delivery address fields are always filled.

```xml
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

If a delivery address is used, this must have one to the master data in the merchandise management. In the XML file, a dummy data record should always be specified in the ADDRESS node. In the simplest case this looks then as follows:

```xml
<ADRESSE>
	<ID.ALIAS>SHOPIFY=SHIPPING</ID.ALIAS>
	<MATCH>SHOPIFY=SHIPPING</MATCH>
	<ZIELID.ALIAS>SHOP.PAID</ZIELID.ALIAS>
</ADRESSE>
```

# Order and repetitions

The order of XML nodes and repetitions are shown here schematically:

```xml
<?xml version="1.0" encoding="utf-8" standalone="yes"?>
<EULANDA>
	<METADATA>
		< ... Meta information ... >
	</METADATA>
	<MERKMALBAUM>
		<ARTIKEL>
			<PFAD>\Shop (Root of property tree in the target app)</PFAD>
			<MERKMAL>
				< ... Property fields ... >
				<MERKMAL>
					< ... nested property ... >
				</MERKMAL>
			</MERKMAL>
		</ARTIKEL>
	</MERKMALBAUM>
	<RABATTLISTE/>
	<ARTIKELLISTE>
		<ARTIKEL>
        	< ... Article fields ... >
		</ARTIKEL>
		<ARTIKEL>			
			< ... more articles ... >
		</ARTIKEL>		
	</ARTIKELLISTE>
	<ADRESSELISTE>
		<ADRESSE>
			< ... Address fields ... >
		</ADRESSE>
		<ADRESSE>
			< ... more addresses ... >
		</ADRESSE>
	</ADRESSELISTE>
	<AUFTRAGLISTE>
		<AUFTRAG>
			<... Header data fields ...>
			<SHOP>
				<SHIPPINGINFO>
					< ... Shipping information ... >
				</SHIPPINGINFO>
			</SHOP>
			<AUFTRAGPOSLISTE>
				<AUFTRAGPOS>
					<... Line item to the order ...>
				</AUFTRAGPOS>
				<AUFTRAGPOS>
					<... more line items for the order ...>
				</AUFTRAGPOS>
			</AUFTRAGPOSLISTE>
		</AUFTRAG>
		<AUFTRAG>
			<... more orders ... >
		</AUFTRAG>		
	</AUFTRAGLISTE>
</EULANDA>
```

If it is about a pure article data, of course the nodes ADRESSLISTE and AUFTRAGLISTE need not be listed. In this case the file name product\*.xml is to be selected. If orders are transmitted, then all nodes are to be transferred and the file name order\*.xml is to be used. If no article master data are to be indicated, it is sufficient to transfer an empty node ARTIKELLISTE. 

If article data are transferred, at least one empty MERKMALBAUM node with an enclosed empty ARTIKEL node must be specified.

```xml
<MERKMALBAUM>
		<ARTIKEL/>
</MERKMALBAUM>
```

# Metadata

The meta data stand at the file beginning and describe different standards, which are used in the XML file. There you can also set which database version of EULANDA is required and who created the file.

## XML representation

```xml
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
    <UDL>Eulanda_1 EULANDA Software GmbH.udl</UDL>
	<ALIAS>Mustermann KG</ALIAS>
	<CLIENT>Mustermann</CLIENT>
	<PARTNER>nopCommerce</PARTNER>
</METADATA>
```

## Field names METADATA

| Field Name | Description | Default | Type | Mandatory |
| --------------- | ------------------------------------------------------------ | -------- | ------------- | ---- |
| ALIAS | Specifies the client name of the EULANDA database. The specification is optional and serves only the error tracking. | | Text max: 60 | no |
| APPLICATION | Name of the program incl. path which has created this XML file. The specification is optional and is only used for error tracking. | | Text max: 100 | no |
| CLIENT | The folder name used to create the internal structure of the data exchange paths. It is a shortened variant of ALIAS and is used as folder name. The specification is optional and is only used for error tracking. |  | text max: 100 |no|
| COUNTRYFORMAT | The format for country abbreviations. Currently only **ISO2** is supported. Here **DE** would be used for Germany. | ISO2 | Text | yes |
| DATABASEVERSION | The version number of the SQL database needed for the XML file. | | Text | yes |
| DATE | Date with time of creation of the XML file. The specification is done with the format given in DATEFORMAT. Such a date would be for example: 2021-05-19T21:16:05 | | DateTime | yes |
| DATEFORMAT | format of the date. This is currently only **ISO8601** | ISO8601 | Text | yes |
| FIELDNAMES | The field names in the XML file are currently only NATIVE. So with the names as they exist in the SQL database. | NATIVE | Text | yes |
| FLOATFORMAT | Format for floating point values. Currently only **US** is allowed. Here only the leading **minus sign** and a **decimal point** are allowed next to the **digits** 0-9. | US | Text | yes |
| GENERATOR | The creator of the XML file is a free text in uppercase without umlauts. | | Text | yes |
| PARTNER | The name is a short name for the target system. It is optional and is only used for error tracking. |  | Text max: 100 |no|
| PCNAME | The field name can be left empty or the PC name of the generating PC is used; this facilitates possible error tracking. |  | Text |yes|
| UDL | Gives for test purposes the UDL file if it is the EULANDA merchandise management. The specification is optional and serves only the error tracking. | | Text max: 100 |no|
| USERNAME | The field name can be left empty, or a contact name as used by the operating system for example; this facilitates possible error searches. |  | Text |yes|
| VERSION | Version number of the file format. The current version is **1.1** | 1.1 | text | yes |

# Merkmale

The classification of products into groups is called **catalogs** or **categories** by store systems. In EULANDA we use properties for this. These are usable with any data object, so also with customers, offers, orders etc. 

Properties are organized in a tree structure. Here all are folders and only the ends can be linked to a data record, such as an article. Each data record can be referenced in any number of properties.

If, for example, article data are transferred, the complete properties tree must always be specified, even if only parts of an article base are transferred and possibly these properties are not referenced there at all. The node name for the properties is **MERKMALBAUM**. This node is to be indicated in each case, if article data in the XML file are transferred, even if properties are not used explicitly.

```xml
<MERKMALBAUM>
		<ARTIKEL/>
</MERKMALBAUM>
```

For synchronization, the initiating system provides a UID for each feature. Changes to the names or in the nestings can be processed efficiently.

## XML representation

The following is a simple feature tree three branches away from the root, The second branch is a folder (MERKMALTYP=0) that contains two end features. The first and last branches have no sub-elements, and thus are automatically end features.

```xml
<MERKMALBAUM>
	<ARTIKEL>
		<PFAD>\Shop</PFAD>
		<MERKMAL>
			<NAME>Bathroom fittings</NAME>
			<MERKMALTYP>1</MERKMALTYP>
			<BESCHREIBUNG>Bathroom fittings etc...</BESCHREIBUNG>				
			<UID>{1884B00B-50C4-402D-BA2E-61140C301099}</UID>
		</MERKMAL>
		<MERKMAL>
			<NAME>Manufacturer</NAME>
			<MERKMALTYP>0</MERKMALTYP>
			<BESCHREIBUNG>We list all well-known manufacturers, should nevertheless...</BESCHREIBUNG>			
			<UID>{1884B00B-60C4-402D-BA2E-61140C301099}</UID>
			<MERKMAL>
				<NAME>Grohe</NAME>
				<MERKMALTYP>1</MERKMALTYP>
				<BESCHREIBUNG>Grohe is since the year...</BESCHREIBUNG>			
				<UID>{2884B00B-60C4-402D-BA2E-61140C301099}</UID>
			</MERKMAL> 
			<MERKMAL>
				<NAME>Roco</NAME>
				<MERKMALTYP>1</MERKMALTYP>
				<BESCHREIBUNG>Roco is since the year...</BESCHREIBUNG>			
				<UID>{2984B00B-60C4-402D-BA2E-61140C301099}</UID>
			</MERKMAL> 				
		</MERKMAL>
			<MERKMAL>
				<NAME>Tools</NAME>
				<MERKMALTYP>1</MERKMALTYP>
				<BESCHREIBUNG>Tools for home and garden...</BESCHREIBUNG>				
				<UID>{9884B00B-50C4-402D-BA2E-61140C301099}</UID>
		</MERKMAL>
	</ARTIKEL>
</MERKMALBAUM>					
```

A fully described end feature including languages would accordingly look like the following:

```xml
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

## Field names

## MERKMALBAUM.ARTIKEL.MERKMAL


| Feld name    | Description                                                  | Standard | Type           | Mandatory |
| ------------ | ------------------------------------------------------------ | -------- | -------------- | --------- |
| BESCHREIBUNG | Various store systems or target systems allow you to add a descriptive text or image to a catalog. The description text can be specified in foreign languages, as described in the **Languages** section. Additionally, it can contain translations for the **NAME** field. |          | Text           | no        |
| BILD         | An image can be assigned to each feature. This is displayed incl. the description with many store systems in the catalog overview. If EULANDA is the initiating system, only the pure picture name is indicated here, for example **mypicture.jpg**. Images are then in the local DMS, normally in about **\\DmsRoot\\DMS\\Article**. The folder can be located in the local system or on an FTP server. However, if the article transfer is initiated by a third-party system and cannot access the local folder or a supported FTP folder, the image specification can be specified as a URL. The image will then be automatically downloaded by the interface and copied to the local DMS or FTP structure. An example for the specification of a URL would then be: "**https://www.meinserver.de/subfolder/meinbild.jpg "**. |          | FILE / URL     | no        |
| COLOR        | Contains an RGB value as a decimal number. This can be used to set the color of a feature. This field is evaluated within EULANDA to highlight features separately. However, there is nothing to prevent a target system from evaluating this value. For the transfer to the store system nopCommerce there is no use here, because there is no meaningful mapping for it there. |          | Integer 32-Bit | no        |
| DISPLAYORDER | The display order is used to sort the display order of features. If the value is not specified, then store systems or target systems often use the creation order, but also an alphabetical output is used by some. Over DISPLAYORDER one has now influence on the order of the display, if the target system supports this with nopCommerce this is the case. | 0        | Integer 0-255  | no        |
| MERKMALTYP   | Indicates which property type it is. 0=folder, 1=end property, 2=SQL property. |          | Integer 8-Bit  | yes       |
| NAME         | The name of the folder or the end element. The name is normally given in the local language, e.g. German. If foreign languages are required, the translations are stored in the field DESCRIPTION. The procedure for this is described in the **Languages** section. The name field is at the same time the standard field, if this is supported by the target system. This is possible if nopCommerce is connected as store system. If a necessary translation is missing there, the content of the standard field is used. For this reason it can be meaningful to indicate just here not in national language the text, but for example in English. |          | Text           | yes       |
| PFAD         | Is an internal specification, from which path the property tree is to be used for the article data exchange. When importing into EULANDA the tree is saved relative to this specification. The default path is "\Shop" and should be specified for compatibility. | \Shop    | Text           | (no)      |
| PUBLISHED    | Here it can be indicated whether the single property is to be indicated in the target system, or not. The specification is normally generated by EULANDA. If the transfer is initiated by a foreign system the indication is normally not necessary. | 1        | Boolean 0/1    | no        |
| SQLBEDINGUNG | The EULANDA ERP supports **intelligent** properties in addition to set single properties. These are SQL commands that interact with the database of the merchandise management. Here you can for example "formulate" a property, that selects all articles, which had a stock movement in a certain period. The specification does not really play a role in the exchange with external systems, but two EULANDA merchandise management systems can also be coupled via the interface in order to realize a multi-client or dropshipping system. |          | Text           | no        |
| TOP          | This specification can be used to control whether a category is to be presented in a special way. For example on the homepage of a store system. The specification is generated normally by EULANDA. If the transfer is initiated by a foreign system the indication is normally not necessary. | 0        | Boolean 0/1    | no        |
| UID          | The UID is necessary to synchronize structures with an external system. The initiating system should store a UID. If this is not available, then **only an initial creation can be realized**. Based on the UID, changes to the name identifiers are recognized and the structure of the feature tree can be synchronized. An example of a UID is **{1884B00B-50C4-402D-BA2E-61140C301099}**, it is the standard method used under Windows to create globally unique keys. |          | GUID           | (yes)     |

# Article master data

Article data, which is used for a pure master data creation, has the file name **product\*.xml**. Besides the usual fields of an article master, it can also contain store specific information. This can be meta information for search engines, image URLs, cross- or up-selling details, etc. 

In addition, the article master can contain the catalogs (= properties) and their assignment. Article texts can be specified in various foreign languages. For closed systems, customer groups with price lists can also be mapped in the master record.

If articles are to be transferred from the Shop to an EULANDA, then the catalog assignment is not necessary as a rule. Conversely however, if EULANDA sends article data to a Shop, these are existential for the representation in the Shop system.

[Load sample...](Samples/product-16FC10E5-E444-4CC9-A14C-743F35BC47CD.xml)

## XML representation


```xml
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
## XML file of a product\*.xml

```xml
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
                <NAME>Bathroom fittings</NAME>
                <MERKMALTYP>0</MERKMALTYP>
                <BESCHREIBUNG>Bathroom fittings</BESCHREIBUNG>
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
                <NAME>WC and accessories</NAME>
                <MERKMALTYP>0</MERKMALTYP>
                <BESCHREIBUNG>WC and accessories</BESCHREIBUNG>
                <BILD>WC and accessories.jpg</BILD>
                <PUBLISHED>1</PUBLISHED>
                <TOP>1</TOP>
                <DISPLAYORDER>0</DISPLAYORDER>
                <SQLBEDINGUNG />
                <UID>{C42223C3-C91E-4C01-A0C4-B03930B53AAF}</UID>
                <COLOR>536870911</COLOR>
                <MERKMAL>
                    <NAME>WC - seats</NAME>
                    <MERKMALTYP>0</MERKMALTYP>
                    <BESCHREIBUNG>WC - seats</BESCHREIBUNG>
                    <BILD>WC - seats.jpg</BILD>
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
                <NAME>Tools</NAME>
                <MERKMALTYP>0</MERKMALTYP>
                <BESCHREIBUNG>Tools</BESCHREIBUNG>
                <BILD>Tools.jpg</BILD>
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
            <LANGTEXT>Keramag/Geberit Cassini WC seat with soft-closing mechanism, removable - white (Alpine)</LANGTEXT>
            <INFO>Hinges: Stainless steel Only suitable for Keramag Cassini WC 203210000!</INFO>
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
                    <PFAD>\WC and accessories\WC - seats\Geberit</PFAD>
                </MERKMAL>
            </MERKMALLISTE>
        </ARTIKEL>
    </ARTIKELLISTE>
    <ADRESSELISTE />
    <AUFTRAGLISTE />
</EULANDA>
```

## Field names

## ARTIKELLISTE.ARTIKEL

| Feld name           | Description                                                  | Standard | Type                                 | Mandatory |
| ------------------- | ------------------------------------------------------------ | -------- | ------------------------------------ | --------- |
| **ID.ALIAS**        | The key field **ID.ALIAS** is the placeholder, for the ID used in the enterprise resource planning. It must be passed in any case and must have the same content as the field ARTNUMMER. |          | Text max: 80; SPECIAL-UPCASE; UNIQUE | yes       |
| ARTNUMMER           | The article number must be indicated in capital letters. The number itself may only contain letters A-Z and digits from 0-9. Umlauts are to be passed in e.g. ü in UE or ß in SS. Additionally, hyphen, dot and the underscore are allowed. The article number must be **unique**. If you import from several data suppliers and there are article number overlaps, you should consider using a prefix. For example **Unielektro** with the article number **568658997** could then be transmitted by the data supplier as **UE.568658997**. Here it would be important that this representation is used then continuously. So for purchase orders, orders, price transfers, etc. |          | Text max: 80; SPECIAL-UPCASE; UNIQUE | yes       |
| ARTNUMMERHERSTELLER | The manufacturer number can contain upper and lower case letters. The field itself may only contain letters A-Z, a-z and digits from 0-9. Additionally, hyphen, dot and the underscore are allowed. The field is ambiguous in the enterprise resource planning. |          | Text max: 80;                        | no        |
| AUSLAUFFLG          | Indicates whether the product is to be discontinued soon. The value can be used to advertise the product in a store system, provided that the interface to the third-party system supports this. | 0        | Boolean                              | no        |
| BARCODE             | The field can be used as GTIN (formerly EAN).  The field itself may only contain letters A-Z and digits from 0-9. Umlauts are to be passed in e.g. ü in UE or ß in SS. Additionally hyphen, dot and the underscore are allowed. The field is ambiguous in the merchandise management. |          | Text max: 30;                        | no        |
| BRUTTOFLG           | This boolean indicates whether the transferred price includes VAT. If BRUTTOFLG has the value 1, the sales price includes VAT. In this case, the VAT percentage is specified in the MWST field. In B2C systems it is common to specify the sales price including VAT. In this case, the end user receives an invoice with inclusive prices, in which the total VAT is deducted and displayed at the end. In contrast, a sales price without VAT is common in B2B. Here the entrepreneur gets a net invoice, where the VAT is added to the net total at the end. Both systems are possible in EULANDA parallel. By the BRUTTOFLG is achieved that end customer prices are cent-exact. |          | BOOLEAN                              | yes       |
| CHANGEDATE          | The modification date of the record.                         |          | DateTime                             | no        |
| EKNETTO             | The purchase price is indicated without VAT. The EKNETTO is in two digits (cents). |          | FLOAT 18.2                           | no        |
| GEWICHT             | The weight must be specified in the unit kg.                 | 0        | FLOAT 18.4                           | yes       |
| INFO                | The Info field has any length. It stores only ANSI so no Unicode special characters. The ANSI conversion depends on the operating system language. The text can contain foreign languages. The field Info should contain a supplementary article description. This in the store usually under the article picture. The Info field uses language tags to specify foreign languages. The **Languages** section describes how to specify them. |          | Text                                 | yes       |
| KURZTEXT1           | The short text 1 contains a short description of the product, in combination with the short text 2. These two texts have less importance today and are mainly necessary for data exchange according to DATANORM/ELDANORM |          | Text max: 100                        | no        |
| KURZTEXT2           | The short text 2 contains a short description of the product, in combination with the short text 1. These two texts have less importance today and are mainly necessary for data exchange according to DATANORM/ELDANORM. |          | Text max: 100                        | no        |
| LANGTEXT            | The long text has any length. It stores only ANSI, so no Unicode special characters. The ANSI conversion depends on the operating system language. The text can contain foreign languages, as well as a multilingual short text, as it is used in store systems to create product headings. The pure long text should be a medium-length product description, which is normally placed next to the article image in the store. Within the merchandise management system, the long text is used in invoices and quotations, etc. as the product text. Depending on the scenario, however, the short text1/2 or the ultra-short text can also be defined for it there. For this reason it is better to fill short text1/2 and ultra short text with data as well. The long text uses language tags to specify foreign languages. In section **Languages** is described how to specify them. Furthermore, the long text can contain the translations for a short text. Here the tag **[DE:SHORT]** is to be used for the German short text. |          | Text                                 | yes       |
| LOESCHFLG           | Indicates that the product can be deleted in the target system. The store system must react to this accordingly and the interface to the store must also support the field in a suitable manner. | 0        | Boolean                              | no        |
| MATCH               | The field must be specified in capital letters. The field is used to quickly find products and is used in overview lists. The field itself may only contain letters A-Z and digits from 0-9. Umlauts are to be passed in e.g. ü in UE or ß in SS. Additionally hyphen, dot and the underscore are allowed. The field is ambiguous in the merchandise management. |          | Text max: 80; UPCASE;                | no        |
| MWSTSATZ            | The VAT rate is shown as a percentage. The default value in Germany is (as of 09/2021 = 19.00). |          | Float                                | yes       |
| NEUFLG              | Indicates that this is a new product. In the target system, such as a store system, products with this property can then be marked separately. Such a marking could be a rubber band (= rubber band) or a stamp on the product image. The store system must react accordingly to this and the interface to the store must also support the field in a suitable manner. | 0        | Boolean                              | no        |
| PREISEH             | The price unit indicates whether the VK, VKNETTO or VKBRUTTO is a unit price, or a price per ten, or any other. It is a divisor, which is only used when the invoice is created. This results in very accurate rounding for unit prices that are less than 1 cent. If prices are indicated with price unit 100, one has two decimal places more at accuracy. | 1        | FLOAT 10.2                           | yes       |
| SHOPEXPORTDATUM     | The last export date of the item if EULANDA is the initiating system. |          | DateTime                             | no        |
| SHOPFREIGABEFLG     | This a field within the merchandise management, which indicates whether the article may be published. During import the field has less relevance and is used especially for EULANDA to EULANDA transfers to synchronize the article master in dependent systems. | 0        | Boolean                              | no        |
| SONDERFLG           | Indicates that this is a special offer. In the target system, such as a store system, products with this property can then be marked separately. Such a marking could be a rubber band or a stamp on the product image. The store system must respond accordingly and the interface to the store must also support the field in a suitable manner. | 0        | Boolean                              | no        |
| ULTRAKURZTEXT       | The Ultrakurtext contains a very compact article description, which is used especially on the slip at POS systems. |          | Text max: 100                        | no        |
| VERPACKEH           | The packaging unit indicates how many identical parts are contained in a product. This information is only used when printing offers or invoices. | 1        | FLOAT 10.2                           | yes       |
| VK                  | The sales price can be specified either with or without VAT. The price will be transferred to the inventory management database in the same way as it is entered. So that the system can calculate internally correctly the additional field BRUTTOFLG is necessary in any case. The sales price has two digits (to the nearest cent). |          | FLOAT 18.2                           | yes       |
| VKNETTO             | Regardless of the sales price stored in the database, it is also possible to output a pure sales price without VAT. This is not a database field, but a calculated field. It cannot be imported into the merchandise management system, but is used for simple further processing by external systems. However, it is common to specify this field in general. The VKNETTO is two-digit (cent-exact). |          | FLOAT 18.2                           | (no)      |
| VKBRUTTO            | Regardless of the sales price stored in the database, a pure sales price with VAT can also be output. This is not a database field, but a calculated field. It cannot be imported into the merchandise management system, but is used for simple further processing by external systems. However, it is common to specify this field in general. The VKBRUTTO has two digits (to the nearest cent). |          | FLOAT 18.2                           | (no)      |
| VOLUMEN             | The volume must be specified in the unit liter.              | 0        | FLOAT 18.4                           | yes       |
| WAEHRUNG            | The currency contains EUR for EURO by default. To process other currencies, the EULANDA merchandise management needs an additional module **multi currency**. These should be set therefore only after consultation of external systems other than **EUR**. | EUR      | Text max: 3                          | yes       |



## Field names

## ARTIKELLISTE.ARTIKEL.SHOP

These field names are only allowed in the SHOP node below the ARTIKEL node. 

> NOTE: They require the installation of the store extension, which is part of the interface.


| Feld name          | Description                                                  | Standard | Type                     | Mandatory |
| ------------------ | ------------------------------------------------------------ | -------- | ------------------------ | --------- |
| ARTICLETYPE        | The field indicates what type of item it is. It is used only in connection with the XML interface. Type=0: The item is not yet specified. Type=1: It is a simple sales item. Type=2: It is a MASTER article. Type=3: It is a variant. Type=4: It is a variant with single listing. | 0        | Integer 8-Bit; 0,1,2,3,4 | no        |
| BASEDIVISOR        | The BASEDIVISOR specifies how the product should be displayed in a store in addition to the product price. According to the Price Indication Ordinance, a customary reference unit with converted price must also be displayed near the product price. If the shampoo offered in 250 ml is to be displayed with a price per liter, the BASEDIVISOR is to be set to 1 and the BASEUNIT accordingly to **l**. If, on the other hand, the product is to be displayed in 100 ml, the BASEDIVISOR would be 100 and the BASEUNIT would be **ml**. | 1        | Float 18.4               | no        |
| BASEUNIT           | The base unit is specified in the BASEUNIT field. The logic for this has already been described under BASEDIVISOR. |          | Text max: 8              | no        |
| CROSS1 .. CROSS6   | Crosselling products show compatible products or accessories that are usually listed on a store system on the product detail page below. Here an article number is expected, which is known in the system. |          | Text max: 32             | no        |
| DIMENSIONDEPTH     | Product depth in mm                                          |          | Integer                  | no        |
| DIMENSIONHEIGHT    | Product height in mm                                         |          | Integer                  | no        |
| DIMENSIONWIDTH     | Product width in mm                                          |          | Integer                  | no        |
| IMAGE1 .. IMAGE15  | These fields can each contain an image URL to product images. The file name, of the jpg or png file, should be structured to contain the article number followed by a hyphen and a running image number from 1-15. If a different file name is chosen, the image can only be used for initial data transfer. The numbering ensures that images can be added, deleted or exchanged in the system. For the article number **4711**, the first image would then be a URL like: **`http://www.eulanda.eu/images/4711-1.jpg`** would be a good choice. Images should all be of the same image type. The image extension should be 3 digits, so not **jpeg** but **jpg**. The sequence numbers should have no gaps in the numbering. The interface then downloads the images via the URL during import and stores them in the local DMS. The resolution should be 2000x2000 pixels if possible (not by enlargement), the proportion should be square if possible. For images with background, it should be pure white. Preferably, images should be on transparent background and be in png format. The main image should have the sequence number 1. Technical drawings should be the last images in the sequence. |          | URL max: 64              | no        |
| INFOURL            | A URL to more detailed information. The field is usually not used because there are better methods today. |          | Text max: 128            | no        |
| INFOURLTEXT        | The text to the INFOURL to create an HTML link from it. The field is usually not used because there are better methods today. |          | Text max: 64             | no        |
| METADESCRIPTION    | The METADESCRIPTION is needed as meta information in HTML pages to provide search engine friendly content. |          | Text                     | no        |
| METAKEYWORDS       | The METAKEYWORDS is needed as meta information in HTML pages to provide search engine friendly content. |          | Text                     | no        |
| METATITLE          | The METATITLE is needed as meta information in HTML pages to provide search engine friendly content. |          | Text max: 128            | no        |
| SALESSIZE          | The sales quantity of the contents, such as a shampoo. If this is usually indicated in ml, it could then say 250. | 0        | Float 18.4               | no        |
| SALESUNIT          | The sales unit in terms of quantity SALESSIZE of the contents. A shampoo is usually indicated in ml. |          | Text max: 8              | no        |
| SHIPPINGFREE       | Freight free                                                 | 0        | Boolean                  | no        |
| SHIPPINGWEIGHT     | In contrast to the product weight, which is specified in the WEIGHT field, the weight including the product packaging can be specified via SHIPPINGWEIGHT. The specification is made in kg. |          | Float 18.4               | no        |
| SUGGESTEDLISTPRICE | Contains the recommended retail price incl. VAT.             |          | Float 18.2               | no        |
| UP1 .. UP6         | Up-selling products show higher-value products that are usually offered in a store system during checkout. Here an article number is expected, which is known in the system. |          | Text                     | no        |

## Field names

## ARTIKELLISTE.ARTIKEL.LAGER

The stock numbers can be passed together with the master data record. Additionally there is an optimized XML file stock\*.xml, which then contains only the stock numbers in a simplified node.

| Feld name          | Description                                                  | Standard | Type       | Mandatory |
| ------------------ | ------------------------------------------------------------ | -------- | ---------- | --------- |
| BESTANDVERFUEGBAR  | Indicates the absolute stock availability. This is the usual field for passing the stock availability. |          | Float 10.2 | no        |
| BESTANDVERFUEGBAR1 | Indicates the stock availability from BESTANDVERFUEGBAR, but less reservations due to sales orders. This field is usually not transmitted. |          | Float 10.2 | no        |
| BESTANDVERFUEGBAR2 | Indicates the stock availability from BESTANDVERFUEGBAR1, but plus purchase orders placed. This field is usually not transmitted. |          | Float 10.2 | no        |



## Field names

## ARTIKELLISTE.ARTIKEL.MERKMALLISTE.MERKMAL

| Feld name | Description                                                  | Standard | Type          | Mandatory |
| --------- | ------------------------------------------------------------ | -------- | ------------- | --------- |
| PFAD      | Indicates the absolute stock availability. The individual path sections are separated by backslash. For example: **\Manufacturer\Geberit**. This field can be repeated in the MERKMAL node as required. If paths are specified, the complete feature tree with its structure must be specified in each case. |          | Text max: 200 | no        |

# Price changes

Price changes can be transferred in a separate file named **price\*.xml**. These can be exported to a store system, for example, or received from a wholesaler.

If the price change file is exported, it must be output to the **\outbox\pending** folder. If the price change file is imported by a wholesaler, it must be output to the **\inbox\pending** folder.

[Load sample...](Samples/price-52D977FD-002C-4494-AE4F-D41C52468BEB.xml)

## XML representation

```xml
<?xml version="1.0" encoding="UTF-8"?>
<EULANDA>
   <METADATA>
      <VERSION>1.1</VERSION>
      <GENERATOR>NOPCOMMERCE</GENERATOR>
      <DATEFORMAT>ISO8601</DATEFORMAT>
      <FLOATFORMAT>US</FLOATFORMAT>
      <COUNTRYFORMAT>ISO2</COUNTRYFORMAT>
      <FIELDNAMES>NATIVE</FIELDNAMES>
      <DATE>2021-09-27T18:46:37</DATE>
      <PCNAME>BETA7276</PCNAME>
      <USERNAME>ADMINISTRATOR</USERNAME>
      <DATABASEVERSION>5.63</DATABASEVERSION>
      <APPLICATION>C:\all\Test\Eul.exe</APPLICATION>
      <UDL>Eulanda_1 EULANDA Software GmbH.udl</UDL>
      <ALIAS>EULANDA Software GmbH</ALIAS>
      <CLIENT>Eulanda</CLIENT>
      <PARTNER>nopCommerce</PARTNER>
   </METADATA>
   <MERKMALBAUM>
      <ARTIKEL />
   </MERKMALBAUM>
   <RABATTLISTE />
   <ARTIKELLISTE>
      <ARTIKEL>
         <ID.ALIAS>3000250531</ID.ALIAS>
         <ARTNUMMER>3000250531</ARTNUMMER>
         <EKNETTO>68.00</EKNETTO>
         <BRUTTOFLG>0</BRUTTOFLG>
         <VK>84.00</VK>
         <VKNETTO>84.00</VKNETTO>
         <VKBRUTTO>99.96</VKBRUTTO>
      </ARTIKEL>
      <ARTIKEL>
         <ID.ALIAS>3000280269</ID.ALIAS>
         <ARTNUMMER>3000280269</ARTNUMMER>
         <EKNETTO>16.00</EKNETTO>
         <BRUTTOFLG>0</BRUTTOFLG>
         <VK>19.12</VK>
         <VKNETTO>19.12</VKNETTO>
         <VKBRUTTO>22.75</VKBRUTTO>
      </ARTIKEL>
   </ARTIKELLISTE>
   <ADRESSELISTE />
   <AUFTRAGLISTE />
</EULANDA>
```

## Field names

## ARTIKELLISTE.ARTIKEL

| Feld name    | Description                                                  | Standard | Type                                 | Mandatory |
| ------------ | ------------------------------------------------------------ | -------- | ------------------------------------ | --------- |
| **ID.ALIAS** | The key field **ID.ALIAS** is the placeholder, for the ID used in the enterprise resource planning. It must be passed in any case and must have the same content as the field ARTNUMMER. |          | Text max: 80; SPECIAL-UPCASE; UNIQUE | yes       |
| ARTNUMMER    | The article number must be indicated in capital letters. The number itself may only contain letters A-Z and digits from 0-9. Umlauts are to be passed in e.g. ü in UE or ß in SS. Additionally, hyphen, dot and the underscore are allowed. The article number must be **unique**. If you import from several data suppliers and there are article number overlaps, you should consider using a prefix. For example **Unielektro** with the article number **568658997** could then be transmitted by the data supplier as **UE.568658997**. Here it would be important that this representation is used then continuously. So for purchase orders, orders, price transfers, etc. |          | Text max: 80; SPECIAL-UPCASE; UNIQUE | yes       |
| BRUTTOFLG    | This boolean indicates whether the transferred price includes VAT. If BRUTTOFLG has the value 1, the sales price includes VAT. In this case, the VAT percentage is specified in the MWST field. In B2C systems it is common to specify the sales price including VAT. In this case, the end user receives an invoice with inclusive prices, in which the total VAT is deducted and displayed at the end. In contrast, a sales price without VAT is common in B2B. Here the entrepreneur gets a net invoice, where the VAT is added to the net total at the end. Both systems are possible in EULANDA parallel. By the BRUTTOFLG is achieved that end customer prices are cent-exact. |          | BOOLEAN                              | yes       |
| EKNETTO      | The purchase price is indicated without VAT. The EKNETTO is in two digits (cents). |          | FLOAT 18.2                           | no        |
| VK           | The sales price can be specified either with or without VAT. The price will be transferred to the inventory management database in the same way as it is entered. So that the system can calculate internally correctly the additional field BRUTTOFLG is necessary in any case. The sales price has two digits (to the nearest cent). |          | FLOAT 18.2                           | yes       |
| VKNETTO      | Regardless of the sales price stored in the database, it is also possible to output a pure sales price without VAT. This is not a database field, but a calculated field. It cannot be imported into the merchandise management system, but is used for simple further processing by external systems. However, it is common to specify this field in general. The VKNETTO is two-digit (cent-exact). |          | FLOAT 18.2                           | (no)      |
| VKBRUTTO     | Regardless of the sales price stored in the database, a pure sales price with VAT can also be output. This is not a database field, but a calculated field. It cannot be imported into the merchandise management system, but is used for simple further processing by external systems. However, it is common to specify this field in general. The VKBRUTTO has two digits (to the nearest cent). |          | FLOAT 18.2                           | (no)      |


# Address master data

Addresses can be created via an XML file, but it is also possible that addresses are created via an order XML file. However, the structure of the address node is identical. If only addresses are transferred, they have the file name address*.xml.

Within the merchandise management all addresses are stored in a table and later qualified to suppliers, customers etc.. In the address management, invoice and delivery addresses are stored accordingly. Due to the too different procedures how external systems handle delivery addresses, only a placeholder address is stored in the master data area. This is used for internal referencing.

## XML representation

The following section contains two addresses. The first one is a full address, the second one is a dummy address to support a link as delivery address.

```xml
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

## Field names

## ADRESSELISTE.ADRESSE

| Feld name        | Description                                                  | Standard | Type                   | Mandatory |
| ---------------- | ------------------------------------------------------------ | -------- | ---------------------- | --------- |
| **ID.ALIAS**     | The key field **ID.ALIAS** is the placeholder, for the ID used in the merchandise management. It must be passed in any case and must have the same content as the MATCH field. |          | Text max: 80; MatchSet | yes       |
| **ZIELID.ALIAS** | Here the name of the payment term used internally by EULANDA can be used. But also a new unknown name can be used here. The import would then first create a corresponding payment condition in the master data and reference it with the address. Common names would be SHOP.PAID, SHOP.PREPAID, SHOP.PAYPAL etc. |          | Text max: 100          | yes       |
| EMAIL            | The e-mail address in the usual spelling.                    |          | Text max: 64           | yes       |
| LAND             | The country of the address is indicated with the two-digit ISO abbreviation. So FR for France, DE for Germany, etc. Internally, the merchandise management system can also work with the country abbreviations from the motor vehicle sector. In this case, Germany would be D instead of DE. However, only the two-digit ISO abbreviation may be used in the interface. The conversion to the internally used format is done automatically. |          | Text max: 6            | yes       |
| MATCH            | The MATCH field is the master key to the address and must be unique. A **special feature** is that the **ID.ALIAS** field must be set with the same content. If the address is generated in a third-party system, such as a store system, it is strongly recommended to use the e-mail address of the orderer for this purpose. The field must be converted to uppercase. If a prefix is used to record the generating system, this can be separated with an equal sign. The Match field only allows characters from the MatchSet, which are described at the beginning of the document. Especially umlauts like **ä** have to be converted to **AE**. |          | Text max: 80; MatchSet | yes       |
| NAME1            | The name range of an address consists of three possible name lines. They can be used freely and as they are normally used in letter addressing. Usually NAME1 contains the company name. For private persons, the field can also be left blank. |          | Text max: 40           | yes       |
| NAME2            | The NAME2 field is usually used for first name and last name. For forms, this is then the contact person. |          | Text max: 40           | yes       |
| NAME3            | The NAME3 field is usually used for special designations, this can be the mailbox, a department or an otherwise important addition. |          | Text max: 40           | no        |
| ORT              | Contains the place name to the address.                      |          | Text max: 40           | yes       |
| PLZ              | The field contains the postal code.                          |          | Text max: 15           | yes       |
| STRASSE          | The field contains the street to the address incl. the house number. The house number can be separated from the street name with the usual special characters. For Italy, for example, this would be a comma. |          | Text max: 40           | yes       |
| TEL              | The phone number to the address. For national numbers, the international prefix, such as +49, can be omitted. Extensions can be separated with a hyphen. This can also separate the area code from the pure phone number. |          | Text max: 30           | no        |

# Order with article master data

## XML file of an order\*.xml

The following file contains a complete order including the master data in a single XML file. Within the XML file the master data is referenced by an ID system.

[Load sample...](Samples/order-3930-32FC10E5-E544-4CC9-A14C-743F35BC47CD.xml)

```xml
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

# Order without article master data

## XML file of an order\*.xml

The same order, without article master data but with the address master data would look accordingly as follows.

```xml
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

It should be noted that the ARTIKELLISTE node must be specified in any case, even if no articles are transferred.

```xml
<ARTIKELLISTE>
	<ARTIKEL/>			
</ARTIKELLISTE>
```

## Field names

## AUFTRAGLISTE.AUFTRAG

| Feld name            | Description                                                  | Standard   | Type          | Mandatory |
| -------------------- | ------------------------------------------------------------ | ---------- | ------------- | --------- |
| **ADRESSEID.ALIAS**  | Contains the key for the address reference. It must have the same content as the corresponding **ID.ALIAS** field of the **ADRESSE** node. |            | Text          | yes       |
| **LADRESSEID.ALIAS** | Contains the key for the delivery address reference. It must have the same content as the corresponding **ID.ALIAS** field of the **ADRESSE** node. The shipping address must be specified only if it is different from the billing address. If it is specified, the LADRESSEID.ALIAS must point to the dummy address record of the delivery address in the ADRESSE node - i.e. have the same content as the ID.ALIAS field of the delivery address there. Since the delivery address in the root must be empty, the delivery address fields LNAME1, LSTRASSE etc. must be filled in any case if the delivery address is different. |            | Text          | (yes)     |
| **ZIELID.ALIAS**     | Contains the key field for the payment term. Here the internally by EULANDA used name of the payment condition can be used. But also a new unknown name can be used here. The import would then first create a corresponding payment condition in the master data and reference it with the address. Common names would be SHOP.PAID, SHOP.PREPAID, SHOP.PAYPAL etc. |            | Text max: 100 | yes       |
| BESTELLDATUM         | The date of the order is usually assigned by the user in a store system. This is usually the same date as the DATUM field. |            | DateTime      | yes       |
| BESTELLNUMMER        | Order number of the external system used for referencing. This is managed in the merchandise management system as the customer order number. |            | Text max: 30  | yes       |
| BRUTTOFLG            | This field indicates whether the sales price **VK** is a price including VAT. In a B2C order, this is usually the case. In a B2C online store, the prices are usually stated including VAT. The shopping cart is added and from the sum the VAT is calculated and indicated. There is only rounding. In order to be able to represent this in the merchandise management, these identical prices must be transferred in such a case and this is indicated with the BRUTTOFLG. |            | Boolean       | yes       |
| DATUM                | Date and time of the order creation in the external system.  |            | DateTime      | yes       |
| LAND                 | The country of the address is indicated with the two-digit ISO abbreviation. So FR for France, DE for Germany, etc. Internally, the merchandise management system can also work with the country abbreviations from the motor vehicle sector. In this case, Germany would be D instead of DE. However, only the two-digit ISO abbreviation may be used in the interface. The conversion to the internally used format is done automatically. |            | Text max: 6   | yes       |
| LLAND                | The country of the delivery address is indicated with the two-digit ISO abbreviation. So FR for France, DE for Germany, etc. Internally, the merchandise management system can also work with the country abbreviations from the motor vehicle sector. In this case, Germany would be D instead of DE. However, only the two-digit ISO abbreviation may be used in the interface. The conversion to the internally used format is done automatically. |            | Text max: 6   | yes       |
| LNAME1               | The name range of a delivery address consists of three possible name lines. They can be used freely. Usually, LNAME1 contains the company name. For private persons, the field can also be left blank. The delivery address only needs to be specified if it differs from the billing address. However, if this is specified, LADRESSEID.ALIAS must also be specified. |            | Text max: 40  | (no)      |
| LNAME2               | The LNAME2 field of the delivery address is usually used for first name and last name. For companies this is then the contact person. |            | Text max: 40  | (no)      |
| LNAME3               | The LNAME3 field of the delivery address is usually used for special designations; this can be the mailbox, a department or an otherwise important addition. |            | Text max: 40  | (no)      |
| LSTRASSE             | The field contains the street to the address incl. the house number. The house number can be separated from the street name with the usual special characters. For Italy, for example, this would be a comma. |            | Text max: 40  | yes       |
| LORT                 | Contains the city name for the delivery address.             |            | Text max: 40  | yes       |
| LPLZ                 | The field contains the postal code for the delivery address. |            | Text max: 15  | yes       |
| NAME1                | The name range of an address consists of three possible name lines. They can be used freely. Usually, NAME1 contains the company name. For private persons, the field can also be left blank. |            | Text max: 40  | yes       |
| NAME2                | The NAME2 field is usually used for first name and last name. For companies, this is then the contact person. |            | Text max: 40  | yes       |
| NAME3                | The NAME3 field is usually used for special designations; this can be the mailbox, a department, or an otherwise important addition. |            | Text max: 40  | no        |
| OBJEKT               | An object designation, this can be a construction site or any other term that the customer chooses. As a rule, simply **Onlineshop** is entered there. It is not a required field, but it helps to search manually in the order data. We recommend to use it. | Onlineshop | Text          | (no)      |
| ORT                  | Contains the place name to the address.                      |            | Text max: 40  | yes       |
| PLZ                  | The field contains the postal code.                          |            | Text max: 15  | yes       |
| SHOPEMAIL            | Contains the e-mail address of the invoice recipient. This e-mail has priority over any e-mail from the address master data. When the invoice document is sent, this e-mail is used. Only if none is specified, the e-mail from the address master will be used. |            | Text max: 64  | (yes)     |
| SHOPLEMAIL           | Contains the e-mail address of the delivery address.         |            | Text max: 64  | (yes)     |
| SHOPLTEL             | The telephone number of the delivery address.                |            | Text max: 30  | no        |
| SHOPTEL              | The phone number of the invoice recipient. This phone number has priority over any phone number from the address master data. |            | Text max: 30  | no        |
| STRASSE              | The field contains the street to the address incl. the house number. The house number can be separated from the street name with the usual special characters. For Italy, for example, this would be a comma. |            | Text max: 40  | yes       |

## Field name

## AUFTRAGLISTE.AUFTRAG.SHOP.SHIPPINGINFO

| Feld name | Description                                 | Standard | Type       | Mandatory |
| --------- | ------------------------------------------- | -------- | ---------- | --------- |
| COST      | Includes the shipping cost of the shipment. |          | Float 18.2 | no        |

## Field name

## AUFTRAGLISTE.AUFTRAG.AUFTRAGPOSLISTE.AUFTRAGPOS

These minimal fields of the order item are sufficient if the article master is known and referenced at this point via the ARTIKELID.ALIAS. Missing fields are then automatically completed by the master data.

| Feld name           | Description                                                  | Standard | Type          | Mandatory |
| ------------------- | ------------------------------------------------------------ | -------- | ------------- | --------- |
| **ARTIKELID.ALIAS** | Contains the key for the article reference. It must have the same content as the corresponding **ID.ALIAS** field of the **ARTIKEL** node. |          | Text max: 100 | yes       |
| MENGE               | Sales quantity of the item.                                  |          |               | yes       |
| BASIS               | Base price of the item, this is the purchase price incl. special conditions that have led to this order. Often it is the same as the purchase price of the item master. The base price can be specified with or without VAT. If the order's BRUTTOFLG is set to 1, all prices of the item must be specified including VAT, otherwise without VAT. |          | Float 18.2    | (no)      |
| VKRAB               | The discounted selling price of this item based on one piece. The VKRAB can be specified with or without VAT. If the order's BRUTTOFLG is set to 1, all prices of the item must be specified including VAT, otherwise without VAT. |          | Float 18.2    | yes       |
| VKVRAB              | The sales price of the item without discount. If the order BRUTTOFLG is set to 1, all prices of the item must be inclusive of VAT, otherwise without VAT. This field is needed to show the discount in an invoice later. |          | Float 18.2    | yes       |

# Status message

Via a status message the carrier and the tracking number can be transmitted to the order entry system like for example a store system after a parcel shipment. Normally, status messages are exported from EULANDA® in this way as an XML file. The information is then transferred in the second step to a store system like nopCommerce.

But it is also possible that a wholesaler drop-ships and sends the ordered goods directly to the end user and sends a status message to EULANDA®. In this case EULANDA® imports the XML file and can then, depending on the stored workflow, book an order, create a delivery and generate an invoice. This can also be sent directly to the end user by e-mail via the interface.

[Load sample...](Samples/status-9430-11FD10E5-E444-4CC9-A14C-743F35BC47CD.xml)

## XML representation

```xml
<?xml version="1.0" encoding="UTF-8"?>
<EULANDA>
   <METADATA>
      <VERSION>1.1</VERSION>
      <GENERATOR>SHOPIFY</GENERATOR>
      <DATEFORMAT>ISO8601</DATEFORMAT>
      <FLOATFORMAT>US</FLOATFORMAT>
      <COUNTRYFORMAT>ISO2</COUNTRYFORMAT>
      <FIELDNAMES>NATIVE</FIELDNAMES>
      <DATE>21-09-2021T18:35:20</DATE>
      <PCNAME />
      <USERNAME />
      <DATABASEVERSION>5.63</DATABASEVERSION>
   </METADATA>
   <AUFTRAGLISTE>
      <AUFTRAG>
         <BESTELLNUMMER>NW-3930</BESTELLNUMMER>
         <SHOP>
            <TRACKING>38110844104</TRACKING>
            <CARRIER>TNT</CARRIER>
         </SHOP>
      </AUFTRAG>
   </AUFTRAGLISTE>
</EULANDA>
```

## Field names
## AUFTRAGLISTE.AUFTRAG
| Feld name     | Description                                                  | Standard | Type | Mandatory |
| ------------- | ------------------------------------------------------------ | -------- | ---- | --------- |
| BESTELLNUMMER | Order number of the external system used for referencing. This is managed in the merchandise management system as the customer order number. If the status message is sent by the drop-shipping system, it must be identical to the previously triggered order. |          | Text | ja        |



## Field names

## AUFTRAGLISTE.AUFTRAG.SHOP

| Feld name | Description                                                  | Standard | Type         | Mandatory |
| --------- | ------------------------------------------------------------ | -------- | ------------ | --------- |
| TRACKING  | The field normally contains one tracking number per shipment. If the shipment consists of multiple packages, the tracking numbers can be separated by CRLF (hex 0D0A) each in a separate line. The field is dynamic and has no size limit. |          | Text         | ja        |
| CARRIER   | The carrier such as GLS, TNT, DHL, etc. with which the package was shipped. If the shipper is missing in the master data of the ERP, it will be created automatically. Outgoing e-mails can be stored via a template, for known carriers the shipment tracking is provided as a link. |          | Text max: 10 | ja        |

