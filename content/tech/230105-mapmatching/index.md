+++
author = "Johannes Ehm"
title = "Geographical Algorithms - Map Matching"
date = "2023-01-05"
description = "Match geopositions to maps"
tags = [
	"tech",
	"algorithms",
	"geospatial",
	"english"
]
+++

Map matching serves the purpose of preprocess geographical data and to fix deviations of geopositions from the real geopositions due to measurement errors or the lack of measurements. Map matching applies geo positions to maps and assign the geo positions to the most probable geo position with considering the features of the map. The following YouTube video is an introduction into map matching.

{{< youtube 92TKceDw-kU>}}

When I was working on a project, I was interested into map matching algorithms especially how they work and what the map service providers do offer. In order to improve my understanding I read the YouTube video to have an introduction into the area. According to the video, most map matching algorithms are build on the work of [Newson, Paul, and John Krumm "Hidden Markov map matching through noise and sparseness"][1]. The [paper of the Microsoft Research employees from 2009][2] describes an approach of an map matching algorithm that uses a Hidden Markov Model to find the most likely road route represented by a time-stamped sequence of latitude/longitude pairs (cmp. [1]). The offered approach should be robust to location data which is noisy and sparse, which makes it important to take not only the gps points but the sequences of gps points into account. The application of a Hidden Markov Model to the map matching problem is to consider sequences and the probability of state transitions. The Hidden Markov Model is used in the area of speech recognition and applied to the map matching problem already before the paper of [Newson, Paul, and John Krumm][1]. A [Hidden Markov Model][3] allows to model state transitions and the probability of state transitions in order to calculate the probability of a certain state given a set of oberservation data. In the terms of map matching, it allows to model road segments and the probability of a transition between the road segments in order to calculate the probability of a route given the emission probability of road segments based on the observed gps points.

# Map Service Providers Map Matching Services

Several map service providers do offer map matching services. However, these map matching services come with restrictions and are expensive. I did collect some links for the map service providers Map Box, Google Maps and Here.

## Map Box

https://docs.mapbox.com/api/navigation/

https://docs.mapbox.com/api/navigation/map-matching/

https://docs.mapbox.com/android/navigation/examples/

https://github.com/mapbox/mapbox-navigation-android-examples

## Google Maps Roads API

https://developers.google.com/maps/documentation/roads

https://developers.google.com/maps/documentation/roads/snap

https://github.com/googlemaps/google-maps-services-java

## Here Location Library

https://developer.here.com/documentation/location-library/dev_guide/index.html

https://developer.here.com/documentation/location-library/dev_guide/docs/high-level.html#create-path-matchers

https://github.com/heremaps/here-workspace-examples-java-scala/tree/master/location

# Open Source Map Matching Implementations

More interesting are the open source map matching engines which are presented in the YouTube video OSRM and Valhalla. While investigating the implementation of map matching in each of these two projects is a chapter on its [own][8], I did read a [medium article][4] where some geojson data is applied to the Valhalla map matching algorithm Meili. Hernandez, the author of this article did publish his python code to [Github][5] and even mentions a nice way to produce the input data using [geojson.io][6]. In order to apply the data of map matching to Valhalla he did run a [Valhalla docker container][7].

## OSRM

https://project-osrm.org/

https://github.com/Project-OSRM

## Valhalla

https://valhalla.github.io/valhalla/

https://github.com/valhalla/valhalla

## Graphhopper

https://www.graphhopper.com/de/

https://github.com/graphhopper/graphhopper/tree/master/map-matching

# Running Python Map Matching with Valhalla Meili

Valhalla has a container image available at [the Github packages image registry][9]. In my [documentation-container github project][10] I did collect some basic information about the image. To use the image, a login into the Github packages image registry and a pull of the image is necessary:

```
docker login ghcr.io

docker pull ghcr.io/gis-ops/docker-valhalla/valhalla:latest
```

The Meili map machting requires map data which we can acquire from an Open Street Map server as a pbf file. A [pbf file][11] is binary protocol buffer file to get Open Street Map data. Similiar to the documentation to [how to use maili][7] I am fetching the OpenStreetMap data of Baden-Würrtemberg:

```
wget http://download.geofabrik.de/europe/germany/baden-wuerttemberg-latest.osm.pbf
```

```
docker run -it --rm --name valhalla -p 8002:8002 -v $PWD/valhalla:/custom_files/ -e serve_tiles=True -e build_admins=True ghcr.io/gis-ops/docker-valhalla/valhalla:latest
```

# Running Map Matching with the Java Graphhopper Map Matching

## References

https://www.microsoft.com/en-us/research/publication/hidden-markov-map-matching-noise-sparseness/

https://www.microsoft.com/en-us/research/wp-content/uploads/2016/12/map-matching-ACM-GIS-camera-ready.pdf

https://de.wikipedia.org/wiki/Hidden_Markov_Model

https://medium.com/@hernandezgalaviz/using-valhalla-map-matching-to-get-route-and-travelled-distance-from-raw-gps-points-4ea6b1c88a4c

https://github.com/galaviz-mx/Valhalla-MapMatching

https://geojson.io/

https://ikespand.github.io/posts/meili/

https://towardsdatascience.com/map-matching-done-right-using-valhallas-meili-f635ebd17053

[1]: https://www.microsoft.com/en-us/research/publication/hidden-markov-map-matching-noise-sparseness/

[2]: https://www.microsoft.com/en-us/research/wp-content/uploads/2016/12/map-matching-ACM-GIS-camera-ready.pdf

[3]: https://de.wikipedia.org/wiki/Hidden_Markov_Model

[4]: https://medium.com/@hernandezgalaviz/using-valhalla-map-matching-to-get-route-and-travelled-distance-from-raw-gps-points-4ea6b1c88a4c

[5]: https://github.com/galaviz-mx/Valhalla-MapMatching

[6]: https://geojson.io/

[7]: https://ikespand.github.io/posts/meili/

[8]: https://towardsdatascience.com/map-matching-done-right-using-valhallas-meili-f635ebd17053

[9]: https://github.com/gis-ops/docker-valhalla/pkgs/container/docker-valhalla%2Fvalhalla

[10]: https://wiki.openstreetmap.org/wiki/PBF_Format