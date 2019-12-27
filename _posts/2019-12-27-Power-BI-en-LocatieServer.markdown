---
layout: post
title: "Power BI en LocatieServer"
date:   2019-12-27
categories: 
---

In deze blog post laat ik zien hoe je adressen in Power BI kunt standaardiseren en verrijken met behulp van [LocatieServer](https://www.pdok.nl/restful-api/-/article/pdok-locatieserver) van [Publieke Dienstverlening Op de Kaart (PDOK)](https://www.pdok.nl). PDOK is een centrale voorziening voor het ontsluiten van geografische gegevens van de overheid. LocatieServer is een [API](https://nl.wikipedia.org/wiki/Application_programming_interface) op een deel van deze gegevens. Met LocatieServer kun je bijvoorbeeld gegevens opvragen van adressen, kadastrale percelen, wegen, buurten, wijken en hectometerpaaltjes.

De adressen van LocatieServer komen uit de Basisregistratie Adressen en Gebouwen (BAG). De BAG bevat de officiële schrijfwijze van adressen. Ieder adres heeft een uniek identificatienummer en coördinaten. Handig voor verdere verwerking!

Met LocatieServer kun je zoeken op adressen, ook als er geen exacte match is. Hartstikke handig! LocatieServer geeft je de mogelijkheid om adressen te standaardiseren op de BAG en te verrijken met andere gegevens. In [deze tutorial](https://github.com/FrieseWoudloper/web-api-workshop-gelderland/wiki/PDOK-LocatieServer) staat beschreven hoe LocatieServer precies werkt.

In het voorbeeld maak ik in de eerste stap een kleine dataset aan met twaalf adressen van provinciehuizen. De adressen zijn onvolledig of de schrijfwijze is niet goed. Ik laat vervolgens zien hoe je met LocatieServer het volledige, officiële adres op kunt halen, en het adres ook nog eens kunt verrijken met buurt, wijk en coördinaten.

Check eerst de landinstellingen voor importeren via _Bestand_ &rarr; _Options and settings_ &rarr; _Options_ &rarr; _CURRENT FILE_ &rarr; _Regional Settings_. De juiste instelling is _English (United States)_ anders staat het decimaal scheidingsteken voor de coördinaten straks op de verkeerde plek.

![]({{site.url}}/assets/img/2019-12-27/img04.png) 

Voer de adressen in.

![]({{site.url}}/assets/img/2019-12-27/img01.png) 

Selecteer de tabel. Kies voor _Edit query_ en daarna voor _Advanced Editor_. Voeg extra stappen toe voor het versturen van een API request naar LocatieServer en het verwerken van de response. Gebruik [deze link](https://gist.githubusercontent.com/FrieseWoudloper/68de3449287c9ed43ac4f140d3991e4c/raw/9be21d051e2645ee581d1b9567410b2e49b3c4b5/provinciehuis.txt) om de code te kopiëren.
       
![]({{site.url}}/assets/img/2019-12-27/img02.png) 

Klik op _Done_ en sluit de _Advanced Editor_.

Sluit het _Query Editor Window_ en voer de wijzigingen door (_Close & Apply_).

Het resultaat is een tabel met voor ieder provinciehuis het officiële, volledige adres, inclusief buurt- en wijknummer, en coördinaten in lengte- en breedtegraden. De code kan eenvoudig uitgebreid worden, zodat je voor ieder adres ook de gemeente, het waterschap en het kadastraal perceel krijgt.
&nbsp;  
&nbsp;
![]({{site.url}}/assets/img/2019-12-27/img03.png) 
&nbsp;         

Met behulp van de coördinaten kun je bijvoorbeeld een kaartje maken.

<iframe width="800" height="400" src="https://app.powerbi.com/view?r=eyJrIjoiNjgwZTk4Y2EtNGMwYi00N2E1LThhZjMtNTI1MzEyMDEyMmIxIiwidCI6ImZkZjllNmE2LWY5NGYtNDE1Zi04NjIzLTk0YWNiYzU5OWU1NCIsImMiOjh9" frameborder="0" allowFullScreen="true"></iframe>
&nbsp;        
&nbsp;      
Kortom: Je kunt LocatieServer gebruiken om de datakwaliteit van adressen in Power BI te verbeteren. Bedenk wel dat LocatieServer alleen werkt voor adressen in Nederland en dat je het aantal requests wel ietwat beperkt moet houden, anders loop je het risico dat je IP adres geblokkeerd wordt.