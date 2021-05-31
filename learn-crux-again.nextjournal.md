# Learn Crux Again

Learn Crux Datalog Today is derived from the classic [learndatalogtoday.org](http://learndatalogtoday.org) tutorial materials, but adapted to focus on the [Crux](https://opencrux.com) API and unique Datalog semantics, and extended with additional topics and exercises. It is an interactive tutorial designed to teach you the Crux dialect of Datalog. Datalog is a declarative database query language with roots in logic programming. Datalog has similar expressive power to SQL.

You can follow along by hitting the "Remix" button on the toolbar to start your own interactive session running entirely on the Nextjournal platform, or you can copy the steps into a suitable Clojure REPL running locally on your machine. You could also use curl (or similar) to send the data and queries in this tutorial via HTTP to a pre-built Docker image.

The `master.md` source file for this notebook is available on [GitHub](https://github.com/crux-labs/learn-crux-datalog-today).

[Eclipse Public License - Version 1.0](https://github.com/crux-labs/learn-crux-datalog-today/blob/master/LICENSE.html)

© 2013 - 2016 Jonas Enlund

Thank you Jonas and contributors for freely licensing your excellent materials!

# Runtime Setup

You need to get Crux running before you can use it. Here we are using Clojure on the JVM and running Crux locally, as an embedded in-memory library. However, the Datalog query API is identical for all clients and all these queries would work equally well over the network via HTTP and the Java/Clojure client library.

```edn no-exec id=905a73c5-dbe6-4630-9e8b-f0f3b7baacea
{:deps
 {org.clojure/clojure {:mvn/version "1.10.0"}
  org.clojure/tools.deps.alpha
  {:git/url "https://github.com/clojure/tools.deps.alpha.git"
   :sha "f6c080bd0049211021ea59e516d1785b08302515"}
  juxt/crux-core {:mvn/version "RELEASE"}}}
```

```clojure id=b179eaf4-3fe6-4f9d-8b99-42027708b512
(require '[crux.api :as crux])
```

Next we need some data. Crux interprets maps as "documents." These require no pre-defined schema -- they only need a valid ID attribute. In the data below (which is defined using edn, and fully explained in the section) we are using negative integers as IDs. Any top-level attributes that refer to these integers can be interpreted in an ad-hoc way and traversed by the query engine -- this capability is known as "schema-on-read".

This vector of maps contains two kinds of documents: documents relating to people (actors and directors) and documents relating to movies. As a convention to aid human interpretation, all persons have IDs like `-1XX` and all movies have IDs like `-2XX`. Many ID value types are supported, such as strings and UUIDs, which may be more appropriate in a real application.

```clojure id=1212bacf-06ad-4e03-9a7f-ce5ff3606a3b
(def my-docs
  [{:person/name "James Cameron",
    :person/born #inst "1954-08-16T00:00:00.000-00:00",
    :crux.db/id -100}
   {:person/name "Arnold Schwarzenegger",
    :person/born #inst "1947-07-30T00:00:00.000-00:00",
    :crux.db/id -101}
   {:person/name "Linda Hamilton",
    :person/born #inst "1956-09-26T00:00:00.000-00:00",
    :crux.db/id -102}
   {:person/name "Michael Biehn",
    :person/born #inst "1956-07-31T00:00:00.000-00:00",
    :crux.db/id -103}
   {:person/name "Ted Kotcheff",
    :person/born #inst "1931-04-07T00:00:00.000-00:00",
    :crux.db/id -104}
   {:person/name "Sylvester Stallone",
    :person/born #inst "1946-07-06T00:00:00.000-00:00",
    :crux.db/id -105}
   {:person/name "Richard Crenna",
    :person/born #inst "1926-11-30T00:00:00.000-00:00",
    :person/death #inst "2003-01-17T00:00:00.000-00:00",
    :crux.db/id -106}
   {:person/name "Brian Dennehy",
    :person/born #inst "1938-07-09T00:00:00.000-00:00",
    :crux.db/id -107}
   {:person/name "John McTiernan",
    :person/born #inst "1951-01-08T00:00:00.000-00:00",
    :crux.db/id -108}
   {:person/name "Elpidia Carrillo",
    :person/born #inst "1961-08-16T00:00:00.000-00:00",
    :crux.db/id -109}
   {:person/name "Carl Weathers",
    :person/born #inst "1948-01-14T00:00:00.000-00:00",
    :crux.db/id -110}
   {:person/name "Richard Donner",
    :person/born #inst "1930-04-24T00:00:00.000-00:00",
    :crux.db/id -111}
   {:person/name "Mel Gibson",
    :person/born #inst "1956-01-03T00:00:00.000-00:00",
    :crux.db/id -112}
   {:person/name "Danny Glover",
    :person/born #inst "1946-07-22T00:00:00.000-00:00",
    :crux.db/id -113}
   {:person/name "Gary Busey",
    :person/born #inst "1944-07-29T00:00:00.000-00:00",
    :crux.db/id -114}
   {:person/name "Paul Verhoeven",
    :person/born #inst "1938-07-18T00:00:00.000-00:00",
    :crux.db/id -115}
   {:person/name "Peter Weller",
    :person/born #inst "1947-06-24T00:00:00.000-00:00",
    :crux.db/id -116}
   {:person/name "Nancy Allen",
    :person/born #inst "1950-06-24T00:00:00.000-00:00",
    :crux.db/id -117}
   {:person/name "Ronny Cox",
    :person/born #inst "1938-07-23T00:00:00.000-00:00",
    :crux.db/id -118}
   {:person/name "Mark L. Lester",
    :person/born #inst "1946-11-26T00:00:00.000-00:00",
    :crux.db/id -119}
   {:person/name "Rae Dawn Chong",
    :person/born #inst "1961-02-28T00:00:00.000-00:00",
    :crux.db/id -120}
   {:person/name "Alyssa Milano",
    :person/born #inst "1972-12-19T00:00:00.000-00:00",
    :crux.db/id -121}
   {:person/name "Bruce Willis",
    :person/born #inst "1955-03-19T00:00:00.000-00:00",
    :crux.db/id -122}
   {:person/name "Alan Rickman",
    :person/born #inst "1946-02-21T00:00:00.000-00:00",
    :crux.db/id -123}
   {:person/name "Alexander Godunov",
    :person/born #inst "1949-11-28T00:00:00.000-00:00",
    :person/death #inst "1995-05-18T00:00:00.000-00:00",
    :crux.db/id -124}
   {:person/name "Robert Patrick",
    :person/born #inst "1958-11-05T00:00:00.000-00:00",
    :crux.db/id -125}
   {:person/name "Edward Furlong",
    :person/born #inst "1977-08-02T00:00:00.000-00:00",
    :crux.db/id -126}
   {:person/name "Jonathan Mostow",
    :person/born #inst "1961-11-28T00:00:00.000-00:00",
    :crux.db/id -127}
   {:person/name "Nick Stahl",
    :person/born #inst "1979-12-05T00:00:00.000-00:00",
    :crux.db/id -128}
   {:person/name "Claire Danes",
    :person/born #inst "1979-04-12T00:00:00.000-00:00",
    :crux.db/id -129}
   {:person/name "George P. Cosmatos",
    :person/born #inst "1941-01-04T00:00:00.000-00:00",
    :person/death #inst "2005-04-19T00:00:00.000-00:00",
    :crux.db/id -130}
   {:person/name "Charles Napier",
    :person/born #inst "1936-04-12T00:00:00.000-00:00",
    :person/death #inst "2011-10-05T00:00:00.000-00:00",
    :crux.db/id -131}
   {:person/name "Peter MacDonald", :crux.db/id -132}
   {:person/name "Marc de Jonge",
    :person/born #inst "1949-02-16T00:00:00.000-00:00",
    :person/death #inst "1996-06-06T00:00:00.000-00:00",
    :crux.db/id -133}
   {:person/name "Stephen Hopkins", :crux.db/id -134}
   {:person/name "Ruben Blades",
    :person/born #inst "1948-07-16T00:00:00.000-00:00",
    :crux.db/id -135}
   {:person/name "Joe Pesci",
    :person/born #inst "1943-02-09T00:00:00.000-00:00",
    :crux.db/id -136}
   {:person/name "Ridley Scott",
    :person/born #inst "1937-11-30T00:00:00.000-00:00",
    :crux.db/id -137}
   {:person/name "Tom Skerritt",
    :person/born #inst "1933-08-25T00:00:00.000-00:00",
    :crux.db/id -138}
   {:person/name "Sigourney Weaver",
    :person/born #inst "1949-10-08T00:00:00.000-00:00",
    :crux.db/id -139}
   {:person/name "Veronica Cartwright",
    :person/born #inst "1949-04-20T00:00:00.000-00:00",
    :crux.db/id -140}
   {:person/name "Carrie Henn", :crux.db/id -141}
   {:person/name "George Miller",
    :person/born #inst "1945-03-03T00:00:00.000-00:00",
    :crux.db/id -142}
   {:person/name "Steve Bisley",
    :person/born #inst "1951-12-26T00:00:00.000-00:00",
    :crux.db/id -143}
   {:person/name "Joanne Samuel", :crux.db/id -144}
   {:person/name "Michael Preston",
    :person/born #inst "1938-05-14T00:00:00.000-00:00",
    :crux.db/id -145}
   {:person/name "Bruce Spence",
    :person/born #inst "1945-09-17T00:00:00.000-00:00",
    :crux.db/id -146}
   {:person/name "George Ogilvie",
    :person/born #inst "1931-03-05T00:00:00.000-00:00",
    :crux.db/id -147}
   {:person/name "Tina Turner",
    :person/born #inst "1939-11-26T00:00:00.000-00:00",
    :crux.db/id -148}
   {:person/name "Sophie Marceau",
    :person/born #inst "1966-11-17T00:00:00.000-00:00",
    :crux.db/id -149}
   {:movie/title "The Terminator",
    :movie/year 1984,
    :movie/director -100,
    :movie/cast [-101 -102 -103],
    :movie/sequel -207,
    :crux.db/id -200}
   {:movie/title "First Blood",
    :movie/year 1982,
    :movie/director -104,
    :movie/cast [-105 -106 -107],
    :movie/sequel -209,
    :crux.db/id -201}
   {:movie/title "Predator",
    :movie/year 1987,
    :movie/director -108,
    :movie/cast [-101 -109 -110],
    :movie/sequel -211,
    :crux.db/id -202}
   {:movie/title "Lethal Weapon",
    :movie/year 1987,
    :movie/director -111,
    :movie/cast [-112 -113 -114],
    :movie/sequel -212,
    :crux.db/id -203}
   {:movie/title "RoboCop",
    :movie/year 1987,
    :movie/director -115,
    :movie/cast [-116 -117 -118],
    :crux.db/id -204}
   {:movie/title "Commando",
    :movie/year 1985,
    :movie/director -119,
    :movie/cast [-101 -120 -121],
    :trivia
    "In 1986, a sequel was written with an eye to having\n  John McTiernan direct. Schwarzenegger wasn't interested in reprising\n  the role. The script was then reworked with a new central character,\n  eventually played by Bruce Willis, and became Die Hard",
    :crux.db/id -205}
   {:movie/title "Die Hard",
    :movie/year 1988,
    :movie/director -108,
    :movie/cast [-122 -123 -124],
    :crux.db/id -206}
   {:movie/title "Terminator 2: Judgment Day",
    :movie/year 1991,
    :movie/director -100,
    :movie/cast [-101 -102 -125 -126],
    :movie/sequel -208,
    :crux.db/id -207}
   {:movie/title "Terminator 3: Rise of the Machines",
    :movie/year 2003,
    :movie/director -127,
    :movie/cast [-101 -128 -129],
    :crux.db/id -208}
   {:movie/title "Rambo: First Blood Part II",
    :movie/year 1985,
    :movie/director -130,
    :movie/cast [-105 -106 -131],
    :movie/sequel -210,
    :crux.db/id -209}
   {:movie/title "Rambo III",
    :movie/year 1988,
    :movie/director -132,
    :movie/cast [-105 -106 -133],
    :crux.db/id -210}
   {:movie/title "Predator 2",
    :movie/year 1990,
    :movie/director -134,
    :movie/cast [-113 -114 -135],
    :crux.db/id -211}
   {:movie/title "Lethal Weapon 2",
    :movie/year 1989,
    :movie/director -111,
    :movie/cast [-112 -113 -136],
    :movie/sequel -213,
    :crux.db/id -212}
   {:movie/title "Lethal Weapon 3",
    :movie/year 1992,
    :movie/director -111,
    :movie/cast [-112 -113 -136],
    :crux.db/id -213}
   {:movie/title "Alien",
    :movie/year 1979,
    :movie/director -137,
    :movie/cast [-138 -139 -140],
    :movie/sequel -215,
    :crux.db/id -214}
   {:movie/title "Aliens",
    :movie/year 1986,
    :movie/director -100,
    :movie/cast [-139 -141 -103],
    :crux.db/id -215}
   {:movie/title "Mad Max",
    :movie/year 1979,
    :movie/director -142,
    :movie/cast [-112 -143 -144],
    :movie/sequel -217,
    :crux.db/id -216}
   {:movie/title "Mad Max 2",
    :movie/year 1981,
    :movie/director -142,
    :movie/cast [-112 -145 -146],
    :movie/sequel -218,
    :crux.db/id -217}
   {:movie/title "Mad Max Beyond Thunderdome",
    :movie/year 1985,
    :movie/director [-142 -147],
    :movie/cast [-112 -148],
    :crux.db/id -218}
   {:movie/title "Braveheart",
    :movie/year 1995,
    :movie/director [-112],
    :movie/cast [-112 -149],
    :crux.db/id -219}])
```

Note that Crux also has a JSON-over-HTTP API that naturally supports JSON documents, this is possible because JSON can be very simply mapped to a subset of edn.

To start an in-memory instance of Crux, you can use the `start-node` function like so:

```clojure id=cb9c4ca4-5912-45ea-b093-10749f1732c6
(def my-node (crux/start-node {}))
```

Loading the small amount of data we defined under `my-docs` above can be comfortably done in a single transaction. In practice you will often find benefit to batch `put` operations into groups of 1000 at a time. The following code maps over the docs to generate a single transaction containing one `:crux.tx/put` operation per document, then submits the transaction. Finally we call the `sync` function to ensure that the documents are fully indexed (and that the transaction has succeeded) before we attempt to run any queries -- this is necessary because of Crux's asynchronous design.

```clojure id=6892e0d1-25a7-4524-807a-12f18ddb2a05
(crux/submit-tx my-node (for [doc my-docs]
                          [:crux.tx/put doc]))

(crux/sync my-node)
```

With Crux running and the data loaded, you can now execute a query, which is a Clojure map, by passing it to Crux's `q` API, which takes the result of a `db` call as it's first value. The meaning of this query will become apparent very soon!

```clojure id=ef6fdcb3-61f5-46c6-adec-4770e7909c19
(crux/q (crux/db my-node)
        '{:find [title]
          :where [[_ :movie/title title]]})
```

To simplify this `crux/q` call throughout the rest of the tutorial we can define a new `q` function that saves us a few characters and visual clutter.

```clojure id=8e258be0-3446-469f-b3d0-1de626134fe6
(defn q [query & args]
  (apply crux/q (crux/db my-node) query args))
```

Queries can then be executed trivially:

```clojure id=1878c10d-919b-4981-9e34-bd678d580b15
(q '{:find [title]
     :where [[_ :movie/title title]]})
```

# Extensible Data Notation

In Crux, a Datalog query is written in [extensible data notation (edn)](http://edn-format.org). Edn is a data format similar to JSON, but it:

* is extensible with user defined value types,
* has more base types,
* is a subset of [Clojure](http://clojure.org) data.

Edn consists of:

* Numbers: `42`, `3.14159`
* Strings: `"This is a string"`
* Keywords: `:kw`, `:namespaced/keyword`, `:foo.bar/baz`
* Symbols: `max`, `+`, `title`, `?title`
* Vectors: `[1 2 3]` `[foo "bar" ?baz 123 ...]`
* Lists: `(3.14 :foo [:bar :baz])`, `(+ 1 2 3 4)`
* Instants: `#inst "2021-05-26"`
* .. and a few other things which we will not need in this tutorial.

Here is an example query that finds all movie titles in our example database:

```edn no-exec id=d24d0a60-306f-4f30-b2cc-13120942eb6b
    {:find [title]
     :where [[_ :movie/title title]]}
```

Note that the query is a map with two key-value pairs:

* the `:find` vector containing the symbol `title`
* the `:where` vector containing a single query clause `[_ :movie/title title]` (which is also a vector)

Crux also supports queries in an alternative "vector" format:

```edn no-exec id=77548e3e-0f4f-4a11-91d1-a338ca76ea40
    [:find ?title
     :where
     [_ :movie/title ?title]]
```

However, in this tutorial we will use the map format. Also note that Crux does not require logical variables to be preceded by `?`, although you may use this convention if you wish.

## Exercises

Q1. Find all the movie titles in the database

```edn no-exec id=b33ce3a1-469a-4113-9fa3-bf502f8939f1
{:find ...}
```

## Solutions

A1.

```clojure id=2a59d90e-d312-428b-96a7-8d0ac5b2fcc8
(q '{:find [title]
     :where [[_ :movie/title title]]})
```

# Basic Queries

The example database we'll use contains *movies* mostly, but not exclusively, from the 80s. You'll find information about movie titles, release year, directors, cast members, etc. As the tutorial advances we'll learn more about the contents of the database and how it's organized.

The data model in Crux is based around *atomic collections of facts.* Those atomic collections of facts are called *documents.* The facts are called triples. A triple is a 3-tuple consisting of:

* Entity ID
* Attribute
* Value

Although it is the document which is atomic in Crux (and not the triple), you can think of the database as a flat **set of triples** of the form:

```no-exec id=45ddd6b8-4d83-438c-9687-d872db77212a
[<e-id>  <attribute>      <value>          ]
...
[ -167    :person/name     "James Cameron" ]
[ -234    :movie/title     "Die Hard"      ]
[ -234    :movie/year      1987            ]
[ -235    :movie/title     "Terminator"    ]
[ -235    :movie/director  -167            ]
...
```

Note that the last two triples share the same entity ID, which means they are facts about the same movie (one *document*). Note also that the last triple's value is the same as the first triple's entity ID, i.e. the value of the `:movie/director` attribute is itself an entity.

A query is represented as a map with at least two key-value pairs. In the first pair, the key is the keyword `:find`, and the value is a vector of one or more **logic variables** (symbols, e.g. `?title` or `e`). The other key-value pair is the `:where` keyword key with a vector of clauses which restrict the query to triples that match the given **data patterns**.

For example, this query finds all entity-ids that have the attribute `:person/name` with a value of `"Ridley Scott"`:

```clojure id=50e60197-c540-44af-9549-233366cead8d
(q '{:find [e]
     :where [[e :person/name "Ridley Scott"]]})
```

The simplest data pattern is a triple with some parts replaced with logic variables. It is the job of the query engine to figure out every possible value of each of the logic variables and return the ones that are specified in the `:find` clause.

The symbol `_` can be used as a wildcard for the parts of the data pattern that you wish to ignore. You can also elide trailing values in a data pattern. Therefore, the following two queries are equivalent.

```clojure id=caefea90-0d89-4246-8fb1-a4675ada2122
(q '{:find [e]
     :where [[e :person/name _]]})
```

```clojure id=d2011551-baf4-4041-9c21-3443b640213d
(q '{:find [e]
     :where [[e :person/name]]})
```

## Exercises

Q1. Find the entity ids of movies made in 1987

```edn no-exec id=7d86753a-b2a1-4a73-8a84-70bc490677dd
{:find [e]
 :where ...}
```

Q2. Find the entity-id and titles of movies in the database

```edn no-exec id=2cd2a217-2d77-4f47-bb91-6b6489a93647
{:find [e title]
 :where ...}
```

Q3. Find the name of all people in the database

```edn no-exec id=7767efa8-4c92-40d6-b36e-d514f3ef0f02
{:find ...}
```

## Solutions

A1.

```clojure id=0bf5caf2-6c92-4277-9a24-73facc3a671f
(q '{:find [e]
     :where [[e :movie/year 1987]]})
```

A2.

```clojure id=708828b6-dac9-4691-8aa7-d1065be5d28e
(q '{:find [e]
     :where [[e :movie/title title]]})
```

A3.

```clojure id=16ea5d97-d22a-4451-8926-0088798f066a
(q '{:find [name]
     :where [[_ :person/name name]]})
```

# Data patterns

In the previous chapter, we looked at **data patterns**, i.e., vectors within the `:where` vector, such as `[e :movie/title "Commando"]`. There can be many data patterns in a `:where` clause:

```clojure id=8b3cb391-e62c-4484-a433-921f05b67323
(q '{:find [title]
     :where [[e :movie/year 1987]
             [e :movie/title title]]})
```

The important thing to note here is that the logic variable `e` is used in both data patterns. When a logic variable is used in multiple places, the query engine requires it to be bound to the same value in each place. Therefore, this query will only find movie titles for movies made in 1987.

The order of the data patterns does not matter. Crux ignores the user-provided clause ordering so the query engine can optimize query execution. Thus, the previous query could just as well have been written this way:

```clojure id=abece898-8f0a-42c8-9e5f-846667abf977
(q '{:find [title]
     :where [[e :movie/title title]
             [e :movie/year 1987]]})
```

In both cases, the result set will be exactly the same.

Let's say we want to find out who starred in "Lethal Weapon". We will need three data patterns for this. The first one finds the entity ID of the movie with "Lethal Weapon" as the title:

```no-exec id=b06adebb-e0ba-49bd-a989-c11a3fd916b2
[m :movie/title "Lethal Weapon"]
```

Using the same entity ID at `m`, we can find the cast members with the data pattern:

```no-exec id=161e975c-8a1a-4c97-8ce4-184447fb0085
[m :movie/cast p]
```

In this pattern, `p` will now be (the entity ID of) a person entity, so we can grab the actual name with:

```no-exec id=f8cf8a77-d65f-45f8-9b41-b1c9ace73e45
[p :person/name name]
```

The query will therefore be:

```no-exec id=4272a7c3-202b-4c42-92bf-1da7cc5b11e4
{:find [name]
 :where [[m :movie/title "Lethal Weapon"]
         [m :movie/cast p]
         [p :person/name name]]}
```

## Exercises

Q1. Find movie titles made in 1985

```edn no-exec id=329647c0-9c6a-4b8d-97a6-d886c8b92990
{:find [title]
 :where ...}
```

Q2. What year was "Alien" released?

```edn no-exec id=632b6260-08ef-4aab-bec4-18284f2ab2f9
{:find [year]
 :where ...}
```

Q3. Who directed RoboCop? You will need to use `[<movie-eid> :movie/director <person-eid>]` to find the director for a movie.

```edn no-exec id=ccc42ae8-df3a-4794-88ca-12d627b4b29d
{:find [name]
 :where ...}
```

Q4. Find directors who have directed Arnold Schwarzenegger in a movie.

```edn no-exec id=727fb427-2d91-422b-b4df-96bba5200812
{:find [name]
 :where ...}
```

## Solutions

A1.

```clojure id=b4d9b4d2-42ce-4f32-af43-638a8fc0cb29
(q '{:find [title]
     :where [[m :movie/title title]
             [m :movie/year 1985]]})
```

A2.

```clojure id=b19cbd15-7a43-422c-81d1-af0fd7ad2b8c
(q '{:find [year]
     :where [[m :movie/title "Alien"]
             [m :movie/year year]]})
```

A3.

```clojure id=7cf7186b-d7b5-4d97-8ac3-9fe4ab82a3e5
(q '{:find [name]
     :where [[m :movie/title "RoboCop"]
             [m :movie/director d]
             [d :person/name name]]})
```

A4.

```clojure id=47ee58bd-2360-475c-9aea-e4bf48194bde
(q '{:find [name]
     :where [[p :person/name "Arnold Schwarzenegger"]
             [m :movie/cast p]
             [m :movie/director d]
             [d :person/name name]]})
```

# Parameterized queries

Looking at this query:

```clojure id=b0105ca4-efce-4565-b12c-ebaa10392da4
(q '{:find [title]
     :where [[p :person/name "Sylvester Stallone"]
             [m :movie/cast p]
             [m :movie/title title]]})
```

It would be great if we could reuse this query to find movie titles for any actor and not just for "Sylvester Stallone". This is possible with an `:in` clause, which provides the query with input parameters, much in the same way that function or method arguments do in your programming language.

Here's that query with an input parameter for the actor:

```clojure id=c167727b-296d-4851-865e-6f0d911b17f0
(q '{:find [title]
     :in [name]
     :where [[p :person/name name]
             [m :movie/cast p]
             [m :movie/title title]]}
   "Sylvester Stallone")
```

This query takes one argument, `name`, which will be the name of some actor.

The above query is executed like `(q db query "Sylvester Stallone")`, where `query` is the query we just saw, and `db` is a database value. You can have any number of inputs to a query.

In the above query, the input logic variable `name` is bound to a scalar - a string in this case. There are four different kinds of input: scalars, tuples, collections, and relations.

## A quick aside

Note that an implicit first argument `$` is also available, should it ever be needed, which is the value of database `db` itself. You can use this in conjunction with more advanced features like subqueries and custom Clojure predicates (more on those later!).

## Tuples

A tuple input is written as e.g. `[name age]` and can be used when you want to destructure an input. Let's say you have the vector `["James Cameron" "Arnold Schwarzenegger"]` and you want to use this as input to find all movies where these two people collaborated:

```clojure id=6360b09f-7981-4238-8b78-d6fd765ae964
(q '{:find [title]
     :in [[director actor]]
     :where [[d :person/name director]
             [a :person/name actor]
             [m :movie/director d]
             [m :movie/cast a]
             [m :movie/title title]]}
   ["James Cameron" "Arnold Schwarzenegger"])
```

Of course, in this case, you could just as well use two distinct inputs instead:

```no-exec id=7ef44683-c204-4c81-a146-e9f9c7790ddb
:in [director actor]
```

## Collections

You can use collection destructuring to implement a kind of *logical OR* in your query. Say you want to find all movies directed by either James Cameron **or** Ridley Scott:

```clojure id=1d918310-4772-4c7d-888e-43b0b014f330
(q '{:find [title]
     :in [[director ...]]
     :where [[p :person/name director]
             [m :movie/director p]
             [m :movie/title title]]}
   ["James Cameron" "Ridley Scott"])
```

Here, the `director` logic variable is initially bound to both "James Cameron" and "Ridley Scott". Note that the ellipsis following `director` is a literal, not elided code.

## Relations

Relations - a set of tuples - are the most interesting and powerful of input types, since you can join external relations with the triples in your database.

As a simple example, let's consider a relation with tuples `[movie-title box-office-earnings]`:

```no-exec id=3e773974-6168-4776-9d56-4b8c7f8fe348
[
 ...
 ["Die Hard" 140700000]
 ["Alien" 104931801]
 ["Lethal Weapon" 120207127]
 ["Commando" 57491000]
 ...
]
```

Let's use this data and the data in our database to find box office earnings for a particular director:

```clojure id=26bbfaa9-bd9b-4458-988b-7b5b5d68cb8e
(q '{:find [title box-office]
     :in [director [[title box-office]]]
     :where [[p :person/name director]
             [m :movie/director p]
             [m :movie/title title]]}
   "Ridley Scott"
   [["Die Hard" 140700000]
     ["Alien" 104931801]
     ["Lethal Weapon" 120207127]
     ["Commando" 57491000]])
```

Note that the `box-office` logic variable does not appear in any of the data patterns in the `:where` clause.

## Exercises

Q1. Find movie title by year

```edn no-exec id=8142ffc8-364c-43d6-ac66-114e4cf6454a
{:find [title]
 :in [year]
 :where ...}
```

Q2. Given a list of movie titles, find the title and the year that movie was released.

```edn no-exec id=f8294e3d-5a0a-48ee-a108-c7950ab1488a
{:find [title year]
 :in ...
 :where ...}
```

Q3. Find all movie `title`s where the `actor` and the `director` has worked together

```edn no-exec id=7ed7180a-622e-423b-bfaa-9714860090ed
{:find [title]
 :in [actor director]
 :where ...}
```

Q4. Write a query that, given an actor name and a relation with movie-title/rating, finds the movie titles and corresponding rating for which that actor was a cast member.

```edn no-exec id=f35c6fbc-a3a1-4c5a-898e-b502bf0707ef
{:find [title rating]
 :in ...
 :where ...}
```

## Solutions

A1.

```clojure id=6603b044-f33b-4616-8500-526655d86ffa
(q '{:find [title]
     :in [year]
     :where [[m :movie/year year]
             [m :movie/title title]]}
   1988)
```

A2.

```clojure id=a5c1f29c-08ba-4b57-a714-6bc98a60427f
(q '{:find [title year]
     :in [[title ...]]
     :where [[m :movie/title title]
             [m :movie/year year]]}
   ["Lethal Weapon" "Lethal Weapon 2" "Lethal Weapon 3"])
```

A3.

```clojure id=e4f1c4f0-6440-4e01-bdfb-670d6cc1aed0
(q '{:find [title]
     :in [actor director]
     :where [[a :person/name actor]
             [d :person/name director]
             [m :movie/cast a]
             [m :movie/director d]
             [m :movie/title title]]}
   "Michael Biehn"
   "James Cameron")
```

A4.

```clojure id=a92cfd0e-49e1-4edd-9e62-d301e552bdcb
(q '{:find [title rating]
     :in [name [[title rating]]]
     :where [[p :person/name name]
             [m :movie/cast p]
             [m :movie/title title]]}
   "Mel Gibson"
   [["Die Hard" 8.3]
    ["Alien" 8.5]
    ["Lethal Weapon" 7.6]
    ["Commando" 6.5]
    ["Mad Max Beyond Thunderdome" 6.1]
    ["Mad Max 2" 7.6]
    ["Rambo: First Blood Part II" 6.2]
    ["Braveheart" 8.4]
    ["Terminator 2: Judgment Day" 8.6]
    ["Predator 2" 6.1]
    ["First Blood" 7.6]
    ["Aliens" 8.5]
    ["Terminator 3: Rise of the Machines" 6.4]
    ["Rambo III" 5.4]
    ["Mad Max" 7.0]
    ["The Terminator" 8.1]
    ["Lethal Weapon 2" 7.1]
    ["Predator" 7.8]
    ["Lethal Weapon 3" 6.6]
    ["RoboCop" 7.5]])
```

# Predicates

So far, we have only been dealing with **data patterns**: `[m :movie/year year]`. We have not yet seen a proper way of handling questions like "*Find all movies released before 1984*". This is where **predicate clauses** come into play.

Let's start with the query for the question above:

```clojure id=0dace1c8-8008-4bc3-829b-2f9a3f640ae9
(q '{:find [title]
     :where [[m :movie/title title]
             [m :movie/year year]
             [(< year 1984)]]})
```

The last clause, `[(< year 1984)]`, is a predicate clause. The predicate clause filters the result set to only include results for which the predicate returns a "truthy" (non-nil, non-false) value. You can use any Clojure function as a predicate function:

```clojure id=5df6a76e-c47f-4aab-bd03-5f244923d4ad
(q '{:find [name]
     :where [[p :person/name name]
             [(clojure.string/starts-with? name "M")]]})
```

Clojure functions must be fully namespace-qualified, so if you have defined your own predicate `awesome?` you must write it as `(my.namespace/awesome? movie)`. All `clojure.core/*` functions may be used as predicates without namespace qualification: `<, >, <=, >=, =, not=` and so on. Crux provides a ["Predicate AllowList"](https://opencrux.com/reference/query-configuration.html#fn-allowlist) feature to restrict the exact set of predicates available to queries.

## Exercises

Q1. Find movies older than a certain year (inclusive)

```edn no-exec id=269fece8-7072-42ee-8f7b-3c3fd9429e79
{:find [title]
 :in [year]
 :where ...}
```

Q2. Find actors older than Danny Glover

```edn no-exec id=ae07508d-df0d-4a59-bde6-607bb3437bfd
{:find [actor]
 :where ...}
```

Q3. Find movies newer than `year` (inclusive) and has a `rating` higher than the one supplied

```edn no-exec id=7b1dbb98-f369-4b7f-bf25-97bc6bf2e16f
{:find [title]
 :in [year rating [[title r]]]
 :where ...}
```

## Solutions

A1.

```clojure id=ce124c08-d13b-4bbe-9f6f-d994f77cf2ca
(q '{:find [title]
     :in [year]
     :where [[m :movie/title title]
             [m :movie/year y]
             [(<= y year)]]}
   1979)
```

A2.

```clojure id=b8937176-f6d7-4370-aca2-35ed8b725358
(q '{:find [actor]
     :where [[d :person/name "Danny Glover"]
             [d :person/born b1]
             [e :person/born b2]
             [_ :movie/cast e]
             [(< b2 b1)]
             [e :person/name actor]]})
```

A3.

```clojure id=91aa9f17-53e6-47e9-ab36-2c34bea001a8
(q '{:find [title]
     :in [year rating [[title r]]]
     :where [[(< rating r)]
             [m :movie/title title]
             [m :movie/year y]
             [(<= year y)]]}
   1990
   8.0
   [["Die Hard" 8.3]
    ["Alien" 8.5]
    ["Lethal Weapon" 7.6]
    ["Commando" 6.5]
    ["Mad Max Beyond Thunderdome" 6.1]
    ["Mad Max 2" 7.6]
    ["Rambo: First Blood Part II" 6.2]
    ["Braveheart" 8.4]
    ["Terminator 2: Judgment Day" 8.6]
    ["Predator 2" 6.1]
    ["First Blood" 7.6]
    ["Aliens" 8.5]
    ["Terminator 3: Rise of the Machines" 6.4]
    ["Rambo III" 5.4]
    ["Mad Max" 7.0]
    ["The Terminator" 8.1]
    ["Lethal Weapon 2" 7.1]
    ["Predator" 7.8]
    ["Lethal Weapon 3" 6.6]
    ["RoboCop" 7.5]])
```

# Transformation functions

**Transformation functions** are pure (side-effect free) functions which can be used in queries as "function expression" predicates to transform values and bind their results to new logic variables. Say, for example, there exists an attribute `:person/born` with type `:db.type/instant`. Given the birthday, it's easy to calculate the (very approximate) age of a person:

```clojure id=17837f57-61d5-4e12-9892-17c23790ef60
(defn age [^java.util.Date birthday ^java.util.Date today]
  (quot (- (.getTime today)
          (.getTime birthday))
        (* 1000 60 60 24 365)))
```

With this function, we can now calculate the age of a person **inside the query itself**:

```clojure id=9272ca34-36bb-4d1e-b0cb-33dbc3c5e5f7
(q '{:find [age]
     :in [name today]
     :where [[p :person/name name]
             [p :person/born born]
             [(user/age born today) age]]}
   "Tina Turner"
   (java.util.Date.))
```

A transformation function clause has the shape `[(<fn> <arg1> <arg2> ...) <result-binding>]` where `<result-binding>` can be the same binding forms as we saw in [chapter 3](/chapter/3):

* Scalar: `age`
* Tuple: `[foo bar baz]`
* Collection: `[name ...]`
* Relation: `[[title rating]]`

One thing to be aware of is that transformation functions can't be nested. For example, you can't write:

```no-exec id=991cf4a0-1983-453e-b15a-8c70195b4198
[(f (g x)) a]
```

Instead, you must bind intermediate results in temporary logic variables:

```no-exec id=4b272a39-ce69-4b34-b46b-7308e5e2ef7e
[(g x) t]
[(f t) a]
```

## Exercises

Q1. Find people by age. Use the function `user/age` to find the age given a birthday and a date representing "today".

```edn no-exec id=06bcaef5-5991-45f9-aeba-b2635c0070ce
{:find [name]
 :in [age today]
 :where ...}
```

Q2. Find people younger than Bruce Willis and their ages.

```edn no-exec id=ba4fbe14-3d9d-436e-9409-1a13f2a72dbe
{:find [name age]
 :in [today]
 :where ...}
```

## Solutions

A1.

```clojure id=873d636b-878e-439a-95e7-d49795e24d72
(q '{:find [name]
     :in [age today]
     :where [[p :person/name name]
             [p :person/born born]
             [(user/age born today) age]]}
   63
   #inst "2013-08-02T00:00:00.000-00:00")
```

A2.

```clojure id=219c4550-d23b-48a1-8d2f-2dfe37a120ce
(q '{:find [name age]
     :in [today]
     :where [[p :person/name "Bruce Willis"]
             [p :person/born sborn]
             [p2 :person/name name]
             [p2 :person/born born]
             [(< sborn born)]
             [(user/age born today) age]]}
   #inst "2013-08-02T00:00:00.000-00:00")
```

# Aggregates

Aggregate functions such as `sum`, `max` etc. are readily available in Crux's Datalog implementation. They are written in the `:find` clause in your query:

```no-exec id=34903538-f626-4f08-ab27-6336b57bc349
{:find [(max date)]
 :where
 ...}
```

An aggregate function collects values from multiple triples and returns

* A single value: `min`, `max`, `sum`, `avg`, etc.
* A collection of values: `(min n d)` `(max n d)` `(sample n e)` etc. where `n` is an integer specifying the size of the collection.

## Exercises

Q1. `count` the number of movies in the database

Q2. Find the birth date of the oldest person in the database.

Q3. Given a collection of actors and (the now familiar) ratings data. Find the average rating for each actor. The query should return the actor name and the `avg` rating.

## Solutions

A1.

```clojure id=06cb79d1-2f7a-4df8-a9b0-7cebdc72cbaa
(q '{:find [(count m)]
     :where [[m :movie/title]]})
```

A2.

```clojure id=abe86f78-a450-4de4-b31f-d0d394e4e3b7
(q '{:find [(min date)]
     :where [[_ :person/born date]]})
```

A3.

```clojure id=ece537a0-482f-415e-b217-4f13685e32c3
(q '{:find [name (avg rating)]
     :in [[name ...] [[title rating]]]
     :where [[p :person/name name]
             [m :movie/cast p]
             [m :movie/title title]]}
   ["Sylvester Stallone" "Arnold Schwarzenegger" "Mel Gibson"]
   [["Die Hard" 8.3]
    ["Alien" 8.5]
    ["Lethal Weapon" 7.6]
    ["Commando" 6.5]
    ["Mad Max Beyond Thunderdome" 6.1]
    ["Mad Max 2" 7.6]
    ["Rambo: First Blood Part II" 6.2]
    ["Braveheart" 8.4]
    ["Terminator 2: Judgment Day" 8.6]
    ["Predator 2" 6.1]
    ["First Blood" 7.6]
    ["Aliens" 8.5]
    ["Terminator 3: Rise of the Machines" 6.4]
    ["Rambo III" 5.4]
    ["Mad Max" 7.0]
    ["The Terminator" 8.1]
    ["Lethal Weapon 2" 7.1]
    ["Predator" 7.8]
    ["Lethal Weapon 3" 6.6]
    ["RoboCop" 7.5]])
```

# Rules

Many times over the course of this tutorial, we have had to write the following three lines of repetitive query code:

```no-exec id=c8a1cf62-a8eb-4ce4-bc7e-88733e109bdc
[p :person/name name]
[m :movie/cast p]
[m :movie/title title]
```

**Rules** are the means of abstraction in Datalog. You can abstract away reusable parts of your queries into rules, give them meaningful names and forget about the implementation details, just like you can with functions in your favorite programming language. Let's create a rule for the three lines above:

```no-exec id=4ff59332-2846-40d2-8f59-89eb9d18f60e
[(actor-movie name title)
 [p :person/name name]
 [m :movie/cast p]
 [m :movie/title title]]
```

The first vector is called the *head* of the rule where the first symbol is the name of the rule. The rest of the rule is called the *body*.

You can think of a rule as a kind of function, but remember that this is logic programming, so we can use the same rule to:

* find movie titles given an actor name, and
* find actor names given a movie title.

Put another way, we can use both `name` and `title` in `(actor-movie name title)` for input as well as for output. If we provide values for neither, we'll get all the possible combinations in the database. If we provide values for one or both, it will constrain the result returned by the query as you'd expect.

To use the above rule, you simply write the head of the rule instead of the data patterns. Any variable with values already bound will be input, the rest will be output.

The query to find cast members of some movie, for which we previously had to write:

```clojure id=82fce931-d3e0-4fd3-8b2b-164fc9ff695d
(q '{:find [name]
     :where [[p :person/name name]
             [m :movie/cast p]
             [m :movie/title "The Terminator"]]})
```

Now becomes:

```clojure id=9c7a89cf-ecc1-42e6-8fba-00cd7e914cab
(q '{:find [name]
     :where [(actor-movie name "The Terminator")]
     :rules [[(actor-movie name title)
              [p :person/name name]
              [m :movie/cast p]
              [m :movie/title "The Terminator"]]]})
```

You can write any number of rules, collect them in a vector, and pass them to the query engine using the `:rules` key, as above.

```no-exec id=4df0ad34-581e-4c93-9fb4-e24929a5b7ca
[[(rule-a a b)
  ...]
 [(rule-b a b)
  ...]
 ...]
```

You can use [data patterns](/chapter/2), [predicates](/chapter/5), [transformation functions](/chapter/6) and calls to other rules in the body of a rule.

Rules can also be used as another tool to write *logical OR* queries, as the same rule name can be used several times:

```no-exec id=d457874a-2e62-49a5-9512-5f31008b4b06
[[(associated-with person movie)
  [movie :movie/cast person]]
 [(associated-with person movie)
  [movie :movie/director person]]]
```

Subsequent rule definitions will only be used if the ones preceding it aren't satisfied.

Using this rule, we can find both directors and cast members very easily:

```clojure id=9928e0f4-fbf2-4de5-abfa-56a13fa29122
(q '{:find [name]
              :where [[m :movie/title "Predator"]
                      (associated-with p m)
                      [p :person/name name]]
              :rules [[(associated-with person movie)
                       [movie :movie/cast person]]
                      [(associated-with person movie)
                       [movie :movie/director person]]]})
```

Given the fact that rules can contain calls to other rules, what would happen if a rule called itself? Interesting things, it turns out, but let's find out in the exercises.

## Exercises

Q1. Write a rule `(movie-year title year)` where `title` is the title of some movie and `year` is that movie's release year.

```edn no-exec id=d005a0af-0d2d-4a5c-a060-8747158523db
{:find [title]
 :where [(movie-year title 1991)]
 :rules [[(movie-year title year)
          ...]]}
```

Q2. Two people are friends if they have worked together in a movie. Write a rule `(friends p1 p2)` where `p1` and `p2` are person entities. Try with a few different `name` inputs to make sure you got it right. There might be some edge cases here.

```edn no-exec id=eccda521-8c43-4234-be30-ef7b50325f60
{:find [friend]
 :in [name]
 :where [[p1 :person/name name]
         (friends p1 p2)
         [p2 :person/name friend]]
 :rules [[(friends p1 p2)
          ...]]}
```

## Solutions

A1.

```clojure id=942f4375-f8f7-4807-b121-9b9cf177a668
(q '{:find [title]
     :where [(movie-year title 1991)]
     :rules [[(movie-year title year)
              [m :movie/title title]
              [m :movie/year year]]]})
```

A2.

```clojure id=1a3b51e5-1b93-48b5-a009-a0911c8a50c8
(q '{:find [friend]
     :in [name]
     :where [[p1 :person/name name]
             (friends p1 p2)
             [p2 :person/name friend]]
     :rules [[(friends ?p1 ?p2)
              [?m :movie/cast ?p1]
              [?m :movie/cast ?p2]
              [(not= ?p1 ?p2)]]
             [(friends ?p1 ?p2)
              [?m :movie/cast ?p1]
              [?m :movie/director ?p2]]
             [(friends ?p1 ?p2)
              (friends ?p2 ?p1)]]}
    "Sigourney Weaver")
```

# Conclusion

Congratulations for making it through the tutorial - we hope this knowledge helps you in your Datalog journey! Any and all feedback is appreciated, as are new contributions, please email [crux@juxt.pro](mailto:crux@juxt.pro) or open an issue via [GitHub](https://github.com/crux-labs/learn-crux-datalog-today)

# Appendix

[github-repository][nextjournal#github-repository#7639668f-682f-4a6b-bc3f-925ff3043dfe]


[nextjournal#github-repository#7639668f-682f-4a6b-bc3f-925ff3043dfe]:
<https://github.com/crux-labs/learn-crux-datalog-today>

<details id="com.nextjournal.article">
<summary>This notebook was exported from <a href="https://nextjournal.com/a/6Hhhn7nV9yRGFLvpyQ9xLM?change-id=Cwcc5a3k5vqdrHwEJDaGpU">https://nextjournal.com/a/6Hhhn7nV9yRGFLvpyQ9xLM?change-id=Cwcc5a3k5vqdrHwEJDaGpU</a></summary>

```edn nextjournal-metadata
{:article
 {:nodes
  {"06bcaef5-5991-45f9-aeba-b2635c0070ce"
   {:id "06bcaef5-5991-45f9-aeba-b2635c0070ce", :kind "code-listing"},
   "06cb79d1-2f7a-4df8-a9b0-7cebdc72cbaa"
   {:id "06cb79d1-2f7a-4df8-a9b0-7cebdc72cbaa",
    :kind "code",
    :runtime [:runtime "72545a3d-110b-44f0-82ff-8c8e0a0824a9"]},
   "0bf5caf2-6c92-4277-9a24-73facc3a671f"
   {:id "0bf5caf2-6c92-4277-9a24-73facc3a671f",
    :kind "code",
    :runtime [:runtime "72545a3d-110b-44f0-82ff-8c8e0a0824a9"]},
   "0dace1c8-8008-4bc3-829b-2f9a3f640ae9"
   {:id "0dace1c8-8008-4bc3-829b-2f9a3f640ae9",
    :kind "code",
    :runtime [:runtime "72545a3d-110b-44f0-82ff-8c8e0a0824a9"]},
   "1212bacf-06ad-4e03-9a7f-ce5ff3606a3b"
   {:id "1212bacf-06ad-4e03-9a7f-ce5ff3606a3b",
    :kind "code",
    :runtime [:runtime "72545a3d-110b-44f0-82ff-8c8e0a0824a9"]},
   "161e975c-8a1a-4c97-8ce4-184447fb0085"
   {:id "161e975c-8a1a-4c97-8ce4-184447fb0085", :kind "code-listing"},
   "16ea5d97-d22a-4451-8926-0088798f066a"
   {:id "16ea5d97-d22a-4451-8926-0088798f066a",
    :kind "code",
    :runtime [:runtime "72545a3d-110b-44f0-82ff-8c8e0a0824a9"]},
   "17837f57-61d5-4e12-9892-17c23790ef60"
   {:id "17837f57-61d5-4e12-9892-17c23790ef60",
    :kind "code",
    :runtime [:runtime "72545a3d-110b-44f0-82ff-8c8e0a0824a9"]},
   "1878c10d-919b-4981-9e34-bd678d580b15"
   {:id "1878c10d-919b-4981-9e34-bd678d580b15",
    :kind "code",
    :runtime [:runtime "72545a3d-110b-44f0-82ff-8c8e0a0824a9"]},
   "1a3b51e5-1b93-48b5-a009-a0911c8a50c8"
   {:id "1a3b51e5-1b93-48b5-a009-a0911c8a50c8",
    :kind "code",
    :runtime [:runtime "72545a3d-110b-44f0-82ff-8c8e0a0824a9"]},
   "1d918310-4772-4c7d-888e-43b0b014f330"
   {:id "1d918310-4772-4c7d-888e-43b0b014f330",
    :kind "code",
    :runtime [:runtime "72545a3d-110b-44f0-82ff-8c8e0a0824a9"]},
   "219c4550-d23b-48a1-8d2f-2dfe37a120ce"
   {:id "219c4550-d23b-48a1-8d2f-2dfe37a120ce",
    :kind "code",
    :runtime [:runtime "72545a3d-110b-44f0-82ff-8c8e0a0824a9"]},
   "269fece8-7072-42ee-8f7b-3c3fd9429e79"
   {:id "269fece8-7072-42ee-8f7b-3c3fd9429e79", :kind "code-listing"},
   "26bbfaa9-bd9b-4458-988b-7b5b5d68cb8e"
   {:id "26bbfaa9-bd9b-4458-988b-7b5b5d68cb8e",
    :kind "code",
    :runtime [:runtime "72545a3d-110b-44f0-82ff-8c8e0a0824a9"]},
   "2a59d90e-d312-428b-96a7-8d0ac5b2fcc8"
   {:id "2a59d90e-d312-428b-96a7-8d0ac5b2fcc8",
    :kind "code",
    :runtime [:runtime "72545a3d-110b-44f0-82ff-8c8e0a0824a9"]},
   "2cd2a217-2d77-4f47-bb91-6b6489a93647"
   {:id "2cd2a217-2d77-4f47-bb91-6b6489a93647", :kind "code-listing"},
   "329647c0-9c6a-4b8d-97a6-d886c8b92990"
   {:id "329647c0-9c6a-4b8d-97a6-d886c8b92990", :kind "code-listing"},
   "34903538-f626-4f08-ab27-6336b57bc349"
   {:id "34903538-f626-4f08-ab27-6336b57bc349", :kind "code-listing"},
   "3e773974-6168-4776-9d56-4b8c7f8fe348"
   {:id "3e773974-6168-4776-9d56-4b8c7f8fe348", :kind "code-listing"},
   "4272a7c3-202b-4c42-92bf-1da7cc5b11e4"
   {:id "4272a7c3-202b-4c42-92bf-1da7cc5b11e4", :kind "code-listing"},
   "45ddd6b8-4d83-438c-9687-d872db77212a"
   {:id "45ddd6b8-4d83-438c-9687-d872db77212a", :kind "code-listing"},
   "47ee58bd-2360-475c-9aea-e4bf48194bde"
   {:id "47ee58bd-2360-475c-9aea-e4bf48194bde",
    :kind "code",
    :runtime [:runtime "72545a3d-110b-44f0-82ff-8c8e0a0824a9"]},
   "4b272a39-ce69-4b34-b46b-7308e5e2ef7e"
   {:id "4b272a39-ce69-4b34-b46b-7308e5e2ef7e", :kind "code-listing"},
   "4df0ad34-581e-4c93-9fb4-e24929a5b7ca"
   {:id "4df0ad34-581e-4c93-9fb4-e24929a5b7ca", :kind "code-listing"},
   "4ff59332-2846-40d2-8f59-89eb9d18f60e"
   {:id "4ff59332-2846-40d2-8f59-89eb9d18f60e", :kind "code-listing"},
   "50e60197-c540-44af-9549-233366cead8d"
   {:id "50e60197-c540-44af-9549-233366cead8d",
    :kind "code",
    :runtime [:runtime "72545a3d-110b-44f0-82ff-8c8e0a0824a9"]},
   "5df6a76e-c47f-4aab-bd03-5f244923d4ad"
   {:id "5df6a76e-c47f-4aab-bd03-5f244923d4ad",
    :kind "code",
    :runtime [:runtime "72545a3d-110b-44f0-82ff-8c8e0a0824a9"]},
   "632b6260-08ef-4aab-bec4-18284f2ab2f9"
   {:id "632b6260-08ef-4aab-bec4-18284f2ab2f9", :kind "code-listing"},
   "6360b09f-7981-4238-8b78-d6fd765ae964"
   {:id "6360b09f-7981-4238-8b78-d6fd765ae964",
    :kind "code",
    :runtime [:runtime "72545a3d-110b-44f0-82ff-8c8e0a0824a9"]},
   "6603b044-f33b-4616-8500-526655d86ffa"
   {:id "6603b044-f33b-4616-8500-526655d86ffa",
    :kind "code",
    :runtime [:runtime "72545a3d-110b-44f0-82ff-8c8e0a0824a9"]},
   "6892e0d1-25a7-4524-807a-12f18ddb2a05"
   {:id "6892e0d1-25a7-4524-807a-12f18ddb2a05",
    :kind "code",
    :runtime [:runtime "72545a3d-110b-44f0-82ff-8c8e0a0824a9"]},
   "708828b6-dac9-4691-8aa7-d1065be5d28e"
   {:id "708828b6-dac9-4691-8aa7-d1065be5d28e",
    :kind "code",
    :runtime [:runtime "72545a3d-110b-44f0-82ff-8c8e0a0824a9"]},
   "72545a3d-110b-44f0-82ff-8c8e0a0824a9"
   {:environment
    [:environment
     {:article/nextjournal.id
      #uuid "5b45eb52-bad4-413d-9d7f-b2b573a25322",
      :change/nextjournal.id
      #uuid "5f045c36-90bd-428b-a26c-b59fa0a2e1db",
      :node/id "0ae15688-6f6a-40e2-a4fa-52d81371f733"}],
    :id "72545a3d-110b-44f0-82ff-8c8e0a0824a9",
    :kind "runtime",
    :language "clojure",
    :type :nextjournal,
    :runtime/mounts
    [{:src [:node "7639668f-682f-4a6b-bc3f-925ff3043dfe"],
      :dest "/learn-crux-datalog-today"}
     {:src [:node "905a73c5-dbe6-4630-9e8b-f0f3b7baacea"],
      :dest "/learn-crux-datalog-today/deps.edn"}]},
   "727fb427-2d91-422b-b4df-96bba5200812"
   {:id "727fb427-2d91-422b-b4df-96bba5200812", :kind "code-listing"},
   "7639668f-682f-4a6b-bc3f-925ff3043dfe"
   {:id "7639668f-682f-4a6b-bc3f-925ff3043dfe",
    :kind "github-repository"},
   "77548e3e-0f4f-4a11-91d1-a338ca76ea40"
   {:id "77548e3e-0f4f-4a11-91d1-a338ca76ea40", :kind "code-listing"},
   "7767efa8-4c92-40d6-b36e-d514f3ef0f02"
   {:id "7767efa8-4c92-40d6-b36e-d514f3ef0f02", :kind "code-listing"},
   "7b1dbb98-f369-4b7f-bf25-97bc6bf2e16f"
   {:id "7b1dbb98-f369-4b7f-bf25-97bc6bf2e16f", :kind "code-listing"},
   "7cf7186b-d7b5-4d97-8ac3-9fe4ab82a3e5"
   {:id "7cf7186b-d7b5-4d97-8ac3-9fe4ab82a3e5",
    :kind "code",
    :runtime [:runtime "72545a3d-110b-44f0-82ff-8c8e0a0824a9"]},
   "7d86753a-b2a1-4a73-8a84-70bc490677dd"
   {:id "7d86753a-b2a1-4a73-8a84-70bc490677dd", :kind "code-listing"},
   "7ed7180a-622e-423b-bfaa-9714860090ed"
   {:id "7ed7180a-622e-423b-bfaa-9714860090ed", :kind "code-listing"},
   "7ef44683-c204-4c81-a146-e9f9c7790ddb"
   {:id "7ef44683-c204-4c81-a146-e9f9c7790ddb", :kind "code-listing"},
   "8142ffc8-364c-43d6-ac66-114e4cf6454a"
   {:id "8142ffc8-364c-43d6-ac66-114e4cf6454a", :kind "code-listing"},
   "82fce931-d3e0-4fd3-8b2b-164fc9ff695d"
   {:id "82fce931-d3e0-4fd3-8b2b-164fc9ff695d",
    :kind "code",
    :runtime [:runtime "72545a3d-110b-44f0-82ff-8c8e0a0824a9"]},
   "873d636b-878e-439a-95e7-d49795e24d72"
   {:id "873d636b-878e-439a-95e7-d49795e24d72",
    :kind "code",
    :runtime [:runtime "72545a3d-110b-44f0-82ff-8c8e0a0824a9"]},
   "8b3cb391-e62c-4484-a433-921f05b67323"
   {:id "8b3cb391-e62c-4484-a433-921f05b67323",
    :kind "code",
    :runtime [:runtime "72545a3d-110b-44f0-82ff-8c8e0a0824a9"]},
   "8e258be0-3446-469f-b3d0-1de626134fe6"
   {:id "8e258be0-3446-469f-b3d0-1de626134fe6",
    :kind "code",
    :runtime [:runtime "72545a3d-110b-44f0-82ff-8c8e0a0824a9"]},
   "905a73c5-dbe6-4630-9e8b-f0f3b7baacea"
   {:id "905a73c5-dbe6-4630-9e8b-f0f3b7baacea",
    :kind "code-listing",
    :name "deps.edn"},
   "91aa9f17-53e6-47e9-ab36-2c34bea001a8"
   {:id "91aa9f17-53e6-47e9-ab36-2c34bea001a8",
    :kind "code",
    :runtime [:runtime "72545a3d-110b-44f0-82ff-8c8e0a0824a9"]},
   "9272ca34-36bb-4d1e-b0cb-33dbc3c5e5f7"
   {:id "9272ca34-36bb-4d1e-b0cb-33dbc3c5e5f7",
    :kind "code",
    :runtime [:runtime "72545a3d-110b-44f0-82ff-8c8e0a0824a9"]},
   "942f4375-f8f7-4807-b121-9b9cf177a668"
   {:id "942f4375-f8f7-4807-b121-9b9cf177a668",
    :kind "code",
    :runtime [:runtime "72545a3d-110b-44f0-82ff-8c8e0a0824a9"]},
   "991cf4a0-1983-453e-b15a-8c70195b4198"
   {:id "991cf4a0-1983-453e-b15a-8c70195b4198", :kind "code-listing"},
   "9928e0f4-fbf2-4de5-abfa-56a13fa29122"
   {:id "9928e0f4-fbf2-4de5-abfa-56a13fa29122",
    :kind "code",
    :runtime [:runtime "72545a3d-110b-44f0-82ff-8c8e0a0824a9"]},
   "9c7a89cf-ecc1-42e6-8fba-00cd7e914cab"
   {:id "9c7a89cf-ecc1-42e6-8fba-00cd7e914cab",
    :kind "code",
    :runtime [:runtime "72545a3d-110b-44f0-82ff-8c8e0a0824a9"]},
   "a5c1f29c-08ba-4b57-a714-6bc98a60427f"
   {:id "a5c1f29c-08ba-4b57-a714-6bc98a60427f",
    :kind "code",
    :runtime [:runtime "72545a3d-110b-44f0-82ff-8c8e0a0824a9"]},
   "a92cfd0e-49e1-4edd-9e62-d301e552bdcb"
   {:id "a92cfd0e-49e1-4edd-9e62-d301e552bdcb",
    :kind "code",
    :runtime [:runtime "72545a3d-110b-44f0-82ff-8c8e0a0824a9"]},
   "abe86f78-a450-4de4-b31f-d0d394e4e3b7"
   {:id "abe86f78-a450-4de4-b31f-d0d394e4e3b7",
    :kind "code",
    :runtime [:runtime "72545a3d-110b-44f0-82ff-8c8e0a0824a9"]},
   "abece898-8f0a-42c8-9e5f-846667abf977"
   {:id "abece898-8f0a-42c8-9e5f-846667abf977",
    :kind "code",
    :runtime [:runtime "72545a3d-110b-44f0-82ff-8c8e0a0824a9"]},
   "ae07508d-df0d-4a59-bde6-607bb3437bfd"
   {:id "ae07508d-df0d-4a59-bde6-607bb3437bfd", :kind "code-listing"},
   "b0105ca4-efce-4565-b12c-ebaa10392da4"
   {:id "b0105ca4-efce-4565-b12c-ebaa10392da4",
    :kind "code",
    :runtime [:runtime "72545a3d-110b-44f0-82ff-8c8e0a0824a9"]},
   "b06adebb-e0ba-49bd-a989-c11a3fd916b2"
   {:id "b06adebb-e0ba-49bd-a989-c11a3fd916b2", :kind "code-listing"},
   "b179eaf4-3fe6-4f9d-8b99-42027708b512"
   {:id "b179eaf4-3fe6-4f9d-8b99-42027708b512",
    :kind "code",
    :runtime [:runtime "72545a3d-110b-44f0-82ff-8c8e0a0824a9"]},
   "b19cbd15-7a43-422c-81d1-af0fd7ad2b8c"
   {:id "b19cbd15-7a43-422c-81d1-af0fd7ad2b8c",
    :kind "code",
    :runtime [:runtime "72545a3d-110b-44f0-82ff-8c8e0a0824a9"]},
   "b33ce3a1-469a-4113-9fa3-bf502f8939f1"
   {:id "b33ce3a1-469a-4113-9fa3-bf502f8939f1", :kind "code-listing"},
   "b4d9b4d2-42ce-4f32-af43-638a8fc0cb29"
   {:id "b4d9b4d2-42ce-4f32-af43-638a8fc0cb29",
    :kind "code",
    :runtime [:runtime "72545a3d-110b-44f0-82ff-8c8e0a0824a9"]},
   "b8937176-f6d7-4370-aca2-35ed8b725358"
   {:id "b8937176-f6d7-4370-aca2-35ed8b725358",
    :kind "code",
    :runtime [:runtime "72545a3d-110b-44f0-82ff-8c8e0a0824a9"]},
   "ba4fbe14-3d9d-436e-9409-1a13f2a72dbe"
   {:id "ba4fbe14-3d9d-436e-9409-1a13f2a72dbe", :kind "code-listing"},
   "c167727b-296d-4851-865e-6f0d911b17f0"
   {:id "c167727b-296d-4851-865e-6f0d911b17f0",
    :kind "code",
    :runtime [:runtime "72545a3d-110b-44f0-82ff-8c8e0a0824a9"]},
   "c8a1cf62-a8eb-4ce4-bc7e-88733e109bdc"
   {:id "c8a1cf62-a8eb-4ce4-bc7e-88733e109bdc", :kind "code-listing"},
   "caefea90-0d89-4246-8fb1-a4675ada2122"
   {:id "caefea90-0d89-4246-8fb1-a4675ada2122",
    :kind "code",
    :runtime [:runtime "72545a3d-110b-44f0-82ff-8c8e0a0824a9"]},
   "cb9c4ca4-5912-45ea-b093-10749f1732c6"
   {:id "cb9c4ca4-5912-45ea-b093-10749f1732c6",
    :kind "code",
    :runtime [:runtime "72545a3d-110b-44f0-82ff-8c8e0a0824a9"]},
   "ccc42ae8-df3a-4794-88ca-12d627b4b29d"
   {:id "ccc42ae8-df3a-4794-88ca-12d627b4b29d", :kind "code-listing"},
   "ce124c08-d13b-4bbe-9f6f-d994f77cf2ca"
   {:id "ce124c08-d13b-4bbe-9f6f-d994f77cf2ca",
    :kind "code",
    :runtime [:runtime "72545a3d-110b-44f0-82ff-8c8e0a0824a9"]},
   "d005a0af-0d2d-4a5c-a060-8747158523db"
   {:id "d005a0af-0d2d-4a5c-a060-8747158523db", :kind "code-listing"},
   "d2011551-baf4-4041-9c21-3443b640213d"
   {:id "d2011551-baf4-4041-9c21-3443b640213d",
    :kind "code",
    :runtime [:runtime "72545a3d-110b-44f0-82ff-8c8e0a0824a9"]},
   "d24d0a60-306f-4f30-b2cc-13120942eb6b"
   {:id "d24d0a60-306f-4f30-b2cc-13120942eb6b", :kind "code-listing"},
   "d457874a-2e62-49a5-9512-5f31008b4b06"
   {:id "d457874a-2e62-49a5-9512-5f31008b4b06", :kind "code-listing"},
   "e4f1c4f0-6440-4e01-bdfb-670d6cc1aed0"
   {:id "e4f1c4f0-6440-4e01-bdfb-670d6cc1aed0",
    :kind "code",
    :runtime [:runtime "72545a3d-110b-44f0-82ff-8c8e0a0824a9"]},
   "eccda521-8c43-4234-be30-ef7b50325f60"
   {:id "eccda521-8c43-4234-be30-ef7b50325f60", :kind "code-listing"},
   "ece537a0-482f-415e-b217-4f13685e32c3"
   {:id "ece537a0-482f-415e-b217-4f13685e32c3",
    :kind "code",
    :runtime [:runtime "72545a3d-110b-44f0-82ff-8c8e0a0824a9"]},
   "ef6fdcb3-61f5-46c6-adec-4770e7909c19"
   {:id "ef6fdcb3-61f5-46c6-adec-4770e7909c19",
    :kind "code",
    :runtime [:runtime "72545a3d-110b-44f0-82ff-8c8e0a0824a9"]},
   "f35c6fbc-a3a1-4c5a-898e-b502bf0707ef"
   {:id "f35c6fbc-a3a1-4c5a-898e-b502bf0707ef", :kind "code-listing"},
   "f8294e3d-5a0a-48ee-a108-c7950ab1488a"
   {:id "f8294e3d-5a0a-48ee-a108-c7950ab1488a", :kind "code-listing"},
   "f8cf8a77-d65f-45f8-9b41-b1c9ace73e45"
   {:id "f8cf8a77-d65f-45f8-9b41-b1c9ace73e45", :kind "code-listing"}},
  :nextjournal/id #uuid "2ad2ad82-bd11-4171-a456-041506b53d0e",
  :article/change
  {:nextjournal/id #uuid "60b4a18e-90d3-4128-b155-f218ac452da5"}}}

```
</details>
