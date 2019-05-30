---
layout: post
title:  "Hergebruik van bodemkwaliteitsgegevens"
date:   2019-05-30
categories: 
---
Follow The Money (FTM) heeft dit voorjaar een nieuw dossier geopend: Gore Grond in je Gemeente. Op 11&nbsp;april deed dit platform voor onderzoeksjournalistiek een oproep: [Bodemvervuiling in jouw gemeente? Zet het op de kaart](https://www.ftm.nl/artikelen/kaart-gore-grond?utm_medium=social&utm_campaign=sharebuttonleden). Volgens FTM zijn overheden niet altijd even transparant over bodemverontreiniging. Informatie is moeilijk te vinden en lastig te interpreteren. Dat geldt met name voor de spoedlocaties. Dat zijn plaatsen waar de vervuiling zo erg is, dat er direct maatregelen genomen moeten worden.

De overheid stelt informatie over de bodemkwaliteit beschikbaar via het [Bodemloket](https://www.bodemloket.nl/kaart). Na het lezen van de oproep vroeg ik me af of de gegevens in het Bodemloket geschikt zijn voor hergebruik door FTM. Ik ging op onderzoek uit.

Een [zoekopdracht op 'Bodemloket' in het Nationaal Georegister](http://www.nationaalgeoregister.nl/geonetwork/srv/dut/catalog.search#/search?facet.q=orgName%2FRijkswaterstaat&isChild='false'&resultType=details&fast=index&_content_type=json&from=1&to=20&sortBy=relevance&any_OR__title=bodemloket) levert vijf datasets op. Jammer genoeg zijn het allemaal [Web Map Services (WMS)](https://nl.wikipedia.org/wiki/Web_Map_Service). Dat betekent dat de services niet gebruikt kunnen worden om de ruwe data op te vragen.

Terug naar de kaart in het Bodemloket. Bij het laden van de pagina zie je onder water - bijvoorbeeld met Chrome DevTools - dat de bodemkwaliteitgegevens worden opgevraagd bij een webservice:

![]({{site.url}}/assets/img/2019-05-30/img01.png) 

Het gaat om het volgende endpoint:

<code>
https://www.gdngeoservices.nl/arcgis/services/blk/lks_blk_rd/MapServer/WMSServer
</code>

Helaas is dit ook een WMS. Toch geeft de URL een belangrijk stukje informatie: de service wordt gepubliceerd met [ArcGIS Server](). Je kunt nu de ArcGIS REST Services Directory opvragen door de URL aan te passen:

<code>
<a href="https://www.gdngeoservices.nl/arcgis/rest/services">https://www.gdngeoservices.nl/arcgis/rest/services</a>
</code>

Via deze mappenstructuur kun je bij de REST API van het Bodemloket:

<code>
<a href="https://www.gdngeoservices.nl/arcgis/rest/services/blk/lks_blk_rd/MapServer">https://www.gdngeoservices.nl/arcgis/rest/services/blk/lks_blk_rd/MapServer</a>
</code>

Met de REST API kun je vier datasets (_layers_) opvragen. `WBB_punten` en `WBB_locaties` bevatten respectievelijk punten en vlakken van Wet bodembescherming (WBB)
locaties. Het volgnummer van de layer is belangrijk om het endpoint van een request te bepalen. WBB_locaties is bijvoorbeeld de tweede layer en heeft volgnummer 1.

Het is nu relatief eenvoudig om locaties als GeoJSON bestand op te vragen:

<code>
<a href="https://www.gdngeoservices.nl/arcgis/rest/services/blk/lks_blk_rd/MapServer/1/query?where=1=1&outFields=*&f=geojson">https://www.gdngeoservices.nl/arcgis/rest/services/blk/lks_blk_rd/MapServer/1/query?where=1=1&outFields=*&f=geojson</a>
</code>

De response bevat maar 1.000 locaties. Het zijn er in werkelijkheid [véél meer](https://www.gdngeoservices.nl/arcgis/rest/services/blk/lks_blk_rd/MapServer/1/query?where=1=1&returnCountOnly=true&f=json). Het is niet mogelijk om meer dan 1.000 locaties per request op te vragen. Hoe kun je dan toch alle locaties opvragen?

Helaas is response paging uitgezet in de configuratie van de service. Een alternatief is om zelf meerdere requests te genereren waarmee in stappen alle locaties worden opgevraagd. Dit kan op twee manieren:
* in de `where` query parameter filteren op een attribuut dat een uniek identificatienummer bevat of
* met de query parameters `geometryType`, `inSR`, `geometry` en `spatialRel` filteren op gebied. 

De eerste optie is in dit geval ongeschikt. Alleen `WBB_DOSSIER_DBK` lijkt in aanmerking te komen als uniek identificatienummer. De [minimum waarde is 17892511](https://www.gdngeoservices.nl/arcgis/rest/services/blk/lks_blk_rd/MapServer/1/query?where=1=1&outFields=WBB_DOSSIER_DBK&orderByFields=WBB_DOSSIER_DBK&returnGeometry=false&f=json) en de [maximum waarde 118107966](https://www.gdngeoservices.nl/arcgis/rest/services/blk/lks_blk_rd/MapServer/1/query?where=1=1&outFields=WBB_DOSSIER_DBK&orderByFields=WBB_DOSSIER_DBK+DESC&returnGeometry=false&f=json). Dat betekent dat er meer dan 100.000 requests nodig zijn om alle locaties op te vragen! Bovendien krijg ik bij zo'n request ook regelmatig foutcode 400 (_Unable to complete operation_).

Bij de tweede optie verdeel je het interessegebied - bijvoorbeeld een provincie of heel Nederland - in een grid. Per tegel binnen het grid doe je een request. Het probleem is dat je de afmetingen van een tegel zo moet kiezen, dat er niet meer dan 1.000 locaties binnen een tegel liggen.

Zo langzamerhand wordt het best ingewikkeld! Daarom heb ik in [FME Desktop](https://www.safe.com/fme/fme-desktop/) een script gemaakt voor het genereren van de requests en samenvoegen van de responses. Het script is te [downloaden](https://github.com/FrieseWoudloper/FME_workspaces/blob/master/bodemloket) van GitHub.

Uit de gegevens die de REST API retourneert, kun je voor iedere locatie ook de URL van het rapport met de voortgang van het bodemonderzoek afleiden. In dit rapport staat méér informatie dan in de API response, bijvoorbeeld de locatienaam, onderzoeksrapporten en besluiten. Helaas is deze informatie niet direct op te vragen bij de API in een gestructureerd formaat. Je moet de gegevens zelf scrapen als je ze voor een analyse wilt gebruiken.

De conclusie is dat FTM gelijk heeft: het is inderdaad niet makkelijk is om de ruwe bodemkwaliteitsgegevens op te vragen, want: 
* de ArcGIS REST API is niet opgenomen in het Nationaal Georegister en daardoor moeilijk vindbaar,
* response paging is uitgezet,
* niet alle (gestructureerde) gegevens worden rechstreeks ontsloten via de REST API.

Daarnaast zijn er nog semantische problemen. De betekenis van attributen is niet meteen duidelijk. En hoe filter je eigenlijk alle spoedlocaties uit het Bodemloket? 

Kortom: er is nog veel ruimte voor verbetering als het gaat om het ontsluiten van bodemkwaliteitgegevens voor hergebruik door andere partijen dan de overheid.










