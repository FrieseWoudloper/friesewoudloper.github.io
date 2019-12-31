---
layout: post
title: "Kaart in Power BI met R en Leaflet"
date:   2019-12-29
categories: 
---

In vorige blog posts heb ik laten zien hoe je met de [Shape-kaart](http://www.friesewoudloper.nl/2019/12/01/CBS-gebieden-in-Power-BI.html) en [Mapbox visuals](http://www.friesewoudloper.nl/2019/12/07/CBS-buurten-visualiseren-met-Mapbox-en-Power-BI.html) een kaart kunt maken in Power BI. De functionaliteit van zo'n kaart is beperkt. Het is bijvoorbeeld niet mogelijk om meerdere thematische lagen toe te voegen die je onafhankelijk van elkaar kunt aan- of uitzetten. Het is ook niet mogelijk om de BRT Achtergrondkaart als topografische ondergrond te gebruiken. De [BRT Achtergrondkaart](https://www.pdok.nl/introductie/-/article/basisregistratie-topografie-achtergrondkaarten-brt-a-) is de officiële topografische kaart van de overheid. Meer weten over de BRT Achtergrondkaart? Lees dan [deze post](http://www.friesewoudloper.nl/2019/11/09/BRT-als-alternatief-voor-Google-Maps.html).

Er zijn open source projecten die deze functionaliteit wél bieden, bijvoorbeeld [Leaflet](http://leaflet.org/). Leaflet is een JavaScript library. Met deze library kunnen ontwikkelaars een kaart-viewer toevoegen aan een website of app. Maar daarvoor moet je wel kunnen programmeren in JavaScript. 

R maakt het voor data-analisten een stuk makkelijker. Met het [leaflet package](https://cran.r-project.org/web/packages/leaflet/index.html) kun je vanuit R namelijk automatisch de benodigde JavaScript code automatisch genereren. Hoe dat moet, vertel ik je in deze blog post. Zet je schrap, want het is niet eenvoudig.

## Installeer R
Zorg er voor dat je [R](https://www.r-project.org/) en eventueel ook [RStudio](https://rstudio.com/products/rstudio/) geïnstalleerd hebt. RStudio is een [IDE](https://nl.wikipedia.org/wiki/Integrated_development_environment) of ontwikkelomgeving voor R. Open de R console of RStudio. Installeer leaflet en de andere R packages die nodig zijn met het volgende commando: 
```
install.packages(c("leaflet", "jsonlite", "XML", "htmlwidgets"))  
```

![]({{site.url}}/assets/img/2019-12-29/img01.png) 

Ga in Power BI naar _Bestand_ &rarr; _Opties en instellingen_ &rarr; _Opties_ &rarr; _GLOBAAL_ &rarr; _R-script_. Zorg er voor dat de verwijzing naar de installatielocatie van R klopt. Doe dit eventueel ook voor RStudio.

![]({{site.url}}/assets/img/2019-12-29/img02.png) 

## Installeer Node.js
Installeer vervolgens [Node.js](https://nodejs.org/en/). Op de Node.js command prompt en installeer de benodigde packages via het commando:
```
npm install -g powerbi-visuals-tools
```

Check of de installatie goed is verlopen door het volgende commando uit te voeren:
```
pbiviz 
``` 
Als alles goed is, ziet de uitvoer er als volgt uit:

![]({{site.url}}/assets/img/2019-12-29/img03.png) 

## Maak een custom visual aan
Maak nu vanaf de Node.js commando prompt een nieuwe custom visual voor Power BI aan met behulp van het [rhtml template](https://github.com/microsoft/PowerBI-visuals/blob/master/RVisualTutorial/CreateRHTML.md).
```
pbiviz new demo -t rhtml
```
![]({{site.url}}/assets/img/2019-12-29/img05.png) 

Node.js heeft op basis van het template een nieuwe directory aangemaakt. De directory bevat bestanden en sub directories.

Open het bestand `.\src\visual.ts` in een teksteditor. Vervang alle voorkomens van `NodeListOf` door `HTMLCollectionOf` en sla het bestand op.

Open daarna het bestand `pbiviz.json`. Vul een waarde in voor `description`, `supportUrl`, `name` en `email`. Sla het bestand op.

![]({{site.url}}/assets/img/2019-12-29/img06.png) 

In de documentatie heb ik het nergens gelezen, maar ik ga er vanuit dat het bestand `dependencies.json` ook gewijzigd moet worden. Hierin staan volgens mij de R packages die nodig zijn. In het voorbeeld maken we geen gebruik van `ggplot2` en `plotly`, maar wel van `leaflet` en `jsonlite`. Pas het bestand hierop aan.

![]({{site.url}}/assets/img/2019-12-29/img14.png) 

Je vraagt je ondertussen misschien af waarom ik mijn eigen custom visual bouw en geen gebruik maak van de R-visual in het venster _Visualisatie_ (zie onderstaande schermafdruk). De reden is dat de laatste alleen overweg kan met statische afbeeldingen als output, en niet met HTML en JavaScript.

![]({{site.url}}/assets/img/2019-12-29/img04.png) 

## Pas het R-script aan
Pas de code in het bestand `.\script.r` aan. Dit kun je doen in RStudio, maar het kan ook gewoon in een teksteditor.
```
source('./r_files/flatten_HTML.r')

############### Library Declarations ###############
libraryRequireInstall("leaflet");
libraryRequireInstall("jsonlite");
####################################################

################### Actual code ####################
brtAchtergrondkaart <- paste0("http://geodata.nationaalgeoregister.nl/",
                              "tiles/service/wmts/brtachtergrondkaart/",
                              "EPSG:3857/{z}/{x}/{y}.png");
veld <- paste0("https://gist.githubusercontent.com/FrieseWoudloper/",
               "fcf4546a064ac7c12198119086601eca/raw/",
               "37c4511d6a48cc1acee54818322278a55e26052b/",
               "groningen-veld.json");
map <- leaflet(data = Values);
map <- addTiles(map, 
                urlTemplate = brtAchtergrondkaart, 
                attribution = "Kaartgegevens &copy; Kadaster", 
                options = tileOptions(minZoom = 6, maxZoom = 18));		
map <- addTopoJSON(map, 
                   topojson = read_json(veld), 
                   weight = 1, color = "grey", 
                   fillOpacity = 0.3, 
                   group = 'Groningen veld');
map <- addCircles(map, 
                  lng = ~ longitude, 
                  lat = ~ latitude, 
                  popup = ~ paste("Locatie:", 
                  locatie, "<br>", "Magnitude:", 
                  magnitude));	
map <- addLayersControl(map, 
                        overlayGroups = c("Groningen veld"),
                        options = layersControlOptions(collapsed = FALSE));
map <- hideGroup(map, "Groningen veld");
map <- fitBounds(map, 
                 lng1 = max(Values$longitude), lat1 = max(Values$latitude),
                 lng2 = min(Values$longitude), lat2 = min(Values$latitude));
####################################################

############# Create and save widget ###############
internalSaveWidget(map, 'out.html');
####################################################
```
De R code genereert een Leaflet kaart met bevingen met als topografische ondergrond de BRT Achtergrondkaart. De kaart bevat ook een laag met het Groninger veld. Deze laag staat standaard uit, maar kan door de gebruiker aangezet worden. De geometrie van het Groninger veld wordt geïmporteerd vanuit een [TopoJSON](https://nl.wikipedia.org/wiki/TopoJSON) bestand. 

## Maak een package voor de custom visual
Ga terug naar de Node.js commando prompt en verander met het `cd` commando de werkdirectory naar de map met de custom visual.
```
cd demo
``` 
Compileer het package.
```
pbiviz package
```
Als het compileren succesvol is verlopen, staat het package nu klaar in de subdirectory `.\dist`.

![]({{site.url}}/assets/img/2019-12-29/img12.png) 

## Laad de custom visual in Power BI
Start Power BI Desktop. Check de landinstellingen voor importeren via _Bestand_ &rarr; _Options and settings_ &rarr; _Options_ &rarr; _CURRENT FILE_ &rarr; _Regional Settings_. De juiste instelling is _English (United States)_ anders staat het decimaal scheidingsteken voor de coördinaten straks op de verkeerde plek.

![]({{site.url}}/assets/img/2019-12-29/img07.png) 

Importeer het [CSV-bestand met bevingen](https://gist.githubusercontent.com/FrieseWoudloper/fcf4546a064ac7c12198119086601eca/raw/37c4511d6a48cc1acee54818322278a55e26052b/bevingen.csv).[^1] Voeg een tabel toe aan het rapport met het veld `locatie`.

Ga naar het deelvenster _Visualisaties_. Klik op de optie _Een aangepast visueel element importeren_. Dat is het pictogram met de drie puntjes. Selecteer _Importeren vanuit bestand_.

![]({{site.url}}/assets/img/2019-12-29/img08.png) 

Importeer het package met de custom visual. Dat is het bestand met de extensie `pbiviz` in de subdirectory `.\dist`. 

De custom visual is nu toegevoegd aan het deelvenster _Visualisaties_.

![]({{site.url}}/assets/img/2019-12-29/img09.png) 

Voeg de custom visual toe aan het rapport. Klik op _Inschakelen_ wanneer je melding krijgt over inschakelen van visuele scriptelementen.

![]({{site.url}}/assets/img/2019-12-29/img10.png) 

Voeg de waarden `latitude`, `locatie`, `longitude` en `magnitude` toe aan de custom visual. Het duurt even, maar dan krijg je een bevingenkaart te zien. 

![]({{site.url}}/assets/img/2019-12-29/img11.png) 

Via de toggle knop rechtsboven kun je de laag met het Groningen veld aanzetten. 

![]({{site.url}}/assets/img/2019-12-29/img13.png) 

Wanneer je op een plaatsnaam klikt in de tabel, zoomt de kaart in op de betreffende locatie. Voor meer interactiviteit kun je bijvoorbeeld een staafdiagram met de frequentie per magnitude toevoegen. Als je dan op een staaf klikt, wordt de kaart automatisch aangepast zodat die alleen bevingen toont van de bijbehorende magnitude.

De code van de demo kun je [hier downloaden](https://github.com/FrieseWoudloper/power-bi-r-leaflet-demo).

## Bevindingen

Het werkt, maar het is niet bepaald lean-and-mean. R wordt in dit geval alleen gebruikt voor het genereren van de JavaScript code. R voegt verder niets toe, alleen complexiteit. 

Een ander probleem is dat iedere keer wanneer de gebruiker een andere dataselectie maakt, de kaart helemaal opnieuw gerenderd wordt. Het duurt dan even voordat de kaart is aangepast. Dat is irritant.

De interactiviteit werkt ook maar één kant op. Het is niet mogelijk om een dataselectie te maken in de kaart die doorwerkt in het hele rapport. Wanneer je op een beving in de kaart klikt, verschijnt er een tooltip, maar de andere visuele elementen in het rapport reageren er niet op.

Het is ook niet gelukt om het rapport met de Leaflet kaart te delen via de [Power BI-service](https://app.powerbi.com).

Kortom:      
Met Leaflet kun je een rijke, interactieve kaart maken in Power BI. Het lijkt echter een beter idee om een custom visual te ontwikkelen die rechtstreeks gebruikt maakt van de Leaflet JavaScript library, zonder tussenkomst van R. Daar zijn op GitHub en in de Marketplace ook wel voorbeelden van, bijvoorbeeld [Icon Map](https://appsource.microsoft.com/en-us/product/power-bi-visuals/WA104381497?tab=Overview), alleen bieden die niet dezelfde functionaliteit als die ik in deze post heb gedemonstreerd. 

## Bronnen
* [Documentation for creating visuals for Power BI](https://github.com/microsoft/PowerBI-visuals)
* [Tutorial: Developing a Power BI visual](https://docs.microsoft.com/en-us/power-bi/developer/visuals/custom-visual-develop-tutorial)
* [Use R-powered Power BI visuals in Power BI](https://docs.microsoft.com/en-us/power-bi/desktop-r-powered-custom-visuals)
* [How to generate custom visuals in Power BI using R](https://towardsdatascience.com/custom-html-visuals-in-power-bi-using-r-2b0494894ff)
* [Power BI Leaflet Custom Visual](https://github.com/avinmathew/power-bi-leaflet-custom-visual)
* [Leaflet based visualization for PowerBI](https://github.com/woodbuffalo/powerbi-leaflet)

[^1]:De gegevens over de bevingen zijn afkomstig van het KNMI, die over het Groningen veld van de NAM. Voor bevingen en gaswinning is de distributie gebruikt van [Bevinggevoeld.nl](http://www.bevinggevoeld.nl).
