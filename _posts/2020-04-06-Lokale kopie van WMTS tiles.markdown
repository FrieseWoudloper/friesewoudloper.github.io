---
layout: post
title: "Lokale kopie van WMTS tiles"
date:   2020-04-06
categories: 
---

Als je gegevens op een kaart in een web applicatie toont, is het ter referentie vaak handig om een topografische ondergrond toe te voegen. Hiervoor kun je bijvoorbeeld de BRT Achtergrondkaart van PDOK gebruiken. De BRT Achtergrond kaart is online te op te vragen via een Web Map Tile Service (WMTS).

Maar wat moet je doen als de web applicatie - bijvoorbeeld om security redenen - geen toegang tot het internet heeft? Daar liep ik tegenaan bij het maken van een [Shiny app](https://shiny.rstudio.com/).

Dan maak je een lokale kopie. Daarvoor kun je de [Mobile Atlas Creator (MOBAC)](https://mobac.sourceforge.io/) gebruiken.

Het is eenvoudig om MOBAC te installeren en te gebruiken. In het README bestand van de applicatie is duidelijk beschreven hoe dat moet. 

Belangrijk is om _Osmdroid ZIP_ als formaat te kiezen. Voor het downloaden van BRT Achtergrond tiles moet je bovendien een _custom map source_ definiëren. Dat is niet moeilijk. Maak een XML bestand met de volgende inhoud aan en plaats het in de map _mapsources_. 

```
<?xml version="1.0" encoding="UTF-8"?>
<customMapSource>
	<name>BRT standaard</name>
	<minZoom>5</minZoom>
	<maxZoom>18</maxZoom>
	<tileType>png</tileType>
	<tileUpdate>None</tileUpdate>
	<url>http://geodata.nationaalgeoregister.nl/tiles/service/wmts/brtachtergrondkaart/EPSG:3857/{$z}/{$x}/{$y}.png</url>
	<backgroundColor>#000000</backgroundColor>
</customMapSource>
```

Na een herstart van MOBAC is _BRT standaard_ toegevoegd als bron. Fluitje van een cent!

Waar je wel op moet letten is de _fair use policy_ van PDOK. Het is niet netjes om tienduizenden requests af te vuren op de server. Beperk daarom het geselecteerde gebied en/of de zoomniveaus.

Om de BRT Achtergrondkaart te gebruiken in een Shiny app, moet je het ZIP bestand uitpakken en de inhoud naar de map _www/map/tiles_ in de root van het Shiny project verplaatsen. In [deze Github repository](https://github.com/FrieseWoudloper/demo_offline_tiles) vind je de code van mijn demo-app.

