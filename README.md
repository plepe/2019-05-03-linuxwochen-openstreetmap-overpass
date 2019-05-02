<!-- footer: CC-BY-SA 4.0 https://github.com/plepe/2019-03-20-presentation-overpass -->
# Overpass API - die Datenbank zur OpenStreetMap
## Mit der Overpass API die Datenvielfalt aus der OpenStreetMap kitzeln

Linuxwochen Wien, 3. Mai 2019 - plepe.at/356

Stephan Bösch-Plepelits - skunk@openstreetmap.at - plepe.at

Social media:
- Twitter: twitter.com/plepe
- Mastodon: @plepe@en.osm.town
- Jabber: skunk@jabber.at

---
<!-- page_number: true -->

## Struktur
- OpenStreetMap
- Overpass API
- Anwendungsbeispiele

---

## OpenStreetMap - was ist das?
- https://www.openstreetmap.org/
- Eine von der Community erstellte Karte der Welt<br>(c) OpenStreetMap-Mitwirkende
- Open Data: ODbL (Rohdaten) bzw. CC-BY-SA 2.0 (Kacheln)
- Schirmherrschaft: OpenStreetMap Foundation

---

## OpenStreetMap - wie funktioniert das?
3 verschiedene Objekttypen:
- Node: ID; Tags; Lat/Lon Koordinaten
- Way: ID; Tags; Liste von Nodes
- Relation: ID; Tags; Liste von Nodes, Ways und Relationen (mit Rolle)

Tags sind Liste von Key/Value Paaren, z.b.:
- amenity=restaurant, cuisine=kebap;pizza, name=Kebaphaus
- highway=primary, name=Dresdner Straße, maxspeed=50, oneway=yes

https://wiki.openstreetmap.org/wiki/Map_Features

---

## Was ist die Overpass API?
<div style='float: right'>

![](400px-Overpass_API_logo.svg.png)

</div>
Eine effiziente Datenbank um Daten aus der OpenStreetMap auszulesen.

-> https://wiki.openstreetmap.org/wiki/Overpass_API

* Abfragesprache: Overpass QL
* Free Software (GNU AGPL 3.0)
* Public Server: z.B. overpass-api.de
* Output: JSON, XML, CSV

---

##  Beispiel für eine Overpass Query
```c
[out:json][bbox:48.21,16.37,48.22,16.38];
node[amenity=cafe];
out body geom;
```

```json
{ "version": 0.6,
  "generator": "Overpass API 0.7.55",
  "elements": [
    { "type": "node", "id": 319735103,
      "lat": 48.2079420, "lon": 16.3698350,
      "tags": {
        "amenity": "cafe",
        "name": "Café Hawelka",
        "name:zh": "哈維卡咖啡",
        "website": "http://www.hawelka.at",
        "wikipedia": "de:Café Hawelka"
      }
    }, ...
  ]
}
```

---

## Overpass Turbo
Frontend für Overpass API: https://overpass-turbo.eu

![Screenshot Overpass Turbo](overpass-turbo-map.png)
*© OpenStreetMap-Mitwirkende, https://osm.org/copyright*

---

## Overpass Turbo - Features

* "Wizard" für das schnelle Erstellen von Abfragen
* "Export" in verschiedene Formate: GeoJSON, GPX, KML, OSM
* "Teilen" Permalinks erstellen
* Daten-Ansicht

---

## Overpass QL (1/6)

Overpass Query Language. Jede Query besteht aus mehreren Teilen:

1. Einstellungen
2. Selektierung
3. Ausgabe

Nach einer Ausgabe kann wieder eine weitere Query folgen.

Im folgenden werden nur die wichtigsten Konstrukte erklärt, eine vollständige Anleitung gibt es auf https://wiki.openstreetmap.org/wiki/Overpass_API/Overpass_QL

---

## Overpass QL (2/6) - Einstellungen

```c
[out:json] vs. [out:xml] vs. [out:csv] // Format
[bbox:48.2,16.2,48.4,16.4] // Region (default: Welt)
[date:"2018-12-24T19:00:00Z"] // Historische Daten
[diff:] and [adiff:] // Änderungen (verwende: [out:xml])
```

---

## Overpass QL (3/6) - Selektierung
Typ:
```c
node; way; relation;
nwr /* alles */, area /* flächen um punkt */
```
Selektiere Tags:
```c
nwr[amenity=cafe]; // Tag hat Wert
nwr[amenity]; // Tag hat irgendeinen Wert
nwr[amenity~"^(bar|cafe)$"]; // Regular Expression
nwr[~"name"~"straße$"]; // Regular Expression
node(id:1234,1235); // Nach ID
```

Bedeutung der Tags: https://wiki.openstreetmap.org/wiki/DE:Map_Features

---

## Overpass QL (4/6) - Ausgabe
```c
out;
```
Detailgrad:
* `ids`: nur IDs
* `meta`: inkludiere Meta-Daten (timestamp, user, ...)

Geo-Informationen:
* `geom`: Volle Geometrie, inkl. Members
* `bb`: Nur bounding box
* `center`: Nur Centroid

---

## Overpass QL (5/6) - Weitere Konstrukte

Union:
```c
( node[amenity=cafe]; node[amenity=bar]; );
```
Members:
```c
way[highway]; out; >; out;
```
Sets:
```c
node[amenity=cafe]->.a; node.a[cuisine]; .a out;
```

---

## Overpass QL (6/6) - komplexes Beispiel

Alle Kinos die nicht mehr als 100m von einer Bus- oder Bimstation in Wien entfernt sind.

```c
area[name="Wien"];
(
  node(area)[highway=bus_stop];
  node(area)[railway=tram_stop];
);
nwr(around:100)[amenity=cinema];
out;
```

---

## Overpass API & QGIS

* Plugin QuickOSM: https://github.com/3liz/QuickOSM

![Screenshot QGIS, QuickOSM](QGisQuickOSM.png)

---

## Exportiere Daten kompatibel mit anderen Tools

Mit der folgenden Abfrage sind die Daten mit anderen OSM-Tools (z.B. JOSM, Osmosis) kompatibel.

```c
[out:xml][bbox:{{bbox}}];

way[highway]; // Or any other query

out meta;
>;
out meta;
```

---

## Abfragen mit CURL direkt in eine Datei schreiben

```sh
curl \
  -X POST \
  -d "[out:json];node[place=continent];out;" \
  https://overpass-api.de/api/interpreter \
  > export.json
```

---

## Webseiten mit eingebetteter Karte und Overpass Overlay erstellen

LeafletJS + verschiedene 

* https://github.com/plepe/overpass-layer
* https://github.com/GuillaumeAmat/leaflet-overpass-layer

---

## OpenStreetBrowser

* https://www.openstreetbrowser.org

![Screenshot OpenStreetBrowser](OpenStreetBrowser.png)

---

## VR Map

- https://vrmap.kairo.at - https://vrmap.plepe.at

![Screenshot VR Map](VRMap.png)


---

## OpenLevelUp!

- https://openlevelup.net/

![Screenshot OpenLevelUp](OpenLevelUp.png)

---

## Happy mapping!

Danke für Eure Aufmerksamkeit!

Fragen?

<hr>

### References
* Abbildungen © OpenStreetMap-Mitwirkende
* Marp (Markdown presentation editor)