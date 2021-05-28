# Learn Crux Datalog Today

Learn Crux Datalog Today is derived from the classic [learndatalogtoday.org](http://learndatalogtoday.org) tutorial materials, but adapted to focus on the [Crux](https://opencrux.com) API and unique Datalog semantics, and extended with additional topics and exercises. It is an interactive tutorial designed to teach you the Crux dialect of Datalog. Datalog is a declarative database query language with roots in logic programming. Datalog has similar expressive power to SQL.

You can follow along by hitting the "Remix" button on the toolbar to start your own interactive session running entirely on the Nextjournal platform, or you can copy the steps into a suitable Clojure REPL running locally on your machine. You could also use curl (or similar) to send the data and queries in this tutorial via HTTP to a pre-built Docker image.

The `.md` source file for this notebook is available on [GitHub](https://github.com/crux-labs/learn-crux-datalog-today).

[Eclipse Public License - Version 1.0](https://github.com/crux-labs/learn-crux-datalog-today/blob/master/LICENSE.html)

© 2013 - 2016 Jonas Enlund

Thank you Jonas and contributors for freely licensing your excellent materials!

# Runtime Setup

You need to get Crux running before you can use it. Here we are using Clojure on the JVM and using Crux locally, as an embedded in-memory library, but the Datalog query API is identical and all these queries would work equally over the network via HTTP and the Java/Clojure client library).

```edn no-exec id=9b14eaa5-fb76-4462-9565-339777e45630
{:deps
 {org.clojure/clojure {:mvn/version "1.10.0"}
  org.clojure/tools.deps.alpha
  {:git/url "https://github.com/clojure/tools.deps.alpha.git"
   :sha "f6c080bd0049211021ea59e516d1785b08302515"}
  juxt/crux-core {:mvn/version "RELEASE"}}}
```

```clojure id=a3533ee5-2a4d-4b3f-b264-2953ab6e080e
(require '[crux.api :as crux])
```

Next we need some data. Crux interpets maps as "documents", which require no pre-defined schema, they only need a valid ID attribute. In the data below (which is defined using edn, and fully explained in the section) we are using negative integers as IDs and any top-level attributes that refer to these integers can be interpreted in an ad-hoc way and traversed by the query engine - this capability is known as "schema-on-read".

This vector of maps contains two kinds of documents, documents relating to people (actors and directors), and documents relating to movies. As a convention to aid human interpretation, all persons have IDs like `-1XX` and all movies have IDs like `-2XX`. Many ID value types are supported, such as strings and UUIDs, which may well be more appropriate in a real application.

```clojure id=efe52cdb-608d-4456-aa95-b08947981068
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

```clojure id=929da32a-05cd-456e-8d53-32a3beb6887b
(def my-node (crux/start-node {}))
```

Loading the small amount of data we defined under `my-docs` above can be comfortably done in a single transaction. In practice you will often find benefit to batch `put` operations into groups of 1000 at a time. This code maps over the docs to generate a single transaction containing a `:crux.tx/put` operation per document, submits the transaction. Finally we call the `sync` function to ensure that the documents are fully indexed (and that the transaction has succeeded) before we attempt to run any queries - this is necessary because of Crux's asynchronous design.

```clojure id=0ab1903f-5628-4716-acbb-4d37c671352f
(crux/submit-tx my-node (for [doc my-docs]
                          [:crux.tx/put doc]))

(crux/sync my-node)
```

With Crux running and the data loaded, you can now execute a query, which is a Clojure map, by passing it to Crux's `q` API, which takes the result of a `db` call as it's first value. The meaning of this query will become apparent very soon!

```clojure id=e6d0da43-d8f1-491e-94df-9a5efc0c2ec3
(crux/q (crux/db my-node)
        '{:find [title]
          :where [[_ :movie/title title]]})
```

To simplify this `crux/q` call throughout the rest of the tutorial we can define a new `q` function that saves us a few characters and visual clutter.

```clojure id=a6ab32f1-552d-4d6a-adaf-4aeab53e4657
(defn q [query & args]
  (apply crux/q (crux/db my-node) query args))
```

Queries can then be executed very trivially:

```clojure id=212bbcb6-3979-4f8c-8bd9-3eb324495433
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

```edn no-exec id=5e98f53d-f608-47c5-a0d5-03098b422ad5
    {:find [title]
     :where [[_ :movie/title title]]}
```

Note that the query is a map with two key-value pairs:

* the `:find` vector containing the symbol `title`
* the `:where` vector containing a single query clause `[_ :movie/title title]` (which is also a vector)

Crux also supports queries in an alternative "vector" format:

```edn no-exec id=a5bdc776-e4d6-4db9-92b4-814c865f6eab
    [:find ?title
     :where 
     [_ :movie/title ?title]]
```

However, in this tutorial we will use the map format. Also note that Crux doesn't not require logical variables to be preceded by `?` although you may use this convention if you wish.

## Exercises

Q1. Find all the movie titles in the database

```edn no-exec id=98652a0b-73b0-4f35-af46-af0e93768068
{:find ...}
```

## Solutions

A1.

```clojure id=54dd73bb-849b-43ed-a0bc-d1e65beda8ec
(q '{:find [title]
     :where [[_ :movie/title title]]})
```

# Basic Queries

The example database we'll use contains *movies* mostly, but not exclusively, from the 80s. You'll find information about movie titles, release year, directors, cast members etc. As the tutorial advances we'll learn more about the contents of the database and how it's organized.

The data model in Crux is based around atomic facts called triples. A triple is a 3-tuple consisting of

* Entity ID
* Attribute
* Value

You can think of the database as a flat **set of triples** of the form:

```no-exec id=331ced5e-62f3-463e-9768-5790ffa1a63a
[<e-id>  <attribute>      <value>          ]
...
[ -167    :person/name     "James Cameron" ]
[ -234    :movie/title     "Die Hard"      ]
[ -234    :movie/year      1987            ]
[ -235    :movie/title     "Terminator"    ]
[ -235    :movie/director  167             ]
...
```

Note that the last two triples share the same entity ID, which means they are facts about the same movie. Note also that the last triple's value is the same as the first triple's entity ID, i.e. the value of the `:movie/director` attribute is itself an entity. All the triples in the above set were added to the database in the same transaction, so they share the same transaction ID.

A query is represented as a map with at least two key-value pairs. In the first pair, the key is the keyword `:find`, and the value is a vector of one or more **logic variables** (symbols, e.g. `title`). The other key-value pair is the `:where` keyword key with a vector of clauses which restrict the query to triples that match the given **data patterns**.

For example, this query finds all entity-ids that have the attribute `:person/name` with a value of `"Ridley Scott"`:

```clojure id=9b0f3f32-872e-4feb-8e33-f0e4956b5666
(q '{:find [e]
     :where [[e :person/name "Ridley Scott"]]})
```

The simplest data pattern is a triple with some parts replaced with logic variables. It is the job of the query engine to figure out every possible value of each of the logic variables and return the ones that are specified in the `:find` clause.

The symbol `_` can be used as a wildcard for the parts of the data pattern that you wish to ignore. You can also elide trailing values in a data pattern. Therefore, the following two queries are equivalent.

```clojure id=9be48b52-ae27-4b05-bd64-303a09ac5c2a
(q '{:find [e]
     :where [[e :person/name _]]})
```

```clojure id=4b706f17-9761-4ff4-8f14-57db295ba08b
(q '{:find [e]
     :where [[e :person/name]]})
```

## Exercises

Q1. Find the entity ids of movies made in 1987

```edn no-exec id=debd7655-639d-445d-9a07-a5880bacb57b
{:find [e]
 :where ...}
```

Q2. Find the entity-id and titles of movies in the database

```edn no-exec id=0bd13232-c0d4-495c-9bf3-beca170634c1
{:find [e title]
 :where ...}
```

Q3. Find the name of all people in the database

```edn no-exec id=8120d3ce-0f52-453e-8d00-f1eb99ae4dcd
{:find ...}
```

## Solutions

A1.

```clojure id=b6a65d2a-5584-430c-8f0d-ddf9843a3714
(q '{:find [e]
     :where [[e :movie/year 1987]]})
```

A2.

```clojure id=a82cf90c-1739-4681-82f9-1ce031495f21
(q '{:find [e]
     :where [[e :movie/title title]]})
```

A3.

```clojure id=04c19ae6-dcf6-4864-9d12-2692eb94eff2
(q '{:find [name]
     :where [[_ :person/name name]]})
```

# Data patterns

In the previous chapter, we looked at **data patterns**, i.e., vectors within the `:where` vector, such as `[e :movie/title "Commando"]`. There can be many data patterns in a `:where` clause:

```clojure id=d33c8cc6-9e57-4827-b99c-8fb0501a492d
(q '{:find [title]
     :where [[e :movie/year 1987]
             [e :movie/title title]]})
```

The important thing to note here is that the logic variable `e` is used in both data patterns. When a logic variable is used in multiple places, the query engine requires it to be bound to the same value in each place. Therefore, this query will only find movie titles for movies made in 1987.

The order of the data patterns does not matter (Crux ignore the user-provided clause ordering), so the previous query could just as well have been written this way:

```clojure id=18f25879-75eb-47fd-be10-6bf31e207720
(q '{:find [title]
     :where [[e :movie/title title]
             [e :movie/year 1987]]})
```

In both cases, the result set will be exactly the same.

Let's say we want to find out who starred in "Lethal Weapon". We will need three data patterns for this. The first one finds the entity ID of the movie with "Lethal Weapon" as the title:

```no-exec id=0f692c49-4793-40f2-8b0c-52384def81e4
[m :movie/title "Lethal Weapon"]
```

Using the same entity ID at `m`, we can find the cast members with the data pattern:

```no-exec id=37ed29ab-a95a-4f8f-a773-c1811ef0e167
[m :movie/cast p] 
```

In this pattern, `p` will now be (the entity ID of) a person entity, so we can grab the actual name with:

```no-exec id=0737ccad-7526-45d1-ae0e-e7403297a2f4
[p :person/name name] 
```

The query will therefore be:

```no-exec id=57c252be-2ae0-47e8-a0a1-05a88b93359a
{:find [name]
 :where [[m :movie/title "Lethal Weapon"]
         [m :movie/cast p]
         [p :person/name name]]}
```

## Exercises

Q1. Find movie titles made in 1985

```edn no-exec id=fd4c1d57-a906-4b5b-973d-f3d9bc5e536b
{:find [title]
 :where ...}
```

Q2. What year was "Alien" released?

```edn no-exec id=891ca85c-001f-4eb7-8d52-503f44bbe2ea
{:find [year]
 :where ...}
```

Q3. Who directed RoboCop? You will need to use \[ :movie/director \] to find the director for a movie.

```edn no-exec id=78bdfbd9-50dc-4f5d-9f32-7710568ee0fb
{:find [name]
 :where ...}
```

Q4. Find directors who have directed Arnold Schwarzenegger in a movie.

```edn no-exec id=d1d403b7-b1ba-45f4-90d1-b10301205d6a
{:find [name]
 :where ...}
```

## Solutions

A1.

```clojure id=05b5e972-47c8-465d-b867-11f52ded85c4
(q '{:find [title]
     :where [[m :movie/title title]
             [m :movie/year 1985]]})
```

A2.

```clojure id=13691607-a59e-4d27-9602-a1ab19d9f728
(q '{:find [year]
     :where [[m :movie/title "Alien"]
             [m :movie/year year]]})
```

A3.

```clojure id=ad2d4749-9039-4afa-8904-660031b34fc3
(q '{:find [name]
     :where [[m :movie/title "RoboCop"]
             [m :movie/director d]
             [d :person/name name]]})
```

A4.

```clojure id=063b172e-2978-4a85-8967-4cdf3859d4e7
(q '{:find [name]
     :where [[p :person/name "Arnold Schwarzenegger"]
             [m :movie/cast p]
             [m :movie/director d]
             [d :person/name name]]})
```

# Parameterized queries

Looking at this query:

```clojure id=353aef3b-b69f-4a34-872f-b89160dfe447
(q '{:find [title]
     :where [[p :person/name "Sylvester Stallone"]
             [m :movie/cast p]
             [m :movie/title title]]})
```

It would be great if we could reuse this query to find movie titles for any actor and not just for "Sylvester Stallone". This is possible with an `:in` clause, which provides the query with input parameters, much in the same way that function or method arguments does in your programming language.

Here's that query with an input parameter for the actor:

```clojure id=6be2eb27-c92d-4644-a4f7-42a9acc3a84a
(q '{:find [title]
     :in [name]
     :where [[p :person/name name]
             [m :movie/cast p]
             [m :movie/title title]]}
   "Sylvester Stallone")
```

This query takes one argument: `name` which presumably will be the name of some actor.

The above query is executed like `(q db query "Sylvester Stallone")`, where `query` is the query we just saw, and `db` is a database value. You can have any number of inputs to a query.

In the above query, the input logic variable `name` is bound to a scalar - a string in this case. There are four different kinds of input: scalars, tuples, collections and relations.

## A quick aside

Note that an implicit first argument `$` is also available, should it ever be needed, which is the value of database `db` itself. You can use this in conjunction with more advanced features like subqueries and custom Clojure predicates (more on those later!).

## Tuples

A tuple input is written as e.g. `[name age]` and can be used when you want to destructure an input. Let's say you have the vector `["James Cameron" "Arnold Schwarzenegger"]` and you want to use this as input to find all movies where these two people collaborated:

```clojure id=7a7b8b54-6fd0-4d73-a6a1-a56e565c4e59
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

```no-exec id=d440ef30-e6ef-4335-bc71-a13e0be993c2
:in [director actor]
```

## Collections

You can use collection destructuring to implement a kind of logical **or** in your query. Say you want to find all movies directed by either James Cameron **or** Ridley Scott:

```clojure id=9873d423-30a8-4663-b1e9-1de45cfeeb63
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

```no-exec id=ba277ef0-35d5-4c9f-830b-ab0f6a70e195
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

```clojure id=c98c3ac3-68ba-4029-b36e-7941738cf006
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

```edn no-exec id=18dc0bbd-570e-4be4-8575-b4b6cf60ab85
{:find [title]
 :in [year]
 :where ...}
```

Q2. Given a list of movie titles, find the title and the year that movie was released.

```edn no-exec id=bf351c5a-8a7c-4413-946c-298ff0b61c11
{:find [title year]
 :in ...
 :where ...}
```

Q3. Find all movie `title`s where the `actor` and the `director` has worked together

```edn no-exec id=6c8d6ecc-279d-470e-ac86-991b29efafe0
{:find [title]
 :in [actor director]
 :where ...}
```

Q4. Write a query that, given an actor name and a relation with movie-title/rating, finds the movie titles and corresponding rating for which that actor was a cast member.

```edn no-exec id=0282a9ba-09b4-4d81-9172-efe384bafc25
{:find [title rating]
 :in ...
 :where ...}
```

## Solutions

A1.

```clojure id=5dc1a8e6-e5c4-404a-871d-3a43e6895f80
(q '{:find [title]
     :in [year]
     :where [[m :movie/year year]
             [m :movie/title title]]}
   1988)
```

A2.

```clojure id=a47db3b2-8f10-4f5a-9d86-994a1600407b
(q '{:find [title year]
     :in [[title ...]]
     :where [[m :movie/title title]
             [m :movie/year year]]}
   ["Lethal Weapon" "Lethal Weapon 2" "Lethal Weapon 3"])
```

A3.

```clojure id=e593d663-2077-4d99-879e-4996af15c39d
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

```clojure id=eae41b71-255f-4600-ab0d-0eab716f9edd
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

```clojure id=bb24e7ee-7056-4639-8431-b1dbe8bbea20
(q '{:find [title]
     :where [[m :movie/title title]
             [m :movie/year year]
             [(< year 1984)]]})
```

The last clause, `[(< year 1984)]`, is a predicate clause. The predicate clause filters the result set to only include results for which the predicate returns a "truthy" (non-nil, non-false) value. You can use any Clojure function as a predicate function:

```clojure id=445a3ad5-b310-4893-8c6e-540a54e58830
(q '{:find [name]
     :where [[p :person/name name]
             [(clojure.string/starts-with? name "M")]]})
```

Clojure functions must be fully namespace-qualified, so if you have defined your own predicate `awesome?` you must write it as `(my.namespace/awesome? movie)`. All `clojure.core/*` functions may be used as predicates without namespace qualification: `<, >, <=, >=, =, not=` and so on. An "allowlist" feature is provided to restrict the exact set of predicates available to queries.

## Exercises

Q1. Find movies older than a certain year (inclusive)

```edn no-exec id=4bd7c440-965c-4c3b-b853-a1ca07bc8bb6
{:find [title]
 :in [year]
 :where ...}
```

Q2. Find actors older than Danny Glover

```edn no-exec id=df24f786-8a12-4cf5-9f25-6e40472fc837
{:find [actor]
 :where ...}
```

Q3. Find movies newer than `year` (inclusive) and has a `rating` higher than the one supplied

```edn no-exec id=d9746235-59a5-438a-9776-fd8a2f4122da
{:find [title]
 :in [year rating [[title r]]]
 :where ...}
```

## Solutions

A1.

```clojure id=3f0903a9-e917-483b-b71d-5c629536617c
(q '{:find [title]
     :in [year]
     :where [[m :movie/title title]
             [m :movie/year y]
             [(<= y year)]]}
   1979)
```

A2.

```clojure id=67adc21d-8478-4e24-b791-444a2a243652
(q '{:find [actor]
     :where [[d :person/name "Danny Glover"]
             [d :person/born b1]
             [e :person/born b2]
             [_ :movie/cast e]
             [(< b2 b1)]
             [e :person/name actor]]})
```

A3.

```clojure id=1a63545a-92d4-4c0d-921a-c634fc29c53e
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

**Transformation functions** are pure (= side-effect free) functions or methods which can be used in queries as "function expression" predicates to transform values and bind their results to new logic variables. Say, for example, there exists an attribute `:person/born` with type `:db.type/instant`. Given the birthday, it's easy to calculate the (very approximate) age of a person:

```clojure id=566ee82b-ea39-4499-b7e2-d6ed832257f4
(defn age [^java.util.Date birthday ^java.util.Date today]
  (quot (- (.getTime today)
          (.getTime birthday))
        (* 1000 60 60 24 365)))
```

With this function, we can now calculate the age of a person **inside the query itself**:

```clojure id=c9e6b471-6eef-498a-9373-07870892e6c5
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

One thing to be aware of is that transformation functions can't be nested. You can't write

```no-exec id=bff73b0f-d5fe-450e-b1ac-98de607e8a00
[(f (g x)) a]
```

instead, you must bind intermediate results in temporary logic variables

```no-exec id=3af0d68b-4c39-410b-a64e-5bc6d7cf0a43
[(g x) t]
[(f t) a]
```

## Exercises

Q1. Find people by age. Use the function `user/age` to find the age given a birthday and a date representing "today".

```edn no-exec id=bf4bc7e8-0894-409a-b975-e5b0af39e62e
{:find [name]
 :in [age today]
 :where ...}
```

Q2. Find people younger than Bruce Willis and their ages.

```edn no-exec id=716cf1e3-f77b-4fda-bffe-66107176e3d7
{:find [name age]
 :in [today]
 :where ...}
```

## Solutions

A1.

```clojure id=dd8ceb7c-c2a5-4044-98a4-10b1a09bda40
(q '{:find [name]
     :in [age today]
     :where [[p :person/name name]
             [p :person/born born]
             [(user/age born today) age]]}
   63
   #inst "2013-08-02T00:00:00.000-00:00")
```

A2.

```clojure id=d57254e0-2276-432a-a548-fea2ee493026
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

```no-exec id=bbd7d9af-9981-48d0-9874-f96103eae7ac
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

```clojure id=7ee03aee-670b-40cb-a8ca-313f473a0b8b
(q '{:find [(count m)]
     :where [[m :movie/title]]})
```

A2.

```clojure id=746a6ae7-6382-4dc8-80ec-86900f64f05d
(q '{:find [(min date)]
     :where [[_ :person/born date]]})
```

A3.

```clojure id=17d7cd6a-1d80-4896-81fb-45bee022a192
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

```no-exec id=f8fc581a-12c1-49eb-8029-5821080f70fb
[p :person/name name]
[m :movie/cast p]
[m :movie/title title]
```

**Rules** are the means of abstraction in Datalog. You can abstract away reusable parts of your queries into rules, give them meaningful names and forget about the implementation details, just like you can with functions in your favorite programming language. Let's create a rule for the three lines above:

```no-exec id=f363c07c-4971-4cec-ad9a-f7e1e51081fd
[(actor-movie name title)
 [p :person/name name]
 [m :movie/cast p]
 [m :movie/title title]]
```

The first vector is called the *head* of the rule where the first symbol is the name of the rule. The rest of the rule is called the *body*.

You can think of a rule as a kind of function, but remember that this is logic programming, so we can use the same rule to:

* find movie titles given an actor name, and
* find actor names given a movie title.

Put another way, we can use both `name` and `title` in `(actor-movie name title)` for input as well as for output. If we provide values for neither, we'll get all the possible combinations in the database. If we provide values for one or both, it'll constrain the result returned by the query as you'd expect.

To use the above rule, you simply write the head of the rule instead of the data patterns. Any variable with values already bound will be input, the rest will be output.

The query to find cast members of some movie, for which we previously had to write:

```clojure id=4bd397aa-d660-4a2e-9218-da447fd91d8c
(q '{:find [name]
     :where [[p :person/name name]
             [m :movie/cast p]
             [m :movie/title "The Terminator"]]})
