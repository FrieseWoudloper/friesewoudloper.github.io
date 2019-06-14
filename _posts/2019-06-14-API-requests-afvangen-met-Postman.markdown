---
layout: post
title:  "API requests afvangen met Postman"
date:   2019-06-14
categories: 
---


Welke requests doet de _ArcGIS Server Feature Service_ reader van FME Desktop 'onder de motorkap'? Dat vroeg ik mij af naar aanleiding van deze [reactie van Arno van Anrooij]() op mijn blog post over het Bodemloket.

Het is heel eenvoudig om de requests af te vangen met Postman. In deze post leg ik vast hoe dat moet voor 'future reference'. Stap 1 is het configureren van Postman en stap 2 het instellen van de proxyserver op je lokale machine.

Postman is een handige tool voor het uitproberen en testen van REST API's. Ik gebruik de gratis versie die je [hier](https://www.getpostman.com/downloads/) kunt downloaden.

Hoe je Postman moet configureren voor het afvangen van requests, is beschreven op [deze pagina](https://learning.getpostman.com/docs/postman/sending_api_requests/capturing_http_requests).

Het IP adres van een computer met Windows als besturingssyteem vraag je snel op met de [ipconfig command line utility](https://www.lifewire.com/ip-config-818377).

Het configureren van een proxy server onder Windows 10 doe je via _Instellingen_ > _Netwerk en internet_ > _Proxy_. Scroll naar beneden naar _Handmatige proxyconfiguratie_. Zet _Proxyserver gebruiken_ aan. Vul het lokale IP adres in en het nummer van de poort waarop Postman luistert. Vergeet niet als laatste stap de configuratie op te slaan.

Als je in FME Desktop nu de workspace uitvoert met de _ArcGIS Server Feature Service_ reader, verschijnen de requests in de _Historie_ tab in de sidebar.
