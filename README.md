# standaardgrids
De standaardgrids bestaan uit een regelmatige geografisch gerefereerde rasterindelingen van het (Rotterdamse) gebied. De grids maken het mogelijk om gegevens te aggregeren naar gelijkvormige gebieden van gelijke grootte, en om gegevensaggregaties onderling te vergelijken.

Zie [metadatacatalogus](https://datarotterdam.dataplatform.nl/dataset/standaardgrids) voor meer informatie.

## toelichting

Vaak worden gegevens geaggregeerd naar ongelijkvormige gebieden van ongelijke grootte. Bijvoorbeeld naar wijk-, blok, of postcodeniveau.
Een dergelijke indeling is voor statistische bewerkingen minder goed geschikt omdat de vorm en grootte van een gebied een directe invloed heeft op de geaggregeerde waardes.

Het CBS biedt onder meer statistieken aan die zijn geaggregeerd naar vierkanten van 100x100 meter en 500x500 meter. Zie
<https://www.cbs.nl/nl-nl/dossier/nederland-regionaal/geografische%20data/kaart-van-100-meter-bij-100-meter-met-statistieken>.
Die levering bestaat alleen uit vierkanten die waardes bevatten.

SO|Ruimte & Wonen biedt vierkanten (zonder statistiche waardes) aan die het gehele grondgebied van Rotterdam bedekken en dezelfde identificatie hebben als de vierkanten van het CBS. Dat maakt het mogelijk om gegevens te combineren met die van het CBS.

Een vierkant heeft een eenvoudige definitie; dat maakt dat (ruimtelijk) rekenen met vierkanten een relatief simpel proces is.
Vierkanten zijn minder geschikt om clusteringen en terreinvormen weer te geven. Dat komt omdat de onderlinge afstand tussen (de middelpunten van) de vierkantige cellen van de richting af hangt. De afstand naar naburige horizontaal en verticaal gelegen cellen is korter dan de afstand naar diagonaal gelegen naburige cellen.

Daarom biedt R&W ook hexagonale cellen aan. De onderlinge afstand tussen naburige cellen is altijd gelijk waardoor (onder meer) clusteringen beter worden weergegeven.

## productformaten

De producten zijn in verschillende formaten beschikbaar.

### bevragen (webservices) gemeente Rotterdam

|formaat                | URL |
| --------------------- | --- | 
|OGC:WFS                | <https://dservices.arcgis.com/zP1tGdLpGvt2qNJ6/arcgis/services/standaardgrids/WFSServer?service=wfs&request=getcapabilities> |
|ArcGIS feature server  | <https://services.arcgis.com/zP1tGdLpGvt2qNJ6/arcgis/rest/services/standaardgrids/FeatureServer> |

### bestandsleveringen gemeente Rotterdam

| formaat               | locatie |
| --------------------- | ------- |
| OGC:Geopackage        | `K:\Geo_Data\Kennisloods\\Data\Standaardgrids\standaardgrids.gpkg`|
| ESRI File Geodatabase | `K:\Geo_Data\Kennisloods\\Data\Standaardgrids\standaardgrids.gdb` |

### leveringen CBS

Het CBS biedt vierkantstatistieken met statistische inhoud aan in verschillende bestandsformaten en services:

* CBS Vierkantstatistieken 100m: <https://data.overheid.nl/data/dataset/cbs-vierkantstatistieken-100m>
* CBS Vierkantstatistieken 500m: <https://data.overheid.nl/data/dataset/cbs-vierkantstatistieken-500m>

## bronnen

* Grid: gegenereerd.
* Afbakening: bestuurlijke gemeentegrens uit de BRK, via PDOK [link](http://www.nationaalgeoregister.nl/geonetwork/srv/dut/catalog.search#/metadata/35359958-c40a-486f-9cf5-567a94de905e).

## bewerking
### hexagonale cellen

1. aanmaken puntvormig raster met
  * opp = gewenste oppervlakte (hier: 10000 m2)
  * afstand in X-richting: `deltaX = 3 * sqrt ( (2 * opp) / (3 * sqrt(3)) )`
  * afstand in Y-richting: `deltaY = 2 * sqrt ( opp / (2 * sqrt(3)))`
2. Aanmaken puntvormig raster met de zelfde afstanden als in 1), en ten opzichte daarvan verschoven met:
  * verschuiving in X-richting: `shiftX = deltaX / 2`
  * verschuiving in Y-richting: `shiftY = deltaY / 2`
3. Berekenen van de hexagonale cellen als de voronoidiagrammen van de rasters uit 1) en 2)
4. Toekennen van een identificatie per cel:
  * `celID := "N"||<Northing>||"E"||<Easting>`, 
  * waarbij Northing en Easting de X-, resp. Y-ordinaat in RD zijn van het middelpunt van de gridcel. 
  * De coordinaten zijn afgerond naar meters.
