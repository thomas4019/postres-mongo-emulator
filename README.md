<p align="center">
  <img alt="pgmongo logo" src="https://user-images.githubusercontent.com/406149/42555295-9ccff858-849c-11e8-81dd-7d5fa5e7bf94.png">
  <p align="center">Replace MongoDB with PosgreSQL</p>
</p>

## What is pgmongo?
- **Drop-in replacement** Applications do not need code changes. pgmongo appears to your app as a MongoDB server.
- **Stateless proxy** pgmongo rewrites queries and proxies them to a Postgres database.
- **JSON** Primarily supports regular JSON data. Advanced BSON types like binary data, JavaSrcipt, ints and timestamps are not well supported.

This implements the [MongoDB wire protocol](https://docs.mongodb.com/manual/reference/mongodb-wire-protocol/) and adapts queries to work with a PostgreSQL database using jsonb fields.
I've tested it with [Keystone.js](http://keystonejs.com/) and it seemed to work reasonably well.

<p align="center">
  <img alt="data flow diagram converting bson to jsonb" src="https://user-images.githubusercontent.com/406149/50381947-f0495980-0650-11e9-8c07-6f1f43fcd321.png">
</p>


## Getting Started
pgmongo requires node 8 or newer and Postgres 9.4+. Then run the following.
```bash
npm install -g pgmongo
pgmongo
```
This will start a mongo-like server on port 27017. If you already have mongo running on your machine you can start it on a different port with the following.
```bash
pgmongo localhost 27018
```

## Supported Features
* listing/creating/dropping collections
* find (including sorting, skip and offset)
* count, distinct
* update (including support for upserting)
* insert (including batch insertion)
* deletion
* creating and listing basic indexes
* most custom parameters like $gt, $exists, $regex, $push, $set, $unset, etc.
See [this repo](https://github.com/thomas4019/mongo-query-to-postgres-jsonb)  for the full list
* admin commands (returns mostly stubbed/fake data)

# Current status
It's not production ready yet, but definitely working enough to play around with or use in basic apps.  
Currently passes 190 of the 916 core mongo [jstests](https://github.com/mongodb/mongo/tree/master/jstests/core).

## Example Query Conversions
```
db.createCollection('users')  ->  CREATE TABLE IF NOT EXISTS "users" (data jsonb)
db.users.find({ lastLogin: { $lte: '2016' } })  ->  SELECT data FROM "users" WHERE data->>'lastLogin'<='2016'
db.users.update({}, { $set: { active: true } })  ->  UPDATE "users" SET data = jsonb_set(data,'{active}','true'::jsonb)
db.users.find({}, { firstName: 1 } )  ->  SELECT jsonb_build_object('firstName', data->'firstName', '_id', data->'_id') as data FROM "users"
db.blogs.insert({ title: 'first post', comments: [] })  ->  INSERT INTO "blogs" VALUES ('{"_id":"5b45b641eb4bd93896d57888","title":"first post","comments":[]}'::jsonb)
db.blogs.remove({ 'state.trash': true })  ->  DELETE FROM "blogs" WHERE data->'state'->'trash'='true'::jsonb
```

## Missing Features / Roadmap (ordered by priority)
Note: contributions/PRs are very much welcome.
* Support for findandmodify
* Better for queries matching array elements
* Preserve BSON (Dates, ObjectIDs, other than _id)
* Cursors (currently all data is returned in first result)
* Better Indexes support (not sure if compound indexes are possible)
* [min](https://docs.mongodb.com/manual/reference/method/cursor.min/) and max
* Support numeric object keys (currently numbers are assumed to be an array index)
* Capped collections
* geo support
* explain queries
* aggregation framework/map reduce queries

### Likely Cannot support
* NaN and Infinity
* Preserve the initial order of object keys
* $eval and $where

## Resources
* [mongo-query-to-postgres-jsonb](https://github.com/thomas4019/mongo-query-to-postgres-jsonb) (heavily used as a dependency)
* [Mongo Admin Commands](https://docs.mongodb.com/manual/reference/command/nav-administration/)
* [JSONB Index Performance](http://bitnine.net/blog-postgresql/postgresql-internals-jsonb-type-and-its-indexes/)

## Want to contribute?
Anyone can help make this project better. Feel free to open an issue to discuss what you want to work on.

## Related Projects
[Mongres](https://github.com/umitanuki/mongres),
[mongolike](https://github.com/JerrySievert/mongolike),
[plv8](https://github.com/plv8/plv8),
[ToroDB](https://news.ycombinator.com/item?id=8527013)
