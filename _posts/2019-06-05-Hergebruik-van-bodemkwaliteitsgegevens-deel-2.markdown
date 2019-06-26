---
layout: post
title:  "Hergebruik van bodemkwaliteitsgegevens - deel 2"
date:   2019-06-05
categories: 
---


Vorige maand schreef ik een blog over hoe je de gegevens uit het Bodemloket kunt downloaden voor hergebruik in eigen toepassingen. [Ivor Bosloper](https://twitter.com/_ivor/status/1135440800563826688) en [Arno van Anrooij](https://www.linkedin.com/feed/update/urn:li:activity:6539908644769718272?commentUrn=urn%3Ali%3Acomment%3A%28activity%3A6539908644769718272%2C6541268229413830656%29) kwamen met suggesties voor verbetering. Het resultaat is een veel eenvoudiger en sneller downloadscript.    

Wat is het probleem?        
De gegevens in het Bodemloket worden ontsloten via de ArcGIS REST API. De dataset bestaat uit meer dan 500.000 rijen. De API retourneert er echter niet meer dan 1.000 per request. Hoe kun je dan toch alle gegevens opvragen? Het antwoord ligt voor de hand: door meerdere requests te doen.    

De meest eenvoudige manier om dit te doen, is door gebruik te maken van de query parameters `resultOffset` en `resultRecordCount`. Dat lukt in dit geval echter niet, omdat de service niet geconfigureerd is voor het ondersteunen van _response paging_.

Een andere optie is om rijen te filteren met behulp van een `where` query parameter. Dat gaat met deze service ook niet altijd goed. Requests met een filter retourneren regelmatig foutcode 400 (_Unable to complete request_).   

Wat werkt dan wél?    
De truc is om een filter op te geven met daarin expliciet alle identificatienummers van de rijen die je wilt opvragen. Dat doe je als volgt.      
    
Iedere rij heeft een uniek identificatienummer. Dat is een eis die de ArcGIS software stelt. Voor de dataset met _WBB locaties _ is dat het attribuut `WBB_DOSSIER_DBK`. Dat kun je checken in de <a href="https://www.gdngeoservices.nl/arcgis/rest/services/blk/lks_blk_rd/MapServer/1">documentatie</a>: het attribuut heeft als is namelijk van het type `esriFieldTypeOID`.    

Je kunt voor alle rijen met één request het unieke identificatienummer opvragen met behulp van de query parameter `returnIdsOnly=True`. De beperking dat de service maximaal 1.000 rijen per request retourneert, is nu niet van toepassing. Dit request retourneert meer dan 500.000 identificatienummers:   

<code>
<a href="https://www.gdngeoservices.nl/arcgis/rest/services/blk/lks_blk_rd/MapServer/1/query?where=1=1&returnIdsOnly=True&f=json">https://www.gdngeoservices.nl/arcgis/rest/services/blk/lks_blk_rd/MapServer/1/query?where=1=1&returnIdsOnly=True&f=json</a>
</code>

Daarna doe je een POST (geen GET!) request, waarbij je de `objectIds` parameter toevoegt aan de request body. Als parameter waarde geef je de eerste 1.000 identificatienummers op gescheiden door een komma.

Endpoint en query parameters:     

`
https://www.gdngeoservices.nl/arcgis/rest/services/blk/lks_blk_rd/MapServer/1/query?where=1=1&outFields=*&f=geojson
`

Body:    

`
  objectIds=18005606,18005621,18005632, ... , 2C17911893,17911904
`

Zo kun je batchgewijs alle rijen opvragen. 

In FME Desktop kun je de gegevens uit het Bodemloket inlezen met behulp van de _Esri ArcGIS Server Feature Service_ reader. Deze reader doet 'onder de motorkap' de requests zoals hierboven beschreven.

Als je meer wilt weten over de mogelijkheden van de ArcGIS REST API, raadpleeg dan de [documentatie](ArcGIS REST API) of volg het eerste deeel van de workshop [Web API's voor beginners](https://github.com/FrieseWoudloper/web-api-workshop/wiki).