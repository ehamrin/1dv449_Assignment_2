#Prestanda- och säkerhetsanalys av applikation
**Erik Hamrin (eh222ve)**

##Säkerhetsproblem
###SQL-injections

SQL-injections är när en användare lyckas exekvera opålitlig data i en databas-fråga och på så vis kunna påverka en applikations persistenta lagring.

På inloggningssidan går det att logga in som en användare/administratör utan att känna till dess annvändaruppgifter och på så sätt få full tillgång till applikationen.

Detta går att åstadkomma genom att skriva in valfritt användarnamn och ````1' or 1='1```` som lösenord, vilket kommer att logga in användaren oavsett om man känner till ett lösenord eller inte.

Det mest förespråkade enligt OWASP är att använda sig utav parametriserade frågor, stored procedures eller liknande som hindrar kod i parametrar som exekveras i tolken.[2] 

###Känslig data

Uppgifter som anses vara privata bör göras oläsliga när de skall lagras i en applikation, annan privat data skall ses över så att den inte kan kommas åt publikt.[2] 

Om en besökare går till /message/data (genom att analysera JavaScript-filen, eller HTTP-svaret som presenteras på inloggningssidan) visas alla meddelanden.

Om en person kommer åt databasen står alla lösenord i klartext. 

För att undvika detta bör alla lösenord hashas och  autentisering/auktorisering begäras för att se meddelanden.

###Cross-Site Scripting (XSS)

Data som presenteras för användare har inte saniterats, vilket leder till att om en person skickar in JavaScript-kod i ett formulär som sedan sparar texten, kommer detta att exekveras i andra användares webbläsare och orsaka problem.

En elak användare kan få tillgång till en annan användares session och på så vis kunna logga in som den personen.

Sanitera alla texter så att tecken verkligen visas rätt och inte kan tolkas som kod.

###Cross Site Request Forgery (CSRF)

CSRF är att en elak användare kan lyckas skicka förfalskade HTTP-anrop genom ett annat webbläsarfönster där en person är inloggad. Detta kan vara att Person A går in på en sida, som skickar ett POST-anrop till sidan http://www.test.com/message/new, då Person A är inloggad på denna sidan i ett annat fönster skickas cookies med som autentisierar Person A och ett meddelande postas som ser ut att vara skrivet av Person A.

Med dessa metoder kan personer utsättas för attacker på deras banker, sociala medier etc. och kan orsaka allt ifrån felaktiga pengaöverföringar till åsikter personen i fråga inte står för. Dessa attacker är svåra att motbevisa då allting görs ifrån den attackerades dator.

I applikationens fall kan meddelanden postas utan att personen som är inloggad skrivit det själv eller godkänt det. 

Detta kan undvikas genom att använda sig av Tokens [2, p.14], som är en teckenkombination ofta inte mindre än 25 tecken, som genereras på servern. Denna token skickas sedan med formuläret och valideras på servern. Detta motverkar CSRF genom att den elaka sidan inte kan komma åt och analysera HTML-taggar på http://www.test.com/message/new och kan då inte se den aktuella token som gäller för den användaren.

Ett annat alternativ är att ha en CAPTCHA på sidan [2, p.14], vilket tvingar användaren att bekräfta att det är en person som sitter vid sidan. Personligen så kan detta vara frusterande om det uppstår alldeles för ofta, och det kan försvåra användandet av sidan för personer med lässvårigheter [3].


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

###Placering av JavaScript
JavaScript skall placeras i slutet på dokumentet för att inte hindra övriga HTTP-anrop, vilket leder till en upplevd snabbare laddningstid.[1] 

###Minifiering av resurser
Att hämta resurser är det som tar längst tid när man i normala fall laddar en sida, en del går inte att göra så mycket åt (t.ex. att ansluta sig till servern), men ett effektivt sätt är att minska storleken på resurserna genom att ta bort kommentarer och whitespaces i filerna.

Detta bör göras på alla javascript- och css-filer som ska hämtas av användaren. 

Effektiviteten av att minifiera resurser varierar och beror på mängden kommentarer, kodstil m.m., men minskar storleken på filer med ungefär 30%.

Detta leder till att sidan laddar snabbare och användare får en trevligare upplevelse.

####Komprimering av resurser
Vidare går det även att komprimera resursfilerna genom att korta ner variabelnamn, detta minskar filstorleken ytterligare men försvårar felsökning av applikationen då det tidigare kan ha sett ut så här:

    var radius = 2;
	var pi = 3.14159265359;
    var area = pi * radius * radius;

Och efter komprimering:

	var a = 2;
	var b = 3.14159265359;
    var c = b * a * a;

###Cache-header
För att undvika onödiga HTTP-anrop till servern bör statiska resurser cachas i webbläsaren. Detta kan göras genom att skicka med "max-age"-headern som talar om i hur många sekunder en fil ska sparas innan webbläsaren bör kolla efter en uppdatering.

De resurser som sällan ändras (javscript, css, logotyper m.m.) kan ha en väldigt lång max-age på 6 månader. I dessa fall kan det vara rekomenderat att ha med versionsnummer i filnamnet, alternativt i en query string:

`stylesheet-1.12.css` alternativt `stylesheet.css?version=1.12`

När versionsnumret byts ut kommer det att betraktas som en ny fil och laddas om.

##Egna reflektioner
Applikationen presenterar en hel del säkerhetshål, och jag hade inte varit nöjd om jag använde applikationen med de brister som finns.
Utöver ovan påpekade problem verkar inte applikationen fungera som den ska. För administratören triggas aldrig eventet för att ta bort meddelanden, och något som gäller samtliga är att Meddelanden efterfrågas innan sidan laddad klart i layout-filen, vilket leder till kompileringsfel.

I övrigt en intressant laboration som uppmuntrar till att tänka på säkerhet som var kul att genomföra! Tråkigt att Vagrant krånglar till det på Windows.

##Referenser
[1] Steve Sounders, O’Reilly, High Performance Web Sites. September 2007. [Online] Tillgänglig: (http://www.it.iitb.ac.in/frg/wiki/images/4/44/Oreilly.Seve.Suoders.High.Performance.Web.Sites.Sep.2007.pdf). [Hämtad: 3 december 2015]

[2] "The Ten Most Critical Web Application Security Risks" Open Web Application Security Project, 12 Juni 2013. [Online] Tillgänglig: (https://www.owasp.org/index.php/Top10#OWASP_Top_10_for_2013). [Hämtad: 3 december 2015]

[3] Abigail Marshall, "Don’t lock us out!", 10 Juni 2012. [Online] Tillgänglig: (http://blog.dyslexia.com/dont-lock-us-out/). [Hämtad: 3 januari 2016]