```

Now becomes:

```clojure id=32423c20-6947-4b18-8caf-d0cfaa8cdfbd
(q '{:find [name]
     :where [(actor-movie name "The Terminator")]
     :rules [[(actor-movie name title)
              [p :person/name name]
              [m :movie/cast p]
              [m :movie/title "The Terminator"]]]})
```

You can write any number of rules, collect them in a vector, and pass them to the query engine using the `:rules` key-value pair.

```no-exec id=db725e74-90bb-4542-83cf-c48d6678493c
[[(rule-a a b)
  ...]
 [(rule-b a b)
  ...]
 ...]
```

You can use [data patterns](/chapter/2), [predicates](/chapter/5), [transformation functions](/chapter/6) and calls to other rules in the body of a rule.

Rules can also be used as another tool to write logical OR queries, as the same rule name can be used several times:

```no-exec id=37259fb5-d3b3-4778-bd7d-212c39d33814
[[(associated-with person movie)
  [movie :movie/cast person]]
 [(associated-with person movie)
  [movie :movie/director person]]]
```

Subsequent rule definitions will only be used if the ones preceding it aren't satisfied.

Using this rule, we can find both directors and cast members very easily:

```clojure id=830bc9e3-1613-4f7f-8dca-b93d5018e566
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

```edn no-exec id=0df7991b-5029-4855-b9dd-c89993ef6170
{:find [title]
 :where [(movie-year title 1991)]
 :rules [[(movie-year title year)
          ...]]}
```

