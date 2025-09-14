---
layout: post
title: "Use OpenStreetMap web API"
date: 2011-06-14
categories: [geomatics, how-to]
tags: [api, geocoding, mapquest, nominatim, openstreetmap, osm, rest, reverse-geocoding, web-service, xapi, xml]
excerpt: "A comprehensive guide to using OpenStreetMap APIs for geocoding and reverse geocoding, with practical examples using MapQuest's Open Initiative."
---

OpenStreetMap (i.e. OSM) is a community which creates and provides free and open geographic data. The project started because most available maps, such as Google Maps or Bing, have legal or technical restrictions on their use: lot of content are not exposed due to copyrights, belonging to mapping companies (e.g. NAVTEQ or Tele Atlas) or national agencies (e.g. Ordnance Survey or IGN). To see how rich is OpenStreetMap data, you can have a preview via the [ito! map project](http://www.ito-world.org/).

The main feature of OpenStreetMap is to provide a map, as Google do. You can access it on [there main site](https://www.openstreetmap.org/) or [embed](https://wiki.openstreetmap.org/wiki/Deploying_your_own_Slippy_Map) it in your own website. But you can also access OSM data in other ways than viewing them in a browser…

## OpenStreetMap available API

Indeed OpenStreetMap provides several API to create, modify and read geographic content. In addition, some API provide advanced querying features (e.g. geocoding or reverse geocoding features).

All the available API are listed below:

* **[API](https://wiki.openstreetmap.org/wiki/API)**: OpenStreetMap editing API for fetching and saving raw geodata from/to the OpenStreetMap database.
* **[Xapi](https://wiki.openstreetmap.org/wiki/Xapi)**: OpenStreetMap extended read-only API, based on a modified version of the standard API, that provides enhanced querying capabilities, as bouding-box or X-path.
* **[Nominatim](https://wiki.openstreetmap.org/wiki/Nominatim)**: OpenStreetMap search engine, for searching OSM data by name and address and generating synthetic addresses of returned OSM points (i.e. reverse geocoding).

Xapi and Nominatim are what will interest us today. These API are used as one of the sources for the Search box on the [OpenStreetMap home page](https://www.openstreetmap.org/), but also powers other websites like [MapQuest](https://www.mapquest.com/).

Because of official OpenStreetMap servers are mostly overloaded and out of date, their use are not recommended. Fortunately, other providers are available (you can see the additional link section for more information). We will focus on one of the most relevant providers: the MapQuest Open Initiative.

[MapQuest](https://www.mapquest.com/) is a mapping company based in the United States and wholly-owned by AOL. In 2010, The company announced their support for OpenStreetMap, making MapQuest the first large online mapping service to embrace OSM. They since earmarked $1 million in resources to help improve the OSM data in the United States, with the stated intention of possibly using OSM data for their maps of the U.S. in the future.

The [MapQuest Open Initiative](https://developer.mapquest.com/) is one of the results of their support efforts. In addition to localized websites, it provides an API to read and query OSM data. These data are frequently updated from OSM (every 15 minutes for map data, every 5 minutes for search data and daily for routing data). So there is no problem to get fresh data quickly!

## Using OpenStreetMap API

### The data model

In order to use API via MapQuest, we first have to understand the data model used by OpenStreetMap. OSM data is stored as XML. Basically, it is a list of instances of 3 data primitives:

* **[Node](https://wiki.openstreetmap.org/wiki/Node)**, a single geospatial point.
* **[Way](https://wiki.openstreetmap.org/wiki/Way)**, an ordered interconnection of at least 2 nodes that describe a linear feature (e.g. street, footpath, railway line, river, …).
* **[Relation](https://wiki.openstreetmap.org/wiki/Relation)**, used to group way or node objects that are geographically related (i.e. connected or adjacent to one another).

To summarize, querying the API should return all or part of the following XML schema:

```xml
<?xml version='1.0' standalone='no'?>
<osm version='0.6' generator='xapi: OSM Extended API'
 xmlns:xapi='http://www.informationfreeway.org/xapi/0.6'
 xapi:uri='/api/0.6/*[amenity=hotel]'
 xapi:planetDate='200803150826'
 xapi:copyright='2008 OpenStreetMap contributors'
 xapi:instance='zappy2'>
  <node id='218963' lat='52.5611324692581' lon='-1.79024812573334' timestamp='2006-03-22T16:47:48+00:00' version='1' changeset='2211'>
  </node>
  <node id='331193' lat='53.7091237972264' lon='-1.50282510180841' timestamp='2007-03-31T00:09:22+01:00' version='1' changeset='2211'>
    <tag k='amenity' v='hotel'/>
  </node>
  ...
  <way id='4958218' timestamp='2007-07-25T01:55:35+01:00' version='1' changeset='2211'>
    <nd ref='218963'/>
    <nd ref='331193'/>
    ...
    <tag k='amenity' v='hotel'/>
    <tag k='building' v='hotel'/>
  </way>
  <relation id='123456' timestamp='2007-10-25T03:05:34Z' version='32' changeset='2211'>
    <member type='node' ref='331193' role=''/>
    <member type='node' ref='331194' role=''/>
    ...
    <tag k='amenity' v='hotel'/>
    <tag k='operator' v='Premier Inns'/>
    <tag k='type' v='operators'/>
  </relation>
</osm>
```

### Querying the API

The previous XML result can be obtained by querying Xapi. The querying engine allows the use of filters also called predicates. There are 3 types of predicates:

* **Tag** predicates, to match any node that has a key with the wanted value. For instance, use the link `api/0.6/node[amenity=hospital]` to get all nodes in which tag `amenity` exists and equals `hospital`
* **BBox** (i.e. bounding box) predicates, which limit the result to the selected bounding box region. For instance, the link `api/0.6/node[bbox=-6,50,2,61]` will return all objects localized in the region limited by the `[left=-6°, bottom=50°, right=2°, top=61°]` box.
* **Child element** predicates, to select objects matching X-path like conditions. For instance, the link `api/0.6/way/14310041` will return the `way` object matching `14310041` id.

However, there is limitations on the use of predicates. Currently each request is limited to one tag predicate and one bbox predicate. Child element predicates are used to query a specific OSM object with a known ID.

### A simple example

To illustrate the use of our API, imagine that we want to know the [highway type](https://wiki.openstreetmap.org/wiki/Key:highway) (e.g. motorway, trunk, primary,…) of a road for a given position. To do so, we have to execute 2 requests:

* The first one is a reverse geocoding request, which will give us the street address of a location and the OSM ID of the associated way.
* The second is a Xapi request, which will give us highway information for a give highway id, like its type.

Assume that the location correspond to the following geo coordinates `(48.888320; 2.249920)`.

The first step can be accomplished by sending the following GET request (follow the link to see the result):

[http://open.mapquestapi.com/nominatim/v1/reverse?lat=48.888320&lon=2.249920](http://open.mapquestapi.com/nominatim/v1/reverse?lat=48.888320&lon=2.249920)

In the result XML, we can see the `...osm_type="way" osm_id="14310041"...` string. We can conclude that the returned object type is `way`, and its OSM id is `14310041`.

These data will allow us to write and execute the second request (follow the link to see the result):

[http://open.mapquestapi.com/xapi/api/0.6/way/14310041](http://open.mapquestapi.com/xapi/api/0.6/way/14310041)

Among the result's tags, we can find the `<tag k="highway" v="motorway"/>` key/value tag. We finally can assert that the highway type is `motorway`!

## Additional links

* To learn more about OpenStreetMap, you can have a see on their [FAQ](https://wiki.openstreetmap.org/wiki/FAQ).
* The community greatly highlights the ability for common user to contribute to the project (create and update geographic content). To learn more on how to contribute, you can read their [beginners' guide](https://wiki.openstreetmap.org/wiki/Beginners'_guide).
* To learn more on the MapQuest API, see their [developer's guide](https://developer.mapquest.com/).
* To learn more on using Xapi, you can read the [dedicated wiki page](https://wiki.openstreetmap.org/wiki/Xapi) and test it on [MapQuest Xapi page](http://open.mapquestapi.com/xapi/).
* To learn more on using Nominatim, you can read the [dedicated wiki page](https://wiki.openstreetmap.org/wiki/Nominatim) and test it on [MapQuest Nominatim page](http://open.mapquestapi.com/nominatim/).
* Other geocoding web services using OpenStreetMap exist on the web. You can have a look at [Worldwide Geocoding](http://www.worldwidegeocoding.com/) or [GeoNames](http://www.geonames.org/).
