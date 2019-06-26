---
layout: post
title:  "API requests afvangen met Postman"
date:   2019-06-14
categories: 
---


Welke requests doet de _ArcGIS Server Feature Service_ reader van FME Desktop 'onder de motorkap'? Dat vroeg ik mij af naar aanleiding van deze [reactie van Arno van Anrooij](https://www.linkedin.com/feed/update/urn:li:activity:6539908644769718272/?commentUrn=urn%3Ali%3Acomment%3A%28activity%3A6539908644769718272%2C6541268229413830656%29) op mijn [blog post over het Bodemloket]([deze post]({{site.url}}/2019/06/05/Hergebruik-van-bodemkwaliteitsgegevens.html)).

Het is eenvoudig om de requests af te vangen met Postman. Deze post is eigenlijk een notitie voor mijzelf, voor 'future reference'. Om het werkend te krijgen moet je drie dingen doen: de configuratie van Postman aanpassen, je lokale IP adres opvragen en een proxyserver instellen.

Postman is een handige tool voor het uitproberen en testen van REST API's. Ik gebruik de gratis versie die je [hier](https://www.getpostman.com/downloads/) kunt downloaden. Hoe je de instellingen van Postman moet aanpassen voor het afvangen van requests, is gedocumenteerd op [deze pagina](https://learning.getpostman.com/docs/postman/sending_api_requests/capturing_http_requests).

Het IP adres van een computer met Windows als besturingssyteem vraag je snel op met de [ipconfig command line utility](https://www.lifewire.com/ip-config-818377).

Het configureren van een proxy server onder Windows 10 doe je via _Instellingen_ > _Netwerk en internet_ > _Proxy_. Scroll naar beneden naar _Handmatige proxyconfiguratie_. Zet _Proxyserver gebruiken_ aan. Vul het lokale IP adres in en het nummer van de poort waarop Postman luistert. Vergeet niet als laatste stap de configuratie op te slaan.

Als je in FME Desktop nu de workspace uitvoert met de _ArcGIS Server Feature Service_ reader, verschijnen de requests in Postman de _Historie_ tab in de sidebar.
