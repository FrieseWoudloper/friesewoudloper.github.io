---
layout: post
title:  "GeoJSON, D3 en de draairichting van polygonen"
date:   2019-05-16 
categories: 
---
Voor een blog post wil ik een simpel kaartje met Groninger gemeenten maken. Als oefening probeer ik dat met de [D3 JavaScript library](https://en.wikipedia.org/wiki/D3.js) te doen. Daarbij loop ik tegen een vreemd probleem aan.


# Gemeentegrenzen van PDOK
De gemeentegrenzen haal ik uit de [bestuurlijke grenzen service](https://www.pdok.nl/introductie/-/article/bestuurlijke-grenzen) van PDOK. Ik gebruik de Web Feature Service (WFS). Het service request pas ik aan, zodat de service alleen Groninger gemeenten retourneert in een formaat dat eenvoudig verwerkt kan worden door D3. Dat formaat is [GeoJSON](https://geojson.org/) met geometrie in het [WGS84](https://nl.wikipedia.org/wiki/WGS_84) coördinatenstelsel.
 
Service endpoint: 
```
https://geodata.nationaalgeoregister.nl/bestuurlijkegrenzen/wfs
```    
Query parameters:
```
service=WFS
version=2.0.0
request=GetFeature
typeName=bestuurlijkegrenzen:gemeenten
cql_filter=gemeentenaam IN ('Het Hogeland', 'Loppersum', 'Appingedam', 'Westerkwartier', 'Groningen', 'Delfzijl', 'Midden-Groningen', 'Oldambt', 'Veendam', 'Pekela', 'Westerwolde', 'Stadskanaal')
outputFormat=JSON
srsName=EPSG:4326
```
Het request - waarbij HTML encoding is toegepast - is dan:
```
https://geodata.nationaalgeoregister.nl/bestuurlijkegrenzen/wfs?service=WFS&request=GetFeature&typeName=bestuurlijkegrenzen:gemeenten&cql_filter=gemeentenaam%20IN%20(%27Het%20Hogeland%27,%20%27Loppersum%27,%20%27Appingedam%27,%20%27Westerkwartier%27,%20%27Groningen%27,%20%27Delfzijl%27,%20%27Midden-Groningen%27,%20%27Oldambt%27,%20%27Veendam%27,%20%27Pekela%27,%20%27Westerwolde%27,%20%27Stadskanaal%27)&outputFormat=JSON&srsName=EPSG:4326
```
Bekijk [hier](https://geodata.nationaalgeoregister.nl/bestuurlijkegrenzen/wfs?service=WFS&request=GetFeature&typeName=bestuurlijkegrenzen:gemeenten&cql_filter=gemeentenaam%20IN%20(%27Het%20Hogeland%27,%20%27Loppersum%27,%20%27Appingedam%27,%20%27Westerkwartier%27,%20%27Groningen%27,%20%27Delfzijl%27,%20%27Midden-Groningen%27,%20%27Oldambt%27,%20%27Veendam%27,%20%27Pekela%27,%20%27Westerwolde%27,%20%27Stadskanaal%27)&outputFormat=JSON&srsName=EPSG:4326) de response.

De [code](https://gist.github.com/FrieseWoudloper/0ff8a3e6824c427554ec603d34861a04#file-index-html) voor het visualiseren van de response in een kaart met D3 is eenvoudig. Het resultaat is echter niet wat ik had verwacht, namelijk [een zwart vierkant](https://bl.ocks.org/FrieseWoudloper/0ff8a3e6824c427554ec603d34861a04).

# Gemeentegrenzen van ESRI
ESRI stelt in de [Levende Atlas van Nederland](http://livingatlas-dcdev.opendata.arcgis.com/datasets/1b3840e5009742ae8b5698eafa438035_0) dezelfde gemeentegrenzen als PDOK beschikbaar. Ze worden ontsloten via de ArcGIS REST API.

Service endpoint:
```
https://services.arcgis.com/nSZVuSZjHpEZZbRo/arcgis/rest/services/Bestuurlijke_Grenzen_Gemeenten_2019/FeatureServer/0/query 
```
Query parameters:
```
where=Gemeentenaam IN ('Het Hogeland', 'Loppersum', 'Appingedam', 'Westerkwartier', 'Groningen', 'Delfzijl', 'Midden-Groningen', 'Oldambt', 'Veendam', 'Pekela', 'Westerwolde', 'Stadskanaal')
outFields=*
outSR=4326
f=geojson
```
Request:
```
https://services.arcgis.com/nSZVuSZjHpEZZbRo/arcgis/rest/services/Bestuurlijke_Grenzen_Gemeenten_2019/FeatureServer/0/query?where=Gemeentenaam%20IN%20(%27Het%20Hogeland%27,%20%27Loppersum%27,%20%27Appingedam%27,%20%27Westerkwartier%27,%20%27Groningen%27,%20%27Delfzijl%27,%20%27Midden-Groningen%27,%20%27Oldambt%27,%20%27Veendam%27,%20%27Pekela%27,%20%27Westerwolde%27,%20%27Stadskanaal%27)&outFields=*&outSR=4326&f=geojson
```
Bekijk [hier](https://services.arcgis.com/nSZVuSZjHpEZZbRo/arcgis/rest/services/Bestuurlijke_Grenzen_Gemeenten_2019/FeatureServer/0/query?where=Gemeentenaam%20IN%20(%27Het%20Hogeland%27,%20%27Loppersum%27,%20%27Appingedam%27,%20%27Westerkwartier%27,%20%27Groningen%27,%20%27Delfzijl%27,%20%27Midden-Groningen%27,%20%27Oldambt%27,%20%27Veendam%27,%20%27Pekela%27,%20%27Westerwolde%27,%20%27Stadskanaal%27)&outFields=*&outSR=4326&f=geojson) de response, die net iets anders is dan bij PDOK.

Wanneer ik dezelfde [code](https://gist.github.com/FrieseWoudloper/534072d43823d039498a7fa7f830510e#file-index-html) als eerder gebruik, maar nu met de GeoJSON response van ESRI, krijgen ik wél [een mooi kaartje](https://bl.ocks.org/FrieseWoudloper/534072d43823d039498a7fa7f830510e).

# Draairichting: left- of right-hand rule

Waarom werkt het niet met het GeoJSON bestand van PDOK en wél met dat van ESRI?     

Het antwoord is dat PDOK de _right-hand rule_ volgt voor de draairichting van polygonen. Bij de right-hand rule loopt de buitenste ring van een polygoon tegen de klok in, en de binnenste (de gaten) met de klok mee. Bij de _left-hand rule_ is dat precies andersom. ESRI - en ook D3 - houden zich aan de left-hand rule. 

D3 interpreteert de gemeentegrenzen uit de PDOK response dus als gaten. Het visualiseert de _inverse_ polygoon. Die omvat de rest van de wereld en vult het hele (vierkante) canvas. Bekijk ter illustratie van het belang van de draairichting in D3 ook [deze voorbeelden](https://bl.ocks.org/mbostock/a7bdfeb041e850799a8d3dce4d8c50c8) van Mike Bostock.

Het probleem met de PDOK polygonen is eenvoudig op te lossen, door de draairichting om te keren. Dit kan bijvoorbeeld met [Turf](https://turfjs.org/) JavaScript library:
```
var fixed = mapdata.features.map(function(f) {
    return turf.rewind(f, {reverse:true});
});
```	
Met de [aangepaste code](https://gist.github.com/FrieseWoudloper/db271923645400786c1696fd80024bb0#file-index-html) lukt het vervolgens wél om de GeoJSON van PDOK te visualiseren in D3:
<style>
    path {
        fill: #ccc;
        stroke: #fff;
        stroke-width: .5px;
    }
    path:hover {
        fill: orange;
    }
    div.tooltip { 
        position: absolute;     
        text-align: center;     
        width: 80px;          
        height: 14px;         
        padding: 2px;       
        font: 12px sans-serif;    
        background: #fff; 
        border: 0px;        
        pointer-events: none;     
    }
</style>
<script src="https://d3js.org/d3.v4.min.js"></script>
<script src="https://cdnjs.cloudflare.com/ajax/libs/Turf.js/5.1.5/turf.min.js"></script>
<div class="graph"></div>
<script>
    var width = 700, height = 400;
    var svg = d3.select(".graph").append("svg")
                                 .attr("viewBox", "0 0 " + (width) + " " + (height))
                                 .style("max-width", "700px");
    var url = "https://geodata.nationaalgeoregister.nl/bestuurlijkegrenzen/wfs?service=wfs&request=GetFeature&typeName=bestuurlijkegrenzen:gemeenten&cql_filter=gemeentenaam%20IN%20(%27Het%20Hogeland%27,%20%27Loppersum%27,%20%27Appingedam%27,%20%27Westerkwartier%27,%20%27Groningen%27,%20%27Delfzijl%27,%20%27Midden-Groningen%27,%20%27Oldambt%27,%20%27Veendam%27,%20%27Pekela%27,%20%27Westerwolde%27,%20%27Stadskanaal%27)&outputFormat=json&srsName=EPSG:4326"; 
    var tooltip = d3.select("body")
	                .append("div") 
                    .attr("class", "tooltip")       
                    .style("opacity", 0);
    d3.json(url, function(error, mapdata) {
        var fixed = mapdata.features.map(function(f) {
            return turf.rewind(f, {reverse:true}); 
        });
        var projection = d3.geoMercator();
        var path = d3.geoPath().projection(projection);
        projection.fitSize([width, height], {"type": "FeatureCollection", "features": fixed});
		console.log(fixed);
        svg.append("g")
           .attr("class", "gemeente")
           .selectAll("path")
           .data(fixed)
           .enter()
           .append("path")
           .attr("d", path)
		   .on("mouseover", function(d) {    
               tooltip.transition()    
                      .duration(200)    
                      .style("opacity", .9);    
               tooltip.html(d.properties.gemeentenaam)  
                      .style("left", (d3.event.pageX) + "px")   
                      .style("top", (d3.event.pageY - 28) + "px");  
           })          
           .on("mouseout", function(d) {   
               tooltip.transition()    
                      .duration(500)    
                      .style("opacity", 0); 
           });
		   
    });
</script>

# GeoJSON standaardspecificatie

GeoJSON is niet ontstaan vanuit een organisatie als de Internet Engineering Task Force (IETF) of het Open Geospatial Consortium (OGC). Het is een initiatief van een aantal slimme mensen, op zoek naar een eenvoudig formaat voor het uitwisselen van geo-informatie. Dit leidde tot de GeoJSON 2008 specificatie, waarin geen afspraken werden opgenomen over de draairichting van polygonen.

Pas in 2016 heeft de IETF de standaardspecificatie voor het GeoJSON formaat vastgelegd in [RFC 7946](https://tools.ietf.org/html/rfc7946). Daarin worden wél eisen gesteld aan de draairichting:

>A linear ring MUST follow the right-hand rule with respect to the area it bounds, i.e., exterior rings are counterclockwise, and holes are clockwise.

Het komt er op neer dat PDOK zich houdt aan de RFC 7946 specificatie. ESRI en D3 doen dat niet.