# WFSGIS

<img src="https://img.shields.io/docker/automated/flaviostutz/wfsgis"/><br/>
[<img src="https://goreportcard.com/badge/github.com/flaviostutz/wfsgis"/>](https://goreportcard.com/report/github.com/flaviostutz/wfsgis)

A WFS 3.0 server with backend storage in Postgis

You can define the timestamp of the feature during insert by using two methods (see examples):
  * add a "time" attribute as one of the properties of GeoJSON in ISO format

See a brief demo on how to add geographic points and show them in a browser at https://youtu.be/HSv-w5_Cc6U

## Usage

* Create a docker-compose.yml file

```yml
version: '3.7'

services:

  wfsgis:
    image: flaviostutz/wfsgis
    ports: 
      - 8080:8080
    restart: always
    environment:
      - POSTGRES_HOST=postgis
      - POSTGRES_USERNAME=wfs3
      - POSTGRES_PASSWORD=wfs3
      - POSTGRES_DBNAME=wfs3

  pgadmin:
    image: dpage/pgadmin4:4.8
    ports:
      - 8081:80
    restart: always
    environment:
      - PGADMIN_DEFAULT_EMAIL=admin
      - PGADMIN_DEFAULT_PASSWORD=admin

  postgis:
    # image: timescale/timescaledb-postgis:1.3.1-pg9.6
    image: mdillon/postgis:11-alpine
    ports:
      - 5432:5432
    environment:
      - POSTGRES_USER=wfs3
      - POSTGRES_PASSWORD=wfs3
      - POSTGRES_DB=wfs3
    volumes:
      - pg-data:/var/lib/postgresql/data

volumes:
  pg-data:

```

* Run "docker-compose up"

* WFS3 API is available at http://localhost:8080/
  * /collections/[collection_name]/items/[item_id]

* Postgis Admin UI at http://localhost:8081


### REST Examples

* Create a new collection

```
curl -X POST \
  http://localhost:8080/collections \
  -H 'Accept: */*' \
  -H 'Content-Type: application/json' \
  -d '{
	"name": "test6",
	"title": "test 6",
	"description": "test 6 test 6",
	"crs": ["EPSG4326"]
}
'
```

* Get collection list

```
curl -X GET \
  http://localhost:8080/collections \
  -H 'Accept: */*' \
```

* Create a new Feature (from a complex GeoJSON)

```
curl -X POST \
  http://localhost:8080/collections/test6/items \
  -H 'Accept: */*' \
  -H 'Content-Type: application/json' \
  -d '{
   "type":"FeatureCollection",
   "features":[
      {
         "type":"Feature",
         "geometry":{
            "type":"Point",
            "coordinates":[
               102.0,
               0.5
            ]
         },
         "properties":{
            "prop0":"value0",
            "time" : "2018-02-01T15:04:02Z"
         }
      },
      {
         "type":"Feature",
         "geometry":{
            "type":"LineString",
            "coordinates":[
               [
                  102.0,
                  0.0
               ],
               [
                  103.0,
                  1.0
               ],
               [
                  104.0,
                  0.0
               ],
               [
                  105.0,
                  1.0
               ]
            ]
         },
         "properties":{
            "prop0":"value1",
            "prop1":0.0
         }
      },
      {
         "type":"Feature",
         "geometry":{
            "type":"Polygon",
            "coordinates":[
               [
                  [
                     100.0,
                     0.0
                  ],
                  [
                     101.0,
                     0.0
                  ],
                  [
                     101.0,
                     1.0
                  ],
                  [
                     100.0,
                     1.0
                  ],
                  [
                     100.0,
                     0.0
                  ]
               ]
            ]
         },
         "properties":{
            "prop0":"value2",
            "prop1":{
               "this":"that"
            }
         },
        "when": {
            "@type": "Instant",
            "datetime": "2019-12-11T10:02:44Z"
        }
      }
   ]
}'
```

* Query Features by bounding box, time range and property value

```
curl -X GET \
  'http://localhost:8080/collections/test6/items?bbox=101,0,103,1&limit=10&time=2019-01-01T15:04:02Z/2019-12-31T15:04:02Z&prop0=value2' \
  -H 'Accept: */*' \
  -H 'cache-control: no-cache'
```

* Get a item

```
curl -X GET \
  'http://localhost:8080/collections/test6/items/2' \
  -H 'Accept: */*'
```

* Delete item

```
curl -X DELETE \
  http://localhost:8080/collections/test6/items/2 \
  -H 'cache-control: no-cache'
```

## Flow

![WFSDiagram](WFSDiagram.png)
