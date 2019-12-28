---
layout: post
title:  "CBS gebieden in Power BI"
date:   2019-12-01
categories: 
---

In Power BI Desktop is het mogelijk om gebieden te tonen op een kaart. Hier kun je bijvoorbeeld de _Shape-kaart_ visual voor gebruiken. Shape-kaart herkent [provincienamen](https://docs.microsoft.com/nl-nl/power-bi/visuals/desktop-shape-map#netherlands-provinces) en koppelt deze automatisch aan de juiste geometrie. Helaas werkt het niet zo voor CBS gebiedsindelingen. Hoe kun je die in een Power BI dashboard tonen? In deze post laat ik dat zien met als voorbeeld arrondissementsgebieden.

Geef eerst via _Bestand_ &rarr; _Opties en instellingen_ &rarr; _Opties_ &rarr; _Preview-functies_ aan dat je de Shape-kaart visual wilt uitproberen.

![]({{site.url}}/assets/img/2019-12-01/img01.png) 

Voer de gegevens in. Zorg er voor dat je een kolom toevoegt waarmee iedere rij straks gekoppeld kan worden aan een vlak. In het voorbeeld is dat de code van het arrondissement. 

![]({{site.url}}/assets/img/2019-12-01/img02.png) 

Ga vervolgens naar het [Nationaal Georegister](https://www.nationaalgeoregister.nl/). Zoek op [CBS gebiedsindelingen](https://www.nationaalgeoregister.nl/geonetwork/srv/dut/catalog.search#/metadata/effe1ab0-073d-437c-af13-df5c5e07d6cd). Open het tabblad _Downloads, views en links_. Selecteer onder _Web Feature Service (WFS)_ de dataset _cbs_arrondissementsgebied_2019_gegeneraliseerd_ en het formaat _application/json_. Download de data.[^1]

![]({{site.url}}/assets/img/2019-12-01/img03.png) 

Het [downloadbestand](https://geodata.nationaalgeoregister.nl/cbsgebiedsindelingen/wfs?request=GetFeature&service=WFS&version=1.1.0&typeName=cbsgebiedsindelingen:cbs_arrondissementsgebied_2019_gegeneraliseerd&outputFormat=application/json) bevat voor elk arrondissement de geometrie in het Rijksdriehoekstelsel aangevuld met vier _properties_.

![]({{site.url}}/assets/img/2019-12-01/img04.png) 

Power BI kan echter alleen overweg met lengte- en breedtegraden en voor het maken van de kaart hebben we genoeg aan de property _statcode_. Voeg `&srsName=EPSG:4326&propertyName=geom,statcode` toe aan de URL en [download](https://geodata.nationaalgeoregister.nl/cbsgebiedsindelingen/wfs?request=GetFeature&service=WFS&version=1.1.0&typeName=cbsgebiedsindelingen:cbs_arrondissementsgebied_2019_gegeneraliseerd&outputFormat=application/json&srsName=EPSG:4326&propertyName=geom,statcode) de gegevens opnieuw. De geometrie is nu in het juiste coördinatenstelsel en het bestand is zo compact mogelijk.

![]({{site.url}}/assets/img/2019-12-01/img05.png) 

Bewaar het bestand. Het formaat is [GeoJSON](https://nl.wikipedia.org/wiki/GeoJSON). Jammer genoeg accepteert de Shape-kaart visual alleen [TopoJSON](https://nl.wikipedia.org/wiki/TopoJSON). Verander de extensie van het bestand in .JSON en zet het daarna met [Mapshaper.org](https://mapshaper.org) om naar TopoJSON. 

![]({{site.url}}/assets/img/2019-12-01/img06.png) 

Voeg in Power BI de Shape-kaart visual toe aan het rapport. Sleep de kolom met de arrondissementscodes naar _Locatie_.

![]({{site.url}}/assets/img/2019-12-01/img07.png) 

Standaard probeert Power BI de arrondissementscodes te koppelen aan staten in de VS. Dat gaat natuurlijk niet lukken. Ga naar het tabblad _Indelingen_ &rarr; _Shape_ &rarr; _Toewijzing toevoegen_ en selecteer het TopoJSON bestand.

![]({{site.url}}/assets/img/2019-12-01/img08.png) 

Dat ziet er al een stuk beter uit! Ga terug naar het tabblad _Velden_ en sleep de kolom met fake data naar _kleurverzadiging_. 

![]({{site.url}}/assets/img/2019-12-01/img09.png) 

Pas de visual nog verder aan. Verander bijvoorbeeld de titel en kies een ander kleurverloop.

![]({{site.url}}/assets/img/2019-12-01/img10.png) 

Een andere interessante optie is _Automatisch inzoomen_.

![]({{site.url}}/assets/img/2019-12-01/img11.png) 

Klaar!

Het resultaat kun je bijvoorbeeld op Internet publiceren.

<iframe width="500" height="300" src="https://app.powerbi.com/view?r=eyJrIjoiZjY3YmZlMTQtOGFjYy00YjEyLThlN2UtMDJlYjc2ZWMyNGMyIiwidCI6ImZkZjllNmE2LWY5NGYtNDE1Zi04NjIzLTk0YWNiYzU5OWU1NCIsImMiOjh9" frameborder="0" allowFullScreen="true"></iframe>

Als je veel gebieden hebt, bijvoorbeeld bij buurten, dan is de performance van de Shape-kaart erg slecht. In een volgende post ga ik onderzoeken of in dat geval de _Mapbox Visual_ uitkomst biedt.


[^1]:Als je een andere dataset kies met veel objecten, kun je de gegevens beter niet in je browser, maar met [QGIS](https://www.qgis.org) downloaden. Anders loopt het downloaden vast.