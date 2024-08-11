---
title: "Meerkat NameParser"
date: 2017-04-11 19:30:00Z
draft: false
---
I've released a little project [Meerkat.NameParser](https://www.nuget.org/packages/Meerkat.NameParser/) which assists in splitting up personal names into their component parts.

This code has been around in one form or another since around 1988 when I wrote the first version in Informix 4GL as part of a project for a client. Subsequent versions were written in Clipper, Visual Basic and this one in C#.

It's not perfect by any means but on projects I've used it on we were getting error rates of ~2 per 100K.

It works on a heuristic basis trying to identify known name parts and then inferring the rest from the relative position of the known values.

With no other information is presumes that someone has multiple given names and a single family name; the terms are used to avoid the ordering implied by forename and surname and makes it easier to handle other cultures where the ordering differs.

We break a personal name up into components as follows...

* **Title**: Professional, civil or military titles
* **Given**: Given name of the person
* **Family**: Family name of the person
* **Letters**: Post-nominal letters, could be professional, academic, military etc
* **Envelope**: How the name appears on an envelope
* **Salutation**: How to formally address the person in a letter, e.g. Sir George Bingham should be ***Dear Sir George***, not ***Dear Sir Bigham***

The family name is a structure in its own right allowing us to collate names more easily e.g. ***John van Damn II*** splits as follows

* **Given**: John
* **Family**:
    * **Prefix**: van
    * **Name**: Damn
    * **Suffix**: II

We generally strip punctuation but use hyphens (*-*) as binders to join elements together e.g. ***Jean-Paul Smith*** vs ***Paul Smith-Jones***.

All of this works by using a simple tokenizer which splits the name into tokens and then tries to recognise them in a symbol table; this gives us the positional estimates for each name element and then we perform a little bit of fixup once we finish processing to improve our likelihood of being correct.

One test case was inspired by a challenge from one of my colleagues on a project ***Buffy the Vampire Slayer*** and yes we can parse this..

* **Title**: The Vampire Slayer
* **Given**: Buffy

This also works nicely for ***Richard the Lionheart*** and ***Kermit the Frog*** :smile:

The really nasty case trying parse the name ***The Rt Hon The Lord The admiral of the fleet Peter Edward Walker of Worcester II MBE PC*** as

* **Title**: The Rt Hon The Lord The Admiral of The fleet
* **Given**: Peter Edward
* **Family**:
  * **Name**: Walker of Worcester
  * **Suffix**: II

Have a look at the [code](https://github.com/phatcher/Meerkat.NameParser) to see some more examples.

### Customisation

The standard parser will work for most scenarios but if you are dealing with a different culture where the rules differ, e.g. Spanish where people have compound forenames, you might need to adjust the name table.

The standard symbol table is held as a embedded resource in the assembly `PersonNames.json` and there is a static factory class which create a parser for you using this `ParserFactory.StandardPersonParser(true)`.

There are a couple of methods here which allow you to load your own `SymbolTable` from a file...

```csharp
public static ISymbolTable FileSymbolTable(string fileName, bool cache = true)
{
    Func<string, SymbolTable> f = x =>
    {
        var data = File.ReadAllText(x);

        return (SymbolTable) SymbolTable(data);
    };

    return SymbolTable(fileName, cache, f);
}
```

This can then be assigned to the parser's lexical analysis module e.g.

```csharp
var parser = new PersonNameParser();
parser.Lex.SymbolTable = ParserFactory.FileSymbolTable("mySymbols.json");  
```

The symbol table structure is serialized `NameSymbol` objects which allows us to carry additional information with the symbols to be used by subsequent processes.

Here's an extract of the current symbols as an example

```
{ "NameType": "Title", "Value": "Admiral" },
{ "NameType": "Title", "Value": "Baron", "Properties":{ "Gender": "M" } },
{ "NameType": "Title", "Value": "Baroness", "Properties":{ "Gender": "F" } },
{ "NameType": "Title", "Value": "Bishop", "Properties":{ "Gender": "M", "SalutationFormat": "My Lord Bishop" } },
{ "NameType": "Title", "Value": "Brigadier" },
{ "NameType": "Title", "Value": "Captain" },
...
{ "NameType": "Given", "Value": "Robert", "Properties":{ "NameCycle": "Robert", "Gender": "M" }},
{ "NameType": "Given", "Value": "Bob", "Properties":{ "NameCycle": "Robert","Gender": "M" }},
...
{ "NameType": "Family", "Value": "da" },
{ "NameType": "Prefix", "Value": "al" },
...
{ "NameType": "Civil", "Value": "ARRC", "Properties":{ "Group": 1, "Order": 25, "Name": "Associate Royal Red Cross" } },
{ "NameType": "Civil", "Value": "Bart", "Properties":{ "Group": 0, "Order": 1, "Name": "Baronet" } },
...
{ "NameType": "Military", "Value": "VC", "Properties":{ "Group": 1, "Order": 1, "Name": "Victoria Cross" } },
...
{ "NameType": "Professional", "Value": "CEng", "Properties":{ "Level": "Chartered", "Society": "Engineering Council", "Name": "Chartered Engineer" } },
{ "NameType": "Professional", "Value": "MBCS", "Properties":{ "Level": "Member", "Society": "British Computer Society" } },           
```

Let me know if you find this useful.