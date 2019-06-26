---
layout: post
title:  "Installatie van GeoPandas"
date:   2019-06-26 
categories: 
---

Deze week was ik bij [GoDataDriven](https://godatadriven.com/) in Amsterdam voor de cursus [Data Science met Python](https://training.xebia.com/data-science/data-science-with-python). Met Pandas wordt het manipuleren en analyseren van gegevens in Python een stuk eenvoudiger! Ik nam mij voor om informatievragen in de komende periode zo veel mogelijk te beantwoorden met behulp van Python. Dan beklijft de cursus beter. 

Omdat ik bij de provincie vooral werk met geografische gegevens, wilde ik [GeoPandas](http://geopandas.org/) installeren. 

>GeoPandas is an open source project to make working with geospatial data in python easier.         
>GeoPandas extends the datatypes used by pandas to allow spatial operations on geometric types.       

Het installeren van GeoPandas op mijn Windows laptop bleek nog niet zo eenvoudig te zijn. GeoPandas is afhankelijk van een aantal andere packages waaronder [GDAL](https://pypi.org/project/GDAL/), [Fiona](https://pypi.org/project/Fiona/), [pyproj](https://pypi.org/project/pyproj/), [RTree](https://pypi.org/project/Rtree/) en [Shapely](https://pypi.org/project/Shapely/). Het installeren van deze _dependencies_ was de moeilijkheid.   

Voor het installeren van GeoPandas volgde ik de stappen in het artikel [Using geopandas on Windows](https://geoffboeing.com/2014/09/using-geopandas-windows/) van Geoff Boeing.

Ik gebruik de [Anaconda](https://www.anaconda.com/distribution/) Python distributie. Daarom probeerde ik eerst om GeoPandas te installeren met de conda package manager:

```
conda install -c conda-forge geopandas
```

Ik kreeg de volgende waarschuwing en besloot de installatie af te breken:    
_WARNING: The conda.compat module is deprecated and will be removed in a future release._

Daarna volgde ik de instructies onder _Installing geopandas and its dependencies manually_. Helaas had dit ook niet het gewenste resultaat. Dit testte ik in Jupyter met de volgende code:

```
from osgeo import gdal, ogr, osr         
from fiona.ogrext import Iterator, ItemsIterator, KeysIterator         
from geopandas import GeoDataFrame         
gdal.VersionInfo()       
```
 
 GDAL (versie 3.0.0) was goed geïnstalleerd, Fiona en GeoPandas niet. Het importeren van Fiona gaf namelijk een foutmelding: _ImportError: DLL load failed: The specified module could not be found_.      
Fiona is afhankelijk van GDAL, en GeoPandas weer van Fiona. Als Fiona niet goed geïnstalleerd is, werkt GeoPandas niet.

Het is uiteindelijk tóch gelukt op GeoPandas te installeren. Dit zijn de stappen die daarvoor nodig waren:    
1. Deïnstalleren van Anaconda.
2. Installeren van de nieuwste versie van Anaconda.
4. Opstarten van Anaconda Powershell Prompt met beheerrechten (_Run as Administrator_).
5. Uitvoeren van `conda update conda`.
6. Uitvoeren van `conda install -c conda-forge geopandas`. 

Als ik nu in Jupyter dezelfde code uitvoer als eerder, krijg ik geen foutmeldingen. 

![]({{site.url}}/assets/img/2019-06-26/img01.png) 

Het versienummer van GDAL is nu 2.0.3. Blijkbaar installeert conda een oudere versie van GDAL. Misschien is de hogere versie van GDAL in de [wheel van Gohlke](https://www.lfd.uci.edu/~gohlke/pythonlibs/#gdal) het probleem? 

Hoe dan ook, ik kan aan de slag!