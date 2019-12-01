---
layout: post
title:  "Arrondissementsgebieden in Power BI"
date:   2019-12-01
categories: 
---

In Power BI Desktop is het mogelijk om gebieden te tonen op een kaart. Hier kun je bijvoorbeeld de _Shape-kaart_ visual voor gebruiken. Shape-kaart herkent [provincienamen](https://docs.microsoft.com/nl-nl/power-bi/visuals/desktop-shape-map#netherlands-provinces) en koppelt deze automatisch aan de juiste geometrie. Helaas geldt dit niet voor andere gebiedsindelingen. Hoe kun je die dan in een Power BI dashboard tonen? In deze post laat ik zien hoe je een kaart kunt maken van arrondissementsgebieden in Nederland.

Geef eerst via _Bestand_ &rarr; _Opties en instellingen_ &rarr; _Opties_ &rarr; _Preview-functies_ aan dat je de Shape-kaart visual wilt uitproberen.

![]({{site.url}}/assets/img/2019-12-01/img01.png) 

Lees de gegevens in. Zorg er voor dat je een kolom toevoegt waarmee iedere rij straks gekoppeld kan worden aan een vlak. In het voorbeeld is dat de code van het arrondissement. 

![]({{site.url}}/assets/img/2019-12-01/img02.png) 

Ga vervolgens naar het Nationaal Georegister. Zoek op [CBS gebiedsindelingen](https://www.nationaalgeoregister.nl/geonetwork/srv/dut/catalog.search#/metadata/effe1ab0-073d-437c-af13-df5c5e07d6cd). Open het tabblad _Downloads, views en links_. Selecteer onder _Web Feature Service (WFS)_ de laag _cbs_arrondissementsgebied_2019_gegeneraliseerd_ en het formaat _application/json_. Download de data.

![]({{site.url}}/assets/img/2019-12-01/img03.png) 

Het [downloadbestand](https://geodata.nationaalgeoregister.nl/cbsgebiedsindelingen/wfs?request=GetFeature&service=WFS&version=1.1.0&typeName=cbsgebiedsindelingen:cbs_arrondissementsgebied_2019_gegeneraliseerd&outputFormat=application/json) bevat geometrie in het Rijksdriehoekstelsel. 

![]({{site.url}}/assets/img/2019-12-01/img04.png) 

Power BI kan echter alleen overweg met lengte- en breedtegraden. Voeg `&srsName=EPSG:4326` toe aan de URL en [download](https://geodata.nationaalgeoregister.nl/cbsgebiedsindelingen/wfs?request=GetFeature&service=WFS&version=1.1.0&typeName=cbsgebiedsindelingen:cbs_arrondissementsgebied_2019_gegeneraliseerd&outputFormat=application/json&srsName=EPSG:4326) de gegevens opnieuw. De geometrie is nu in het juiste coördinatenstelsel.

![]({{site.url}}/assets/img/2019-12-01/img05.png) 

Bewaar het bestand. Het formaat is [GeoJSON](https://nl.wikipedia.org/wiki/GeoJSON). Jammer genoeg accepteert de Shape-kaart visual alleen [TopoJSON](https://nl.wikipedia.org/wiki/TopoJSON). Verander de extensie van het bestand in .JSON en zet het daarna met [Mapshaper.org](https://www.mapshaper.org) om naar TopoJSON. 

![]({{site.url}}/assets/img/2019-12-01/img06.png) 

Voeg in Power BI de Shape-kaart visual toe aan het rapport. Sleep de kolom met de arrondissementscodes naar _locatie_.

![]({{site.url}}/assets/img/2019-12-01/img07.png) 

Standaard probeert Power BI de arrondissementscodes te koppelen aan staten in de VS. Dat gaat natuurlijk niet lukken! Ga naar het tabblad _Indelingen_ &rarr; _Shape_ &rarr; _Toewijzing toevoegen_ en selecteer het TopoJSON bestand.

![]({{site.url}}/assets/img/2019-12-01/img08.png) 

Dat ziet er al een stuk beter uit! Ga terug naar het tabblad _Velden_ en sleep de kolom met fake data naar _kleurverzadiging_. 

![]({{site.url}}/assets/img/2019-12-01/img09.png) 

Pas de visual nog verder aan. Verander bijvoorbeeld de titel en kies een ander kleurverloop.

![]({{site.url}}/assets/img/2019-12-01/img10.png) 

Een andere interessante optie is _Automatisch inzoomen_.

![]({{site.url}}/assets/img/2019-12-01/img11.png) 

Klaar!

Als je veel gebieden hebt, bijvoorbeeld bij buurten, dan is de performance van de Shape-kaart erg slecht. In een volgende post ga ik onderzoeken of in dat geval de _Map Box Visual_ uitkomst biedt.