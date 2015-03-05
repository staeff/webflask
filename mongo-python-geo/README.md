# Getting started with MongoDB

taken from here: https://github.com/behackett/presentations/tree/master/where2_2011

The original data is coming from http://www.bart.gov/schedules/developers/gtfs  


## Setup

The load script assumes you are on *nix. The python code
should work on any platform.

To load the datasets and geo index them first start mongod then:
$ ./load_data.sh
$ python geo_index_data.py

Examples of geo queries using python available in geo_examples.py:
$ python
Python 2.6.1 (r261:67515, Jun 24 2010, 21:47:49)
[GCC 4.2.1 (Apple Inc. build 5646)] on darwin
Type "help", "copyright", "credits" or "license" for more information.
>>> import geo_examples as ge
>>> help(ge)


## Tutorial

### Getting to know MongoDB

Adding a simple document  

```javascript
> post = { author: "Stephan", date: new Date(), text: "Working with Geo Data", tags: [ "mongodb", "geo" ]}
> db.posts.save(post)
```

Find the document  

```javascript
> db.posts.find()
```

Add an index, find via Index

Secondary index for "author" //   1 means ascending, -1 means descending

```javascript
> db.posts.ensureIndex( {author: 1 } )
> db.posts.find( { author: 'Stephan' } )
```

### Geo Data

Two-dimensional geospatial indexes, Points must be specified in a subdocument or array:

* { loc : [ 50 , 30 ] }
* { loc : { x : 50 , y : 30 } }
* { loc : { foo : 50 , y : 30 } }
* { loc : { long : 40.739037, lat: 73.992964 } }

For spherical queries order must be long, lat.

```javascript
> db.zips.findOne({'zip':'95054'})

> db.stops.findOne()
```

Add a Geo index, find via index

```javascript
> db.stops.ensureIndex({'stop_geo': '2d'})
> db.stops.find({ 'stop_geo': {'$near': [-122.419088, 37.75689]}}).limit(1)
```

Find distance using geoNear

```javascript
> var mult = 3963 * (Math.PI / 180) // Scale to miles on Earth (very rough approximation).
> db.runCommand({'geoNear': 'stops', 'near': [-122.419088, 37.75689], distanceMultiplier: mult})
```

Bounds Queries (Find within a shape)  

```javsacript
> // Number of people living within (roughly)100 miles
> // of the Empire State Building.
> // Second argument to $center is the radius.
> var q = {'$within': {'$center': [[-73.985656, 40.748433], 100 / (3963 * (Math.PI / 180))]}}
> var sum = 0
> var cur = db.zips.find({'loc': q})
> for( var i = 0; i < cur.length(); i++ ){
      sum += cur[i].pop;
}
```

Spherical example with geoNear  

```javascript
> db.runCommand( { 'geoNear': 'stops','near': [-122.419088, 37.75689], distanceMultiplier: 3963, spherical: true})
```

Another spherical example...

```javascript
> // Number of people living within100 miles of the> // Empire State Building (again).
> var sum = 0
> var maxdist = 100 / 3963
> // limit(100) is default if not specified...
> var cur = db.zips.find({'loc': {'$nearSphere': [-73.985656, 40.748433], $maxDistance: maxdist}}).limit(60000)
> for(var i = 0; i < cur.length(); i++){
     sum += cur[i].pop;
}
```
