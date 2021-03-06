MongoDB River Plugin for ElasticSearch
==================================

This plugin uses MongoDB as datasource to store data in ElasticSearch. Filtering and transformation are also possible. 
See the [wiki](https://github.com/richardwilly98/elasticsearch-river-mongodb/wiki) for more details.

In order to install the plugin, simply run: ```bin/plugin --install com.github.richardwilly98.elasticsearch/elasticsearch-river-mongodb/2.0.0```

| MongoDB River Plugin     | ElasticSearch    | MongoDB       |
|--------------------------|------------------|---------------|
| master                   | 1.1.1            | 2.4.9 -> 2.6.1|
| 2.0.0                    | 1.0.0            | 2.4.9         |
| 1.7.4                    | 0.90.10          | 2.4.8         |
| 1.7.3                    | 0.90.7           | 2.4.8         |
| 1.7.2                    | 0.90.5           | 2.4.8         |
| 1.7.1                    | 0.90.5           | 2.4.6         |
| 1.7.0                    | 0.90.3           | 2.4.5         |
| 1.6.11                   | 0.90.2           | 2.4.5         |
| 1.6.9                    | 0.90.1           | 2.4.4         |
| 1.6.8                    | 0.90.0           | 2.4.3         |
| 1.6.7                    | 0.90.0           | 2.4.3         |
| 1.6.6                    | 0.90.0           | 2.4.3         |
| 1.6.5                    | 0.20.6           | 2.4.1         |
| 1.6.4                    | 0.20.5           | 2.2.3         |
| 1.6.2                    | 0.20.1           | 2.2.2         |
| 1.6.0                    | 0.20.1 -> master | 2.2.2         |
| 1.5.0                    | 0.19.11          | 2.2.1         |
| 1.4.0                    | 0.19.8           | 2.0.5         |
| 1.3.0                    | 0.19.4           |               |
| 1.2.0                    | 0.19.0           |               |
| 1.1.0                    | 0.19.0           | 2.0.2         |
| 1.0.0                    | 0.18             |               |

Build status
-------

[![Build Status](https://drone.io/github.com/richardwilly98/elasticsearch-river-mongodb/status.png)](https://drone.io/github.com/richardwilly98/elasticsearch-river-mongodb/latest)

Initial implementation by [aparo](https://github.com/aparo). For the initial implementation see [tutorial](http://www.matt-reid.co.uk/blog_post.php?id=68#&slider1=4).

Modified to get the same structure as the other Elasticsearch rivers (like [CouchDB](http://www.elasticsearch.org/blog/2010/09/28/the_river_searchable_couchdb.html))

The latest version monitors the oplog capped collection and supports attachment (GridFS).

Configure the river using the definition described in the [wiki](https://github.com/richardwilly98/elasticsearch-river-mongodb/wiki):

  curl -XPUT 'http://localhost:9200/_river/mongodb/_meta' -d '{
    "type": "mongodb", 
    "mongodb": { 
      "db": "DATABASE_NAME", 
      "collection": "COLLECTION", 
      "gridfs": true
    }, 
    "index": { 
      "name": "ES_INDEX_NAME", 
      "type": "ES_TYPE_NAME" 
    }
  }'

Example:

  curl -XPUT 'http://localhost:9200/_river/mongodb/_meta' -d '{ 
    "type": "mongodb", 
    "mongodb": { 
      "db": "testmongo", 
      "collection": "person"
    }, 
    "index": {
      "name": "mongoindex", 
      "type": "person" 
    }
  }'

Import data from mongo console:

  use testmongo
  var p = {firstName: "John", lastName: "Doe"}
  db.person.save(p)

Query index:

  curl -XGET 'http://localhost:9200/mongoindex/_search?q=firstName:John'

  curl -XPUT 'http://localhost:9200/_river/mongodb/_meta' -d '{ 
    "type": "mongodb", 
    "mongodb": { 
      "db": "testmongo", 
      "collection": "fs", 
      "gridfs": true 
    }, 
    "index": {
      "name": "mongoindex", 
      "type": "files" 
    }
  }'

Import binary content in mongo:

  %MONGO_HOME%\bin>mongofiles.exe --host localhost:27017 --db testmongo --collection fs put test-document-2.pdf
  connected to: localhost:27017
  added file: { _id: ObjectId('4f230588a7da6e94984d88a1'), filename: "test-document-2.pdf", chunkSize: 262144, uploadDate: new Date(1327695240206), md5: "c2f251205576566826f86cd969158f24", length: 173293 }
  done!

Query index:

  curl -XGET 'http://localhost:9200/files/4f230588a7da6e94984d88a1?pretty=true'

Admin URL: http://localhost:9200/_plugin/river-mongodb/

See more details check the [wiki](https://github.com/richardwilly98/elasticsearch-river-mongodb/wiki)

License
-------

    This software is licensed under the Apache 2 license, quoted below.

    Copyright 2009-2012 Shay Banon and ElasticSearch <http://www.elasticsearch.org>

    Licensed under the Apache License, Version 2.0 (the "License"); you may not
    use this file except in compliance with the License. You may obtain a copy of
    the License at

        http://www.apache.org/licenses/LICENSE-2.0

    Unless required by applicable law or agreed to in writing, software
    distributed under the License is distributed on an "AS IS" BASIS, WITHOUT
    WARRANTIES OR CONDITIONS OF ANY KIND, either express or implied. See the
    License for the specific language governing permissions and limitations under
    the License.

Changelog
-------

#### 2.0.0
- Update versions ES 1.0.0, MongoDB 2.4.9, MongoDB driver 2.11.4
- TODO

#### 1.7.4
- TODO

#### 1.7.3
- Update versions ES 0.90.7
- Optimization of oplog.rs query. The current query was too complex and not efficient in MongoDB. oplog.rs query now uses only $ts filter.
- New ```options/import_all_collections``` parameter can be used to import all collection of a database (see issue [#177](https://github.com/richardwilly98/elasticsearch-river-mongodb/issues/177))
- Version, commit data and commit id are displayed in the log when the river starts.
- New ```options/store_statistics``` parameter can be used to store statistic each time bulk processor is flushed. Data are store in _river/{river.name}
- Default value for ```index/bulk/concurrent_bulk_requests``` has been changed to the number of cores available.
- Capture failures from bulk processor and set status to IMPORT_FAILED when found.
- Fix issues in document indexed counter in administration.
- Refactoring to use 1 bulk processor per index/type.

#### 1.7.2
- Update versions MongoDB 2.4.8
- Optimization of oplog.rs filter (see issue [#123](https://github.com/richardwilly98/elasticsearch-river-mongodb/issues/123))
- ```options/drop_collection``` will also track ```dropDatabase``` (see issue [#133](https://github.com/richardwilly98/elasticsearch-river-mongodb/issues/133))
- Add filter in the initial import by @bernd (see issue [#157](https://github.com/richardwilly98/elasticsearch-river-mongodb/issues/157))
- MongoDB default port is 27017 (if not specific in configuration) (see issue [#159](https://github.com/richardwilly98/elasticsearch-river-mongodb/issues/159))
- Allow alias in place of index (see issue [#163](https://github.com/richardwilly98/elasticsearch-river-mongodb/issues/163))
- Initial import supports existing index (see issue [#167](https://github.com/richardwilly98/elasticsearch-river-mongodb/issues/167))
- New parameter ```options/skip_initial_import``` to skip initial import using collection data. Default value is false.
- Stop properly the river when deleted (see issue [#169](https://github.com/richardwilly98/elasticsearch-river-mongodb/issues/169))
- Administration updates: stopped river can be deleted from the administration, auto-refresh.
- Switch to BulkProcessor API to provide more flexible configuration during bulk indexing. (see [commit](https://github.com/richardwilly98/elasticsearch-river-mongodb/commit/973dba973f6c1c353a38ed060754967504485769))

#### 1.7.1
- Update versions ES 0.90.5, MongoDB driver 2.11.3
- Initial import using the collection by @benmccann. (see issue [#47](https://github.com/richardwilly98/elasticsearch-river-mongodb/issues/47))
- Add unit test to validate Chinese support (see issue [#95](https://github.com/richardwilly98/elasticsearch-river-mongodb/issues/95))
- Ensure MongoDB cursor is closed (see issue [#comment-24427369](https://github.com/richardwilly98/elasticsearch-river-mongodb/pull/106#issuecomment-24427369))
- River administration has been improved. (see issue [#109](https://github.com/richardwilly98/elasticsearch-river-mongodb/pull/109))
- Allow fields ```ts``` or ```op``` to be used in user collection. (see pr [#136](https://github.com/richardwilly98/elasticsearch-river-mongodb/pull/136))
- Use of OPLOG_REPLAY to query oplog.rs

#### 1.7.0
- Update versions ES 0.90.3, MongoDB 2.4.6
- Ability to index documents from a given datetime (see issue [#102](https://github.com/richardwilly98/elasticsearch-river-mongodb/issues/102))
- Fix for ```options/exclude_fields``` by @ozanozen (see issue [#103](https://github.com/richardwilly98/elasticsearch-river-mongodb/issues/103))
- Fix for ```options/drop_collection``` (see issue [#105](https://github.com/richardwilly98/elasticsearch-river-mongodb/issues/105))
- New advanced transformation feature.  (see issue [#106](https://github.com/richardwilly98/elasticsearch-river-mongodb/pull/106))
- Add site to the river. Initial implementation (only start / stop river or display river settings).  (see issue [#109](https://github.com/richardwilly98/elasticsearch-river-mongodb/pull/109))
- Implement include fields (see issue [#119](https://github.com/richardwilly98/elasticsearch-river-mongodb/issues/119))
- Refactoring of the river definition (new class MongoDBRiverDefinition).

#### 1.6.11
- Add SSL support by @alistair (see [#94](https://github.com/richardwilly98/elasticsearch-river-mongodb/pull/94))
- Add support for $set operation (see issue [#91](https://github.com/richardwilly98/elasticsearch-river-mongodb/issues/91))
- Add Groovy unit test (for feature [#87](https://github.com/richardwilly98/elasticsearch-river-mongodb/issues/87))
- Update versions ES 0.90.2, MongoDB 2.4.5 and MongoDB driver 2.11.2
- Fix for ```options/drop_collection``` option (issue [#79](https://github.com/richardwilly98/elasticsearch-river-mongodb/issues/79))
- New ```options/include_collection``` parameter to include the collection name in the document indexed. (see [#101](https://github.com/richardwilly98/elasticsearch-river-mongodb/issues/101))

#### 1.6.9
- Allow the script filters to modify the document id (see [#83](https://github.com/richardwilly98/elasticsearch-river-mongodb/pull/83))
- Support for Elasticsearch 0.90.1 and MongoDB 2.4.4
- Improve exclude fields (support multi-level - see [#76](https://github.com/richardwilly98/elasticsearch-river-mongodb/issues/76))
- Fix to support ObjectId (see issue [#85](https://github.com/richardwilly98/elasticsearch-river-mongodb/issues/85))
- Add logger object to script filters
- Provide example for Groovy script (see issue[#87](https://github.com/richardwilly98/elasticsearch-river-mongodb/issues/87))

#### 1.6.8 
- Implement exclude fields (see issue #76)
- Improve reconnection to MongoDB when connection is lost (see issue #77)
- Implement drop collection feature (see issue #79). The river will drop all documents from the index type.

#### 1.6.7 
- Issue with sharded collection (see issue #46)

#### 1.6.6 
- Support for Elasticsearch 0.90.0 and MongoDB 2.4.3
- MongoDB driver 2.11.1 (use of MongoClient)  

#### 1.6.5 
- Add support for _parent, _routing (see issue #64)

#### 1.6.4 
- Fix NPE (see issue #60)
- Remove database user, password river settings. Local or admin user, password should be used instead.

#### 1.6.3 
- First attempt to stored the artifact in Maven central (please ignore this version

#### 1.6.2 
- Support for secured sharded collection (see issue #60)

#### 1.6.0 
- Support for sharded collection
- Script filters
- MongoDB driver 2.10.1 (use of MongoClient)



[![Bitdeli Badge](https://d2weczhvl823v0.cloudfront.net/richardwilly98/elasticsearch-river-mongodb/trend.png)](https://bitdeli.com/free "Bitdeli Badge")