5. Clippen op de gemeentegrens

### CBSVierkanten

1. Aanmaken vierkant grid met 
  * de gewenste vierkantgrootte
  * de coordinaten op veelvouden van de gewenste vierkantgrootte
2. Toekennen van identificaties conform CBS:
  * 100x100-grid: `<kolomnaam> := "CRS28992RES100M"`
  * 500x500-grid: `<kolomnaam> := "CRS28992RES500M"`
  * `<kolomnaam> := "N"||<Northing>||"E"||<Easting>`,
    * waarbij Northing en Easting de X-, resp. Y-ordinaat in RD zijn van de zuidwestelijke hoek van de gridcel;
    * waarbij de coordinaten zijn afgerond naar 100 meters.
3. Clippen op de gemeentegrens

## entiteiten en kenmerken
### entiteiten

Er zijn verschillende standaardgrids beschikbaar:

| entiteit | betekenis |
| -------- | --------- |
| `CBSVierkanten100m` | Vierkantig grid volgens de definitie van het CBS met een celgootte van 100 x 100 m. |
| `CBSVierkanten500m` |	Vierkantig grid volgens de definitie van het CBS met een celgootte van 500 x 500 m. |
| `Hexagons_1ha`      | Hexagonaal grid met celgroottes van 1 ha.                                           |

### generieke kenmerken

Alle entiteiten kennen de volgende kenmerken:

| kenmerk | type | voorbeeld | betekenis |
| ------- | ---- | --------- | --------- |
| **`UUID`**           | `char(36)` | `"6d64c4c3-5955-4da5-aaae-fb3aa5886834"`| Unieke, betekenisloze identificatie. |
| **`peildatum`**      | `smallint` | `2017`                                  | De datum waarop het gegeven actueel was. |
| **`ind_actueel`**    | `boolean`  | `1`                                     | Indicatie of dit gegeven voor het gebied de meest actueel bekende waarde is. |
| **`oppervlakte_m2`** | `integer`  | `6359`                                  | Totale oppervlakte van de cel in m2. |

De waarde in 'oppervlakte_m2' is niet per definitie 250000 (500x500-grid) of 10000 (100x100-grid en 1ha-hexagons). De cellen worden op de gemeentegrens geclipped en zijn daar dus kleiner. De grootte van de hexagonale cellen kan afwijken door afrondingsverschillen bij de constructie van de cellen. 

### specifieke kenmerken per entiteit

Voor de specifieke entiteiten gelden de volgende kenmerken:

#### CBSVierkanten100m

| kenmerk | type | voorbeeld | betekenis |
| ------- | ---- | --------- | --------- |
| **`CRS28992RES100M`** | `char(10)` | `"N4370E884"` | Betekenisvolle identificatie van de cel zoals door CBS gehanteerd. |

De waarde in `CRS28992RES100M` geeft de Y- en X-coördinaat (N, resp. E) van de zuidwestelijke hoek van de cel in hectometer t.o.v. RD weer. 

 
####  CBSVierkanten500m

| kenmerk | type | voorbeeld | betekenis |
| ------- | ---- | --------- | --------- |
| **`CRS28992RES500M`** | `char(10)` | `"N4370E890"` | Betekenisvolle identificatie van de cel zoals door CBS gehanteerd. |

De waarde in `CRS28992RES500M` geeft de Y- en X-coördinaat (N, resp. E) van de zuidwestelijke hoek van de cel in hectometer t.o.v. RD weer. 

 
#### Hexagons_1ha

| kenmerk | type | voorbeeld | betekenis |
| ------- | ---- | --------- | --------- |
| **`celID`** | `char(14)` | `"N437082E88350"` | Betekenisvolle identificatie van de cel. |

De waarde in `celID` geeft de Y- en X-coördinaat (N, resp. E) van het middelpunt van de cel in meters t.o.v. RD weer.