Q2. Two people are friends if they have worked together in a movie. Write a rule `(friends p1 p2)` where `p1` and `p2` are person entities. Try with a few different `name` inputs to make sure you got it right. There might be some edge cases here.

```edn no-exec id=0c4ddd80-5018-4d98-b077-76599c281afe
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

```clojure id=7c3724c5-1a71-4c55-bddb-4b3c4bea31b1
(q '{:find [title]
     :where [(movie-year title 1991)]
     :rules [[(movie-year title year)
              [m :movie/title title]
              [m :movie/year year]]]})
```

A2.

```clojure id=3d5fe073-29c0-4046-98e4-9ad84c20dc19
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

[github-repository][nextjournal#github-repository#028a2851-053e-469d-afdd-e3fa570d103d]


[nextjournal#github-repository#028a2851-053e-469d-afdd-e3fa570d103d]:
<https://github.com/crux-labs/learn-crux-datalog-today>

<details id="com.nextjournal.article">
<summary>This notebook was exported from <a href="https://nextjournal.com/a/TZRfY6AdC8CnWLxEcXKnDy?change-id=CwWL5MdGidHfwGthT3VBgq">https://nextjournal.com/a/TZRfY6AdC8CnWLxEcXKnDy?change-id=CwWL5MdGidHfwGthT3VBgq</a></summary>

```edn nextjournal-metadata
{:article
 {:nodes
  {"0282a9ba-09b4-4d81-9172-efe384bafc25"
   {:id "0282a9ba-09b4-4d81-9172-efe384bafc25", :kind "code-listing"},
   "028a2851-053e-469d-afdd-e3fa570d103d"
   {:id "028a2851-053e-469d-afdd-e3fa570d103d",
    :kind "github-repository"},
   "04c19ae6-dcf6-4864-9d12-2692eb94eff2"
   {:id "04c19ae6-dcf6-4864-9d12-2692eb94eff2",
    :kind "code",
    :runtime [:runtime "d75fd937-681d-4fe5-8c10-ac9cf8955e00"]},
   "05b5e972-47c8-465d-b867-11f52ded85c4"
   {:id "05b5e972-47c8-465d-b867-11f52ded85c4",
    :kind "code",
    :runtime [:runtime "d75fd937-681d-4fe5-8c10-ac9cf8955e00"]},
   "063b172e-2978-4a85-8967-4cdf3859d4e7"
   {:id "063b172e-2978-4a85-8967-4cdf3859d4e7",
    :kind "code",
    :runtime [:runtime "d75fd937-681d-4fe5-8c10-ac9cf8955e00"]},
   "0737ccad-7526-45d1-ae0e-e7403297a2f4"
   {:id "0737ccad-7526-45d1-ae0e-e7403297a2f4", :kind "code-listing"},
   "0ab1903f-5628-4716-acbb-4d37c671352f"
   {:compute-ref #uuid "1e70927c-0f7e-4877-8aa6-4c81bb1ba108",
    :exec-duration 883,
    :id "0ab1903f-5628-4716-acbb-4d37c671352f",
    :kind "code",
    :output-log-lines {},
    :runtime [:runtime "d75fd937-681d-4fe5-8c10-ac9cf8955e00"]},
   "0bd13232-c0d4-495c-9bf3-beca170634c1"
   {:id "0bd13232-c0d4-495c-9bf3-beca170634c1", :kind "code-listing"},
   "0c4ddd80-5018-4d98-b077-76599c281afe"
   {:id "0c4ddd80-5018-4d98-b077-76599c281afe", :kind "code-listing"},
   "0df7991b-5029-4855-b9dd-c89993ef6170"
   {:id "0df7991b-5029-4855-b9dd-c89993ef6170", :kind "code-listing"},
   "0f692c49-4793-40f2-8b0c-52384def81e4"
   {:id "0f692c49-4793-40f2-8b0c-52384def81e4", :kind "code-listing"},
   "13691607-a59e-4d27-9602-a1ab19d9f728"
   {:id "13691607-a59e-4d27-9602-a1ab19d9f728",
    :kind "code",
    :runtime [:runtime "d75fd937-681d-4fe5-8c10-ac9cf8955e00"]},
   "17d7cd6a-1d80-4896-81fb-45bee022a192"
   {:id "17d7cd6a-1d80-4896-81fb-45bee022a192",
    :kind "code",
    :runtime [:runtime "d75fd937-681d-4fe5-8c10-ac9cf8955e00"]},
   "18dc0bbd-570e-4be4-8575-b4b6cf60ab85"
   {:id "18dc0bbd-570e-4be4-8575-b4b6cf60ab85", :kind "code-listing"},
   "18f25879-75eb-47fd-be10-6bf31e207720"
   {:id "18f25879-75eb-47fd-be10-6bf31e207720",
    :kind "code",
    :runtime [:runtime "d75fd937-681d-4fe5-8c10-ac9cf8955e00"]},
   "1a63545a-92d4-4c0d-921a-c634fc29c53e"
   {:id "1a63545a-92d4-4c0d-921a-c634fc29c53e",
    :kind "code",
    :runtime [:runtime "d75fd937-681d-4fe5-8c10-ac9cf8955e00"]},
   "212bbcb6-3979-4f8c-8bd9-3eb324495433"
   {:id "212bbcb6-3979-4f8c-8bd9-3eb324495433",
    :kind "code",
    :runtime [:runtime "d75fd937-681d-4fe5-8c10-ac9cf8955e00"]},
   "32423c20-6947-4b18-8caf-d0cfaa8cdfbd"
   {:id "32423c20-6947-4b18-8caf-d0cfaa8cdfbd",
    :kind "code",
    :runtime [:runtime "d75fd937-681d-4fe5-8c10-ac9cf8955e00"]},
   "331ced5e-62f3-463e-9768-5790ffa1a63a"
   {:id "331ced5e-62f3-463e-9768-5790ffa1a63a", :kind "code-listing"},
   "353aef3b-b69f-4a34-872f-b89160dfe447"
   {:id "353aef3b-b69f-4a34-872f-b89160dfe447",
    :kind "code",
    :runtime [:runtime "d75fd937-681d-4fe5-8c10-ac9cf8955e00"]},
   "37259fb5-d3b3-4778-bd7d-212c39d33814"
   {:id "37259fb5-d3b3-4778-bd7d-212c39d33814", :kind "code-listing"},
   "37ed29ab-a95a-4f8f-a773-c1811ef0e167"
   {:id "37ed29ab-a95a-4f8f-a773-c1811ef0e167", :kind "code-listing"},
   "3af0d68b-4c39-410b-a64e-5bc6d7cf0a43"
   {:id "3af0d68b-4c39-410b-a64e-5bc6d7cf0a43", :kind "code-listing"},
   "3d5fe073-29c0-4046-98e4-9ad84c20dc19"
   {:id "3d5fe073-29c0-4046-98e4-9ad84c20dc19",
    :kind "code",
    :runtime [:runtime "d75fd937-681d-4fe5-8c10-ac9cf8955e00"]},
   "3f0903a9-e917-483b-b71d-5c629536617c"
   {:id "3f0903a9-e917-483b-b71d-5c629536617c",
    :kind "code",
    :runtime [:runtime "d75fd937-681d-4fe5-8c10-ac9cf8955e00"]},
   "445a3ad5-b310-4893-8c6e-540a54e58830"
   {:id "445a3ad5-b310-4893-8c6e-540a54e58830",
    :kind "code",
    :runtime [:runtime "d75fd937-681d-4fe5-8c10-ac9cf8955e00"]},
   "4b706f17-9761-4ff4-8f14-57db295ba08b"
   {:id "4b706f17-9761-4ff4-8f14-57db295ba08b",
    :kind "code",
    :runtime [:runtime "d75fd937-681d-4fe5-8c10-ac9cf8955e00"]},
   "4bd397aa-d660-4a2e-9218-da447fd91d8c"
   {:id "4bd397aa-d660-4a2e-9218-da447fd91d8c",
    :kind "code",
    :runtime [:runtime "d75fd937-681d-4fe5-8c10-ac9cf8955e00"]},
   "4bd7c440-965c-4c3b-b853-a1ca07bc8bb6"
   {:id "4bd7c440-965c-4c3b-b853-a1ca07bc8bb6", :kind "code-listing"},
   "54dd73bb-849b-43ed-a0bc-d1e65beda8ec"
   {:id "54dd73bb-849b-43ed-a0bc-d1e65beda8ec",
    :kind "code",
    :runtime [:runtime "d75fd937-681d-4fe5-8c10-ac9cf8955e00"]},
   "566ee82b-ea39-4499-b7e2-d6ed832257f4"
   {:id "566ee82b-ea39-4499-b7e2-d6ed832257f4",
    :kind "code",
    :runtime [:runtime "d75fd937-681d-4fe5-8c10-ac9cf8955e00"]},
   "57c252be-2ae0-47e8-a0a1-05a88b93359a"
   {:id "57c252be-2ae0-47e8-a0a1-05a88b93359a", :kind "code-listing"},
   "5dc1a8e6-e5c4-404a-871d-3a43e6895f80"
   {:id "5dc1a8e6-e5c4-404a-871d-3a43e6895f80",
    :kind "code",
    :runtime [:runtime "d75fd937-681d-4fe5-8c10-ac9cf8955e00"]},
   "5e98f53d-f608-47c5-a0d5-03098b422ad5"
   {:id "5e98f53d-f608-47c5-a0d5-03098b422ad5", :kind "code-listing"},
   "67adc21d-8478-4e24-b791-444a2a243652"
   {:id "67adc21d-8478-4e24-b791-444a2a243652",
    :kind "code",
    :runtime [:runtime "d75fd937-681d-4fe5-8c10-ac9cf8955e00"]},
   "6be2eb27-c92d-4644-a4f7-42a9acc3a84a"
   {:id "6be2eb27-c92d-4644-a4f7-42a9acc3a84a",
    :kind "code",
    :runtime [:runtime "d75fd937-681d-4fe5-8c10-ac9cf8955e00"]},
   "6c8d6ecc-279d-470e-ac86-991b29efafe0"
   {:id "6c8d6ecc-279d-470e-ac86-991b29efafe0", :kind "code-listing"},
   "716cf1e3-f77b-4fda-bffe-66107176e3d7"
   {:id "716cf1e3-f77b-4fda-bffe-66107176e3d7", :kind "code-listing"},
   "746a6ae7-6382-4dc8-80ec-86900f64f05d"
   {:id "746a6ae7-6382-4dc8-80ec-86900f64f05d",
    :kind "code",
    :runtime [:runtime "d75fd937-681d-4fe5-8c10-ac9cf8955e00"]},
   "78bdfbd9-50dc-4f5d-9f32-7710568ee0fb"
   {:id "78bdfbd9-50dc-4f5d-9f32-7710568ee0fb", :kind "code-listing"},
   "7a7b8b54-6fd0-4d73-a6a1-a56e565c4e59"
   {:id "7a7b8b54-6fd0-4d73-a6a1-a56e565c4e59",
    :kind "code",
    :runtime [:runtime "d75fd937-681d-4fe5-8c10-ac9cf8955e00"]},
   "7c3724c5-1a71-4c55-bddb-4b3c4bea31b1"
   {:id "7c3724c5-1a71-4c55-bddb-4b3c4bea31b1",
    :kind "code",
    :runtime [:runtime "d75fd937-681d-4fe5-8c10-ac9cf8955e00"]},
   "7ee03aee-670b-40cb-a8ca-313f473a0b8b"
   {:id "7ee03aee-670b-40cb-a8ca-313f473a0b8b",
    :kind "code",
    :runtime [:runtime "d75fd937-681d-4fe5-8c10-ac9cf8955e00"]},
   "8120d3ce-0f52-453e-8d00-f1eb99ae4dcd"
   {:id "8120d3ce-0f52-453e-8d00-f1eb99ae4dcd", :kind "code-listing"},
   "830bc9e3-1613-4f7f-8dca-b93d5018e566"
   {:id "830bc9e3-1613-4f7f-8dca-b93d5018e566",
    :kind "code",
    :runtime [:runtime "d75fd937-681d-4fe5-8c10-ac9cf8955e00"]},
   "891ca85c-001f-4eb7-8d52-503f44bbe2ea"
   {:id "891ca85c-001f-4eb7-8d52-503f44bbe2ea", :kind "code-listing"},
   "929da32a-05cd-456e-8d53-32a3beb6887b"
   {:compute-ref #uuid "6c7c4e07-7e6f-4db0-a17b-b2da29e332dd",
    :exec-duration 7220,
    :id "929da32a-05cd-456e-8d53-32a3beb6887b",
    :kind "code",
    :output-log-lines {},
    :runtime [:runtime "d75fd937-681d-4fe5-8c10-ac9cf8955e00"]},
   "98652a0b-73b0-4f35-af46-af0e93768068"
   {:id "98652a0b-73b0-4f35-af46-af0e93768068", :kind "code-listing"},
   "9873d423-30a8-4663-b1e9-1de45cfeeb63"
   {:id "9873d423-30a8-4663-b1e9-1de45cfeeb63",
    :kind "code",
    :runtime [:runtime "d75fd937-681d-4fe5-8c10-ac9cf8955e00"]},
   "9b0f3f32-872e-4feb-8e33-f0e4956b5666"
   {:id "9b0f3f32-872e-4feb-8e33-f0e4956b5666",
    :kind "code",
    :runtime [:runtime "d75fd937-681d-4fe5-8c10-ac9cf8955e00"]},
   "9b14eaa5-fb76-4462-9565-339777e45630"
   {:id "9b14eaa5-fb76-4462-9565-339777e45630",
    :kind "code-listing",
    :name "deps.edn"},
   "9be48b52-ae27-4b05-bd64-303a09ac5c2a"
   {:id "9be48b52-ae27-4b05-bd64-303a09ac5c2a",
    :kind "code",
    :runtime [:runtime "d75fd937-681d-4fe5-8c10-ac9cf8955e00"]},
   "a3533ee5-2a4d-4b3f-b264-2953ab6e080e"
   {:compute-ref #uuid "43438de8-4f51-4f00-95f8-d3c26cd1309a",
    :exec-duration 13137,
    :id "a3533ee5-2a4d-4b3f-b264-2953ab6e080e",
    :kind "code",
    :output-log-lines {},
    :runtime [:runtime "d75fd937-681d-4fe5-8c10-ac9cf8955e00"]},
   "a47db3b2-8f10-4f5a-9d86-994a1600407b"
   {:id "a47db3b2-8f10-4f5a-9d86-994a1600407b",
    :kind "code",
    :runtime [:runtime "d75fd937-681d-4fe5-8c10-ac9cf8955e00"]},
   "a5bdc776-e4d6-4db9-92b4-814c865f6eab"
   {:id "a5bdc776-e4d6-4db9-92b4-814c865f6eab", :kind "code-listing"},
   "a6ab32f1-552d-4d6a-adaf-4aeab53e4657"
   {:id "a6ab32f1-552d-4d6a-adaf-4aeab53e4657",
    :kind "code",
    :runtime [:runtime "d75fd937-681d-4fe5-8c10-ac9cf8955e00"]},
   "a82cf90c-1739-4681-82f9-1ce031495f21"
   {:id "a82cf90c-1739-4681-82f9-1ce031495f21",
    :kind "code",
    :runtime [:runtime "d75fd937-681d-4fe5-8c10-ac9cf8955e00"]},
   "ad2d4749-9039-4afa-8904-660031b34fc3"
   {:id "ad2d4749-9039-4afa-8904-660031b34fc3",
    :kind "code",
    :runtime [:runtime "d75fd937-681d-4fe5-8c10-ac9cf8955e00"]},
   "b6a65d2a-5584-430c-8f0d-ddf9843a3714"
   {:id "b6a65d2a-5584-430c-8f0d-ddf9843a3714",
    :kind "code",
    :runtime [:runtime "d75fd937-681d-4fe5-8c10-ac9cf8955e00"]},
   "ba277ef0-35d5-4c9f-830b-ab0f6a70e195"
   {:id "ba277ef0-35d5-4c9f-830b-ab0f6a70e195", :kind "code-listing"},
   "bb24e7ee-7056-4639-8431-b1dbe8bbea20"
   {:id "bb24e7ee-7056-4639-8431-b1dbe8bbea20",
    :kind "code",
    :runtime [:runtime "d75fd937-681d-4fe5-8c10-ac9cf8955e00"]},
   "bbd7d9af-9981-48d0-9874-f96103eae7ac"
   {:id "bbd7d9af-9981-48d0-9874-f96103eae7ac", :kind "code-listing"},
   "bf351c5a-8a7c-4413-946c-298ff0b61c11"
   {:id "bf351c5a-8a7c-4413-946c-298ff0b61c11", :kind "code-listing"},
   "bf4bc7e8-0894-409a-b975-e5b0af39e62e"
   {:id "bf4bc7e8-0894-409a-b975-e5b0af39e62e", :kind "code-listing"},
   "bff73b0f-d5fe-450e-b1ac-98de607e8a00"
   {:id "bff73b0f-d5fe-450e-b1ac-98de607e8a00", :kind "code-listing"},
   "c98c3ac3-68ba-4029-b36e-7941738cf006"
   {:id "c98c3ac3-68ba-4029-b36e-7941738cf006",
    :kind "code",
    :runtime [:runtime "d75fd937-681d-4fe5-8c10-ac9cf8955e00"]},
   "c9e6b471-6eef-498a-9373-07870892e6c5"
   {:id "c9e6b471-6eef-498a-9373-07870892e6c5",
    :kind "code",
    :runtime [:runtime "d75fd937-681d-4fe5-8c10-ac9cf8955e00"]},
   "d1d403b7-b1ba-45f4-90d1-b10301205d6a"
   {:id "d1d403b7-b1ba-45f4-90d1-b10301205d6a", :kind "code-listing"},
   "d33c8cc6-9e57-4827-b99c-8fb0501a492d"
   {:id "d33c8cc6-9e57-4827-b99c-8fb0501a492d",
    :kind "code",
    :runtime [:runtime "d75fd937-681d-4fe5-8c10-ac9cf8955e00"]},
   "d440ef30-e6ef-4335-bc71-a13e0be993c2"
   {:id "d440ef30-e6ef-4335-bc71-a13e0be993c2", :kind "code-listing"},
   "d57254e0-2276-432a-a548-fea2ee493026"
   {:id "d57254e0-2276-432a-a548-fea2ee493026",
    :kind "code",
    :runtime [:runtime "d75fd937-681d-4fe5-8c10-ac9cf8955e00"]},
   "d75fd937-681d-4fe5-8c10-ac9cf8955e00"
   {:environment
    [:environment
     {:article/nextjournal.id
      #uuid "5b45eb52-bad4-413d-9d7f-b2b573a25322",
      :change/nextjournal.id
      #uuid "5f045c36-90bd-428b-a26c-b59fa0a2e1db",
      :node/id "0ae15688-6f6a-40e2-a4fa-52d81371f733"}],
    :id "d75fd937-681d-4fe5-8c10-ac9cf8955e00",
    :kind "runtime",
    :language "clojure",
    :type :nextjournal,
    :runtime/mounts
    [{:src [:node "028a2851-053e-469d-afdd-e3fa570d103d"],
      :dest "/learn-crux-datalog-today"}
     {:src [:node "9b14eaa5-fb76-4462-9565-339777e45630"],
      :dest "/learn-crux-datalog-today/deps.edn"}]},
   "d9746235-59a5-438a-9776-fd8a2f4122da"
   {:id "d9746235-59a5-438a-9776-fd8a2f4122da", :kind "code-listing"},
   "db725e74-90bb-4542-83cf-c48d6678493c"
   {:id "db725e74-90bb-4542-83cf-c48d6678493c", :kind "code-listing"},
   "dd8ceb7c-c2a5-4044-98a4-10b1a09bda40"
   {:id "dd8ceb7c-c2a5-4044-98a4-10b1a09bda40",
    :kind "code",
    :runtime [:runtime "d75fd937-681d-4fe5-8c10-ac9cf8955e00"]},
   "debd7655-639d-445d-9a07-a5880bacb57b"
   {:id "debd7655-639d-445d-9a07-a5880bacb57b", :kind "code-listing"},
   "df24f786-8a12-4cf5-9f25-6e40472fc837"
   {:id "df24f786-8a12-4cf5-9f25-6e40472fc837", :kind "code-listing"},
   "e593d663-2077-4d99-879e-4996af15c39d"
   {:id "e593d663-2077-4d99-879e-4996af15c39d",
    :kind "code",
    :runtime [:runtime "d75fd937-681d-4fe5-8c10-ac9cf8955e00"]},
   "e6d0da43-d8f1-491e-94df-9a5efc0c2ec3"
   {:compute-ref #uuid "c4494548-2ebe-4ea5-821c-24ba46f34341",
    :exec-duration 257,
    :id "e6d0da43-d8f1-491e-94df-9a5efc0c2ec3",
    :kind "code",
    :output-log-lines {},
    :runtime [:runtime "d75fd937-681d-4fe5-8c10-ac9cf8955e00"]},
   "eae41b71-255f-4600-ab0d-0eab716f9edd"
   {:id "eae41b71-255f-4600-ab0d-0eab716f9edd",
    :kind "code",
    :runtime [:runtime "d75fd937-681d-4fe5-8c10-ac9cf8955e00"]},
   "efe52cdb-608d-4456-aa95-b08947981068"
   {:compute-ref #uuid "ccdb21cd-bfe5-4806-836f-5e481e3893a1",
    :exec-duration 388,
    :id "efe52cdb-608d-4456-aa95-b08947981068",
    :kind "code",
    :output-log-lines {},
    :runtime [:runtime "d75fd937-681d-4fe5-8c10-ac9cf8955e00"]},
   "f363c07c-4971-4cec-ad9a-f7e1e51081fd"
   {:id "f363c07c-4971-4cec-ad9a-f7e1e51081fd", :kind "code-listing"},
   "f8fc581a-12c1-49eb-8029-5821080f70fb"
   {:id "f8fc581a-12c1-49eb-8029-5821080f70fb", :kind "code-listing"},
   "fd4c1d57-a906-4b5b-973d-f3d9bc5e536b"
   {:id "fd4c1d57-a906-4b5b-973d-f3d9bc5e536b", :kind "code-listing"}},
  :nextjournal/id #uuid "d7149b14-b95f-4b3c-af9f-ca9c1cfd3994",
  :article/change
  {:nextjournal/id #uuid "60b0c36e-822a-483b-91a0-3596a1cf49ae"}}}

```
</details>
