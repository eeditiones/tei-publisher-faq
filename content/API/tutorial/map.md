---
title: "Showing Places on a Map"
menuTitle: "Maps"
date: 2022-01-30T12:14:39+01:00
tags: ["api"]
weight: 7
---

In addition to browsing through the places, it would also be nice to display their location on a map. Ideally the map should display all places occurring in the edition, so we can easily see which places are geographically close. We would thus need an additional endpoint, returning a simple list of all places with their geographical coordinates. Let's add an endpoint for `/places/all` above our existing endpoint for `/places`:

```json
"/api/places/all": {
    "get": {
        "summary": "List all places",
        "description": "Retrieve list of all places",
        "operationId": "custom:places-all",
        "responses": {
            "200": {
                "description": "List of all places",
                "content": {
                    "application/json": {
                        "schema": {
                            "type": "array",
                            "items": {
                                "type": "object",
                                "properties": {
                                    "latitude":{
                                        "type": "string"
                                    },
                                    "longitude":{
                                        "type": "string"
                                    },
                                    "label": {
                                        "type": "string",
                                        "description": "Label to show in the tooltip"
                                    }
                                }
                            }
                        }
                    }
                }
            }
        }
    }			
}
```

The implementation of `custom:places-all` is straightforward: add the following function to `modules/custom-api.xql`:

```xquery
declare function api:places-all($request as map(*)) {
    let $places := doc($config:data-root || "/playground/places.xml")//tei:listPlace/tei:place
    return 
        array { 
            for $place in $places
                let $tokenized := tokenize($place/tei:location/tei:geo)
                return 
                    map {
                        "latitude":$tokenized[1],
                        "longitude":$tokenized[2],
                        "label":$place/@n/string()
                    }
            }        
};
```

Now we can extend the HTML template, `places.html` to add a map component:

1. import the javascript library for the map component below the last `<script>` in the header:

    ```html
    <script type="module" src="pb-leaflet-map.js" data-template="pages:load-components"/>
    ```
2. into the CSS `<style>` section, add a rule to size the map (otherwise it will be 0 height):
    ```css
    pb-leaflet-map {
        height: 40vh;
        width: 100%;
    }
    ```
3. add the HTML for the map component before the `<main>` element:
    ```html
    <pb-leaflet-map id="map" subscribe="map" emit="map" zoom="10" cluster="" latitude="47.3686498" longitude="8.5391825">
        <pb-map-layer show=""
            base="" 
            label="Mapbox OSM"                                 
            url="https://api.mapbox.com/styles/v1/mapbox/streets-v11/tiles/{z}/{x}/{y}?access_token={accessToken}" 
            max-zoom="19" 
            access-token="pk.eyJ1Ijoid29sZmdhbmdtbSIsImEiOiJjam1kMjVpMnUwNm9wM3JwMzdsNGhhcnZ0In0.v65crewF-dkNsPF3o1Q4uw" 
            attribution="© Mapbox © OpenStreetMap">
        </pb-map-layer>
    </pb-leaflet-map>
    ```
4. finally we need a bit of javascript before the closing `</body>`:
   ```javascript
   <script>
        window.addEventListener('WebComponentsReady', function() {
            pbEvents.subscribe('pb-page-ready', null, function() {
                const endpoint = document.querySelector("pb-page").getEndpoint();
                const url = `${endpoint}/api/places/all`;
                console.log(`fetching places from: ${url}`);
                fetch(url)                
                .then(function(response) {
                    return response.json();
                })
                .then(function(json) {
                    pbEvents.emit("pb-update-map", "map", json)
                });
            });
        });
    </script>
    ```
    This script waits until webcomponents have been loaded and fully initialized (`pb-page-ready` event). It then uses the browser's [fetch](https://developer.mozilla.org/en-US/docs/Web/API/Fetch_API) function to retrieve the list of places from our endpoint, and sends the resulting JSON array via a `pb-update-map` event to the map for display.

![Map display](/images/map.png)

### Zooming the Map to a Place

One last feature is still missing though: the places listed in the browsing view are not connected to the map. Would be nice if we had an icon next to each place name to click on and have the map zoom to the corresponding coordinates. To make this happen, replace the function `api:output-place` in `modules/custom-api.xql` with the following version:

```xquery
    declare function api:output-place($list, $category as xs:string, $search as xs:string?) {
    array {
        for $place in $list
        let $categoryParam := if ($category = "all") then substring($place/@n, 1, 1) else $category
        let $params := "category=" || $categoryParam || "&amp;search=" || $search
        let $label := $place/@n/string()
        let $coords := tokenize($place/tei:location/tei:geo)
        return
            <span class="place">
                <a href="{$label}?{$params}">{$label}</a>
                <pb-geolocation latitude="{$coords[1]}" longitude="{$coords[2]}" label="{$label}" emit="map" event="click">
                    <iron-icon icon="maps:map"></iron-icon>
                </pb-geolocation>
            </span>
    }
};
```

This will add a little map icon next to each place. The `pb-geolocation` component around the icon creates a link, which - when clicked - instructs the map to zoom to the given coordinates.