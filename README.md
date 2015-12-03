#Prestanda- och säkerhetsanalys av applikation
**Erik Hamrin (eh222ve)**

##Säkerhetsproblem
###SQL-injections

SQL-injections är när en användare lyckas exekvera opålitlig data i en databas-fråga och på så vis kunna påverka en applikations persitenta lagring.

På inloggningssidan går det att logga in som en användare/administratör utan att känna till dess annvändaruppgifter och på så sätt få full tillgång till applikationen.

Det mest förespråkade enligt OWASP är att använda sig utav parametriserade frågor, stored procedures eller liknande som hindrar kod i parametrar som exekveras i tolken.[2] 

###Känslig data

Uppgifter som anses vara privata bör göras oläsliga när de skall lagras i en applikation, annan privat data skall ses över så att den inte kan kommas åt publikt.[2] 

Om en besökare går till /message/data (genom att analysera JavaScript-filen, eller HTTP-svaret som presenteras på inloggningssidan) visas alla meddelanden.

Om en person kommer åt databasen står alla lösenord i klartext. 

För att undvika detta bör alla lösenord hashas och  autentisiering/auktorisering begäras för att se meddelanden.

###Cross-Site Scripting (XSS)

Data som presenteras för användare har inte saniterats, vilket leder till att om en person skickar in JavaScript-kod i ett formulär som sedan sparar texten, kommer detta att exekveras i andra användares webbläsare och orsaka problem.

En elak användare kan få tilgång till en annan användares session och på så vis kunna logga in som den personen.

Sanitera alla texter så att tecken verkligen visas rätt och inte kan tolkas som kod.

###Öppna resurser

Data som inte är tänkt att kunna kommas åt exponeras mot personer utan behörighet.

Om en användare analyserar JavaScript-koden går det att göra egna anrop mot URLer som tar bort meddelanden då dessa inte kollar behörighet. Databasfilen går även att ladda ner publikt vilket ger tillgång till alla lösenord. 

Se till att alla resurser som anses privata och går att komma åt publikt kräver någon form av auktorisering

##Prestandaproblem

###Kombinera filer
Flera anrop mot webbserver orsakar längre laddtider[1, 31], därför bör alla Javascript/CSS-filer samlas i en fil och alla inline-script/css skall även de flyttas tillrespektive fil. 

Applikationen får klienten att efterfråga resurser som inte finns på servern, vilket leder till anrop som blir överflödiga. Referenser till obefintliga resurser bör därför tas bort.

Filer som inte används på en sida bör tas bort, t.ex. JavaScript för meddelanden på inloggningssidan eller bildfil i CSS-filen som aldrig visas upp.

###Placering av CSS
I dagsläget finns det CSS mellan header och body-taggen, dessa skall placeras inuti header-taggen.[1]

###Placering av Javascript
Javascript skall placeras i slutet på dokumentet för att inte hindra övriga HTTP-anrop, vilket leder till en upplevd snabbara laddningstid.[1] 

##Egna reflektioner
Applikationen presenterar en hel del säkerhetshål, och jag hade inte varit nöjd om jag använde applikationen med de brister som finns.
Utöver ovan påpekade problem verkar inte applikationen fungera som den ska. För administratören triggas aldrig eventet för att ta bort meddeelanden, och något som gäller samtliga är att Meddelanden efterfrågas innan sidan laddad klart i layout-filen, vilket leder till kompileringsfel.

I övrigt en intressant laboration som uppmuntrar till att tänka på säkerhet som var kul att genomföra! Tråkigt att Vagrant krånglar till det på Windows.

##Referenser
[1] Steve Sounders, O’Reilly, High Performance Web Sites. September 2007. [Online] Tillgänglig: (http://www.it.iitb.ac.in/frg/wiki/images/4/44/Oreilly.Seve.Suoders.High.Performance.Web.Sites.Sep.2007.pdf). [Hämtad: 3 december 2015]

[2] "The Ten Most Critical Web Application Security Risks" Open Web Application Security Project, 12 Juni 2013. [Online] Tillgänglig: (https://www.owasp.org/index.php/Top10#OWASP_Top_10_for_2013). [Hämtad: 3 december 2015]
