---
layout: post
title: "CBS buurten visualiseren met Mapbox en Power BI"
date:   2019-12-07
categories: 
---

In deze post laat ik zien hoe je CBS buurtgegevens met de Mapbox Visual kunt tonen op een kaart in Power BI.

Ga in de browser naar het [Nationaal Georegister](https://www.nationaalgeoregister.nl/). Zoek op [CBS gebiedsindelingen](https://www.nationaalgeoregister.nl/geonetwork/srv/dut/catalog.search#/metadata/effe1ab0-073d-437c-af13-df5c5e07d6cd). Open het tabblad _Downloads, views en links_. Noteer de URL van de [WFS](https://nl.wikipedia.org/wiki/Web_Feature_Service) service en de naam van de kaartlaag (het _feature type_) met de actuele buurtindeling.
![]({{site.url}}/assets/img/2019-12-07/img01.png) 

Voor het downloaden en converteren van de buurten gebruiken we [cURL](https://nl.wikipedia.org/wiki/CURL). Check eerst of deze command line utility al beschikbaar is op jouw systeem. [Installeer cURL](https://curl.haxx.se/) als dit niet het geval is.

Open een opdrachtprompt en voer het volgende commando uit:

`
curl "https://geodata.nationaalgeoregister.nl/cbsgebiedsindelingen/wfs?request=GetFeature&service=WFS&version=1.1.0&typeName=cbsgebiedsindelingen:cbs_buurt_2019_gegeneraliseerd&outputFormat=application/json&srsName=EPSG:4326&propertyName=geom,statcode" -o E:\\Temp\\buurten.geojson
`

Je kunt het resultaat ook via [deze link]({{site.url}}/assets/misc/2019-12-07/buurten.geojson) downloaden.

Merk op dat de _base URL_ van het service request is overgenomen uit het Nationaal Georegister. Hetzelfde geldt voor de waarde van de query parameter _typeName_. Het outputbestand is in het GeoJSON formaat. Pas indien nodig het pad van de output directory aan. De geometrie is geconverteerd naar het [World Geodetic System 84 (EPSG:4326)](https://nl.wikipedia.org/wiki/WGS_84). Mapbox kan namelijk niet overweg met het het standaard coördinatenstelsel van de service. Dat is het [Rijksdriehoekstelsel, ofwel EPSG:28992](https://nl.wikipedia.org/wiki/Rijksdriehoeksco%C3%B6rdinaten).

Maak [een gratis account Mapbox account](https://account.mapbox.com/) aan. Ga naar de [Account pagina](https://account.mapbox.com/) en kopieer je Mapbox access token naar het klembord.

![]({{site.url}}/assets/img/2019-12-07/img03.png) 

Ga naar de [Tilesets](https://studio.mapbox.com/tilesets/) pagina. Klik op _New tileset_ en upload het `buurten.geojson` bestand.

![]({{site.url}}/assets/img/2019-12-07/img04.png) 

Er wordt nu een nieuwe tileset toegevoegd met de buurten. Dit kan wel een aantal minuten duren. 

![]({{site.url}}/assets/img/2019-12-07/img05.png) 

Let op: Mapbox genereert de naam van de tileset automatisch, die van jou heeft een andere naam dan buurten-809yxk.

Vraag detailinformatie op door op de naam van de nieuwe tileset te klikken. In onderstaande schermafdruk zie je een preview van de tileset. Dat is niet altijd zo. Soms krijg je alleen maar een zwart vlak. Desondanks werkt de tileset wel in Power BI.

![]({{site.url}}/assets/img/2019-12-07/img06.png) 

Noteer het _Tileset ID_, de _Layer Name_ en de _Property Name_ van het veld met de buurtcodes.

Merk op dat de tileset een _zoom extent_ heeft: De data zijn níet zichtbaar onder zoom level 8 en boven 14.

Ga in de browser naar de pagina van het CBS over [Kerncijfers wijken en buurten 2019](https://www.cbs.nl/nl-nl/maatwerk/2019/31/kerncijfers-wijken-en-buurten-2019). Download het XLS bestand. XLS is een verouderd formaat. Bij het laden van de gegevens in Power BI krijg ik een foutmelding.

![]({{site.url}}/assets/img/2019-12-07/img07.png) 

Open het XLS bestand in Excel. Sla het op als XLSX bestand. Laad het XLSX bestand in Power BI. Selecteer alleen rijen met buurtgegevens. Dat zijn rijen waarvoor geldt: `recs = buurt`.

![]({{site.url}}/assets/img/2019-12-07/img08.png)

Het doel is om een choropleet te maken met de bevolkinsdichtheid per buurt. Daarvoor gebruiken we de waarde in de kolom `bev_dich`. Deze kolom bevat echter een tekst, geen numerieke waarde. Dit komt doordat in het CBS bestand missende waarden gerepresenteerd worden door een punt. Maak een nieuwe kolom `bevolkingsdichtheid` aan met een numerieke waarde. Zorg er voor dat velden met missende waarden leeg zijn.

![]({{site.url}}/assets/img/2019-12-07/img21.png)

Ga naar het _Visualisaties_ panel. Klik op de optie _Een aangepast visueel element importeren_. Dat is het pictogram met de drie puntjes. Selecteer _Importeren vanuit de marketplace_.

![]({{site.url}}/assets/img/2019-12-07/img09.png)

Zoek op _Mapbox_. Klik op de knop _Toevoegen_ van de Mapbox Visual.

![]({{site.url}}/assets/img/2019-12-07/img10.png)

De Mapbox Visual is toegevoegd aan het _Visualisaties_ panel.

![]({{site.url}}/assets/img/2019-12-07/img11.png)

Als ik nu de Mapbox Visual toevoeg aan mijn rapport, krijg ik een foutmelding:

![]({{site.url}}/assets/img/2019-12-07/img12.png)

De workaround is om éérst een andere visual toe te voegen - bijvoorbeeld een tabel - en daarna pas de Mapbox Visual. Vervolgens kun je zonder problemen de eerste visual weer verwijderen. Raar maar waar.

Selecteer de Mapbox Visual in je rapport. Pas eventueel de grootte en positie aan. 

Ga naar het tabblad _Velden_ in het _Visualisaties_ panel. Sleep de kolom `gwb_code` naar _Location_ en `bevolkingsdichtheid` naar _Color_. De kolommen bevatten respectievelijk de buurtcode en het aantal huishoudens.

![]({{site.url}}/assets/img/2019-12-07/img13.png)

Ga naar het tabblad _Velden_. Vul onder het kopje _Viz Settings_ je Mapbox acces token in.

![]({{site.url}}/assets/img/2019-12-07/img14.png)

Zet de _Circle_ optie uit en de _Choropleth_ optie aan.

![]({{site.url}}/assets/img/2019-12-07/img15.png)

Selecteer onder het kopje _Choropleth_ voor _Data Level 1_ de waarde _Custom Tileset_. Vul voor _Vector Tile Url Level 1_, _Source Layer Name Level 1_ en _Vector Property Level 1_ respectievelijk de volgende waarden in: `mapbox://` gevolgd door het Tileset ID, de Layer Name en Property Name die je hebt overgenomen van de website van Mapbox.

![]({{site.url}}/assets/img/2019-12-07/img16.png)

De Mapbox Visual toont nu een kaart van Nederland. Op de kaart staan nog geen buurten. Dit komt door het zoomniveau. Zoom verder in en de buurten verschijnen.

Pas de kleuren en de zoomlevels aan.

![]({{site.url}}/assets/img/2019-12-07/img17.png)

Verander eventueel de doorzichtheid (_opacity_) aan.

![]({{site.url}}/assets/img/2019-12-07/img22.png)

Pas de titel aan.

![]({{site.url}}/assets/img/2019-12-07/img18.png)

Ga naar het tabblad _Velden_ en sleep de kolom `bevolkingsdichtheid` naar _Tooltips_. 

![]({{site.url}}/assets/img/2019-12-07/img19.png) 

Voilà! 

![]({{site.url}}/assets/img/2019-12-07/img20.png) 

Via [deze link](https://app.powerbi.com/view?r=eyJrIjoiMjhkNjdiYmMtMDNhYy00ODIyLTk2MzItNjVmYzBlY2ViNDE4IiwidCI6ImZkZjllNmE2LWY5NGYtNDE1Zi04NjIzLTk0YWNiYzU5OWU1NCIsImMiOjh9) kun je het resultaat bekijken. Vergeet niet dat je eerst in moet zoomen, voordat je iets te zien krijgt.
