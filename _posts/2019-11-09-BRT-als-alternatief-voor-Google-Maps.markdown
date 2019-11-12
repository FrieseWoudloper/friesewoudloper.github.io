---
layout: post
title:  "BRT als alternatief voor Google Maps"
date:   2019-11-09
categories: 
---

Een mooie topografische kaart maakt je website of app helemaal af. Voor het toevoegen van een kaart maken webontwikkelaars vaak gebruik van Google Maps. Dat is jammer. Er kleven namelijk nadelen aan Google Maps en er zijn goede alternatieven beschikbaar. 

_Wat is het probleem met Google Maps?_      
Het staat buiten kijf dat een ontwikkelaar met de Google Maps API snel en eenvoudig een kaart kan toevoegen aan zijn project. Maar daar staat tegenover dat het gebruik van de API niet altijd gratis is. Iedere maand krijg je 200 dollar aan API calls cadeau, maar daarna worden [kosten](https://cloud.google.com/maps-platform/pricing/sheet/) in rekening gebracht. Je bent verplicht om een creditcardnummer op te geven, ook al blijf je onder de limiet voor gratis gebruik.

Een ander nadeel is dat Google bepaalt hoe de kaart er uit ziet. Google kiest bijvoorbeeld welke bedrijven worden weggelaten, of juist extra worden benadrukt.

Voor overheidsorganisaties is er nóg een reden om geen Google Maps te gebruiken. Ze zijn namelijk wettelijk verplicht om bij de uitvoering van hun werkzaamheden de [Basisregistratie Topografie (BRT)]((https://zakelijk.kadaster.nl/brt)) te gebruiken. Dit is vastgelegd in de [Wet basisregistraties kadaster en topografie](https://wetten.overheid.nl/BWBR0021547/2010-01-01). De BRT is de officiële topografische kaart van de Nederlandse overheid. 

_Welke alternatieven zijn er voor de kaart van Google Maps?_      
De kaart van [OpenStreetMap](https://www.openstreetmap.org) is een goede optie. Binnen de overheid is de [BRT-Achtergrondkaart](https://www.pdok.nl/introductie/-/article/basisregistratie-topografie-achtergrondkaarten-brt-a-) echter de beste keuze. De kaart is gebaseerd op gegevens uit de basisregistratie. Er zijn geen kosten voor het gebruik. De BRT-Achtergrondkaart wordt beschikbaar gesteld via een [WMTS](https://en.wikipedia.org/wiki/Web_Map_Tile_Service) en [TMS](https://en.wikipedia.org/wiki/Tile_Map_Service) API. Je specificeert het coördinatenstelsel, zoomniveau, X- en Y-coördinaat in je request, en krijgt een afbeelding terug. De BRT-Achtergrondkaart is overigens open data en ook vrij beschikbaar voor (commerciële) toepassingen buiten de overheid. 

_Hoe voeg je kaartviewer-functionaliteit toe?_    
Google Maps biedt méér dan een plaatje. Het is ook een kaartviewer. Je kunt in- en uitzoomen, een legenda maken, markers aan de kaart toevoegen, enzovoort. Hoe doe je dat met de BRT-Achtergrondkaart? Daarvoor heb je een JavaScript library nodig, bijvoorbeeld [OpenLayers](http://openlayers.org) of [Leaflet](http://leaflet.org/).

<link rel="stylesheet" href="https://unpkg.com/leaflet@1.5.1/dist/leaflet.css" />
<script src="https://unpkg.com/leaflet@1.5.1/dist/leaflet.js"></script>
<div id="voorbeeld1" style="width: 600px; height: 450px;"></div>
<script>
	var map1 = L.map('voorbeeld1').setView([52.15, 5.35], 7); 

	L.tileLayer('https://geodata.nationaalgeoregister.nl/tiles/service/wmts/brtachtergrondkaart/EPSG:3857/{z}/{x}/{y}.png', {
		maxZoom: 18,
		attribution: '<a href="https://creativecommons.org/licenses/by/4.0/">CC-BY</a> Kadaster '
	}).addTo(map1);

</script> 
<br/>
Je kunt kiezen welke kaartopmaak je wilt: standaard, grijs, pastel of water. Daarnaast kun je ook een actuele luchtfoto als ondergrond toevoegen.

<div id="voorbeeld2" style="width: 600px; height: 450px;"></div>
<script>
	var options = {maxZoom: 18, attribution: '<a href="https://creativecommons.org/licenses/by/4.0/">CC-BY</a> Kadaster'}

	var standaard = L.tileLayer('https://geodata.nationaalgeoregister.nl/tiles/service/wmts/brtachtergrondkaart/EPSG:3857/{z}/{x}/{y}.png', options),
		grijs     = L.tileLayer('https://geodata.nationaalgeoregister.nl/tiles/service/wmts/brtachtergrondkaartgrijs/EPSG:3857/{z}/{x}/{y}.png', options),
		pastel    = L.tileLayer('https://geodata.nationaalgeoregister.nl/tiles/service/wmts/brtachtergrondkaartpastel/EPSG:3857/{z}/{x}/{y}.png', options),
		water     = L.tileLayer('https://geodata.nationaalgeoregister.nl/tiles/service/wmts/brtachtergrondkaartwater/EPSG:3857/{z}/{x}/{y}.png', options),
		luchtfoto = L.tileLayer('https://geodata.nationaalgeoregister.nl/luchtfoto/rgb/wmts/Actueel_ortho25/EPSG:3857/{z}/{x}/{y}.jpeg', options);

	var map2 = L.map('voorbeeld2', {
		center: [53.219515, 6.568813],
		zoom: 13,
		layers: [standaard]
	});

	var baseMaps = {
		"Standaard": standaard,
		"Grijs":     grijs,
		"Pastel":    pastel,
		"Water":     water,
		"Luchtfoto": luchtfoto
	};

	L.control.layers(baseMaps, null, {collapsed: false}).addTo(map2);
	
	var marker = L.marker([53.219515, 6.568813]).addTo(map2);
	marker.bindPopup("Dit is de Martinitoren").openPopup();
</script> 
<br/>

Naast de verschillende weergaven van de BRT-Achtergrondkaart is er ook nog de [OpenTopo](https://www.pdok.nl/introductie/-/article/opentopo) achtergrondkaart. OpenTopo is gebaseerd op meerdere gegevensbronnen van de overheid, waaronder de BRT, en maakt daarnaast gebruik van OpenStreetMap. De kaart is alleen beschikbaar in het [Rijksdriehoekstelsel](https://nl.wikipedia.org/wiki/Rijksdriehoeksco%C3%B6rdinaten) (EPSG:28992) en niet in de Web Mercator projectie (EPSG:3857). Hier moet je rekening mee houden in de code. 

<script src="https://cdnjs.cloudflare.com/ajax/libs/proj4js/2.5.0/proj4.js"></script>
<script src="https://cdnjs.cloudflare.com/ajax/libs/proj4leaflet/1.0.2/proj4leaflet.js"></script>
<div id="voorbeeld3" style="width: 600px; height: 450px;"></div>
<script>
	var crs = new L.Proj.CRS('EPSG:28992', '+proj=sterea +lat_0=52.15616055555555 +lon_0=5.38763888888889 +k=0.9999079 +x_0=155000 +y_0=463000 +ellps=bessel +units=m +towgs84=565.2369,50.0087,465.658,-0.406857330322398,0.350732676542563,-1.8703473836068,4.0812 +no_defs',
		{
			resolutions: [3440.640, 1720.320, 860.160, 430.080, 215.040, 107.520, 53.760, 26.880, 13.440, 6.720, 3.360, 1.680, 0.840, 0.420, 0.210],
			bounds: L.bounds([-285401.92, 22598.08], [595401.9199999999, 903401.9199999999]),
			origin: [-285401.92, 22598.08]
		}
	);
		
	var map3 = L.map('voorbeeld3', {
		crs: crs,
        zoom: 13, 
        center: [53.219515, 6.568813] 
	});
	
	new L.TileLayer('https://geodata.nationaalgeoregister.nl/tiles/service/tms/1.0.0/opentopoachtergrondkaart/EPSG:28992/{z}/{x}/{y}.png', {
		minZoom: 0,
		maxZoom: 13,
		tms: true,
		attribution: '<a href="https://creativecommons.org/licenses/by/4.0/">CC-BY</a> Bron: J.W. van Aalst, <a href="http://www.opentopo.nl">www.opentopo.nl</a>'
	}).addTo(map3);
	
	var marker = L.marker([53.219515, 6.568813]).addTo(map3);
	marker.bindPopup("Dit is de Martinitoren");
</script> 
<br/>

De code van de voorbeelden in deze post kun je downloaden via deze [link](https://gist.github.com/FrieseWoudloper/b8f68be19299e19a3239ea2ee06c5a0f).      

_Meer weten over Leaflet?_     
Volg de [web map workshop for developers](https://github.com/NieneB/webmapping_for_developers) van Niene Boeijen. Vergeet ook vooral niet om op haar te [stemmen voor de Geo Prestige Award 2019](https://response.questback.com/isa/qbv.dll/ShowQuest?QuestID=5409765&sid=Lf7iwmHaek).

_Niet zo handig met JavaScript?_
[NL Maps](https://www.nlmaps.nl) maakt het nóg makkelijker om een kaart toe te voegen aan je website. Je beantwoordt een aantal vragen, waarna NL Maps de JavaScript code automatisch voor je genereert. Kopiëren, en klaar!