<div id="table-of-contents">
<h2>Table of Contents</h2>
<div id="text-table-of-contents">
<ul>
<li><a href="#sec-1">1. A Datomic Tutorial</a>
<ul>
<li><a href="#sec-1-1">1.1. Data Conceptual Shape</a></li>
<li><a href="#sec-1-2">1.2. Keywords</a></li>
<li><a href="#sec-1-3">1.3. Basic Schema</a></li>
<li><a href="#sec-1-4">1.4. Testdata</a></li>
<li><a href="#sec-1-5">1.5. Blow away and recreate DB</a></li>
</ul>
</li>
</ul>
</div>
</div>

# A Datomic Tutorial<a id="sec-1" name="sec-1"></a>

## Data Conceptual Shape<a id="sec-1-1" name="sec-1-1"></a>

Datomic allows clojure programmers to work with data in the structures
they are comfortable with, maps and vectors.  The way to think about
datomic is that it is one giant vector of maps.  Each map in this
vector is what they call and *Entity*.  So lets draw this out with an
example: 

    [{:db/id 1
      :car/make "toyota"
      :car/model "tacoma"
      :year 2014}
    
     {:db/id 2
      :car/make "BMW"
      :car/model "325xi"
      :year 2001}
    
     {:db/id 3
      :user/name "ftravers"
      :user/age 54
      :cars [{:db/id 1}
             {:db/id 2}]}]

So this datomic database has 3 entries/entities/maps.  A user,
`ftravers`, who owns 2 cars.  Every map has to have a `:db/id`.  This
is what uniquely identifies that entity to datomic.  Datomic
`:db/id`'s are actually very large integers, so the data above is
actually a bit fake, but I keep it simple to communicate the concept.

As we can see in the above example, the `:cars` field of the user
`ftravers` points (refers/links) to the cars he owns using the
`:db/id` field.

## Keywords<a id="sec-1-2" name="sec-1-2"></a>

The keywords that our maps use are not just hodge-podge, willy-nilly,
used.  Rather we have to specify ahead of time the set of fields that
maps can have/use.  This is called creating a schema.  A schema in
regular relational database (RDBMS) means specifying tables and columns.  In
datomic, we ONLY specify 'columns', or the entire set of allowed
keywords.  Just like in an RDBMS, when we specify a column, we
indicate the type of data that will live in that field.  We might
setup a column to be a foriegn key (reference/link) to the primary key
of another table.  Thats how you link rows together.

So lets take a break from concepts and see how to connect to a
database quickly.

    (def db-url "datomic:free://127.0.0.1:4334/omn-dev")
    (def db-conn (atom (d/connect db-url)))

## Basic Schema<a id="sec-1-3" name="sec-1-3"></a>

Now lets define a schema with just one field.

    (def schema [{:db/doc "A users email."
                  :db/id #db/id[:db.part/db]
                  :db/ident :user/email
                  :db/valueType :db.type/string
                  :db/cardinality :db.cardinality/one
                  :db.install/_attribute :db.part/db}])

This field is of type `string`.  The name of the field is:
`:user/email`.  It should hold just one value, `:db.cardinality/one`.
Now we can load this into the database by transacting it like so:

    (d/transact @db-conn schema)

## Testdata<a id="sec-1-4" name="sec-1-4"></a>

Now we can actually start to load up a bit of testdata into the db.

    (def test-data
      [{:db/id #db/id[:db.part/user -1]
        :user/email "fenton.travers@gmail.com"}])

Whenever we add data into datomic we need to create and give the
entity a `:db/id`.

    #db/id[:db.part/user -1]

is the way we do this.  The -1 could be any negative number, and is
like our fake temporary id.  Datomic will, upon inserting this record
(entity/map), create the real permanent datomic id, `:db/id`.

## Blow away and recreate DB<a id="sec-1-5" name="sec-1-5"></a>

When experimenting with datomic, I like to blow the database away, so
I know I'm starting with a clean slate each time.

    (d/delete-database db-url)
    (d/create-database db-url)
    (reset! db-conn (d/connect db-url))
    (d/transact @db-conn schema)
    (d/transact @db-conn test-data)

Here I blow it away, recreate a blank DB, recreate the connection,
transact the schema and testdata.