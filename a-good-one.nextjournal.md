# A Good One

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

## ns

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

[github-repository][nextjournal#github-repository#5f5fbdc7-0f65-4203-84b7-e15157cc7766]


[nextjournal#github-repository#5f5fbdc7-0f65-4203-84b7-e15157cc7766]:
<https://github.com/zampino/nextjournal-github-explorer-test>

<details id="com.nextjournal.article">
<summary>This notebook was exported from <a href="https://nextjournal.com/a/V19A9Tt9exrsM51AdmskcG?change-id=CwcdAKceRG2fK9rN9Tx5KW">https://nextjournal.com/a/V19A9Tt9exrsM51AdmskcG?change-id=CwcdAKceRG2fK9rN9Tx5KW</a></summary>

```edn nextjournal-metadata
{:article
 {:nodes
  {"06bcaef5-5991-45f9-aeba-b2635c0070ce"
   {:id "06bcaef5-5991-45f9-aeba-b2635c0070ce", :kind "code-listing"},
   "06cb79d1-2f7a-4df8-a9b0-7cebdc72cbaa"
   {:id "06cb79d1-2f7a-4df8-a9b0-7cebdc72cbaa",
    :kind "code",
    :runtime [:runtime "72545a3d-110b-44f0-82ff-8c8e0a0824a9"]},
   "0dace1c8-8008-4bc3-829b-2f9a3f640ae9"
   {:id "0dace1c8-8008-4bc3-829b-2f9a3f640ae9",
    :kind "code",
    :runtime [:runtime "72545a3d-110b-44f0-82ff-8c8e0a0824a9"]},
   "17837f57-61d5-4e12-9892-17c23790ef60"
   {:id "17837f57-61d5-4e12-9892-17c23790ef60",
    :kind "code",
    :runtime [:runtime "72545a3d-110b-44f0-82ff-8c8e0a0824a9"]},
   "1a3b51e5-1b93-48b5-a009-a0911c8a50c8"
   {:id "1a3b51e5-1b93-48b5-a009-a0911c8a50c8",
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
   "34903538-f626-4f08-ab27-6336b57bc349"
   {:id "34903538-f626-4f08-ab27-6336b57bc349", :kind "code-listing"},
   "3e773974-6168-4776-9d56-4b8c7f8fe348"
   {:id "3e773974-6168-4776-9d56-4b8c7f8fe348", :kind "code-listing"},
   "4b272a39-ce69-4b34-b46b-7308e5e2ef7e"
   {:id "4b272a39-ce69-4b34-b46b-7308e5e2ef7e", :kind "code-listing"},
   "4df0ad34-581e-4c93-9fb4-e24929a5b7ca"
   {:id "4df0ad34-581e-4c93-9fb4-e24929a5b7ca", :kind "code-listing"},
   "4ff59332-2846-40d2-8f59-89eb9d18f60e"
   {:id "4ff59332-2846-40d2-8f59-89eb9d18f60e", :kind "code-listing"},
   "5df6a76e-c47f-4aab-bd03-5f244923d4ad"
   {:id "5df6a76e-c47f-4aab-bd03-5f244923d4ad",
    :kind "code",
    :runtime [:runtime "72545a3d-110b-44f0-82ff-8c8e0a0824a9"]},
   "5f5fbdc7-0f65-4203-84b7-e15157cc7766"
   {:id "5f5fbdc7-0f65-4203-84b7-e15157cc7766",
    :kind "github-repository",
    :ref "main"},
   "6603b044-f33b-4616-8500-526655d86ffa"
   {:id "6603b044-f33b-4616-8500-526655d86ffa",
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
    [{:src [:node "5f5fbdc7-0f65-4203-84b7-e15157cc7766"],
      :dest "/nextjournal-github-explorer-test"}
     {:src [:node "905a73c5-dbe6-4630-9e8b-f0f3b7baacea"],
      :dest "/nextjournal-github-explorer-test/deps.edn"}]},
   "7b1dbb98-f369-4b7f-bf25-97bc6bf2e16f"
   {:id "7b1dbb98-f369-4b7f-bf25-97bc6bf2e16f", :kind "code-listing"},
   "7ed7180a-622e-423b-bfaa-9714860090ed"
   {:id "7ed7180a-622e-423b-bfaa-9714860090ed", :kind "code-listing"},
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
   "ae07508d-df0d-4a59-bde6-607bb3437bfd"
   {:id "ae07508d-df0d-4a59-bde6-607bb3437bfd", :kind "code-listing"},
   "b179eaf4-3fe6-4f9d-8b99-42027708b512"
   {:compute-ref #uuid "2b7508e5-c0ca-4ffd-9448-53a41348c159",
    :exec-duration 3446,
    :id "b179eaf4-3fe6-4f9d-8b99-42027708b512",
    :kind "code",
    :output-log-lines {},
    :runtime [:runtime "72545a3d-110b-44f0-82ff-8c8e0a0824a9"]},
   "b8937176-f6d7-4370-aca2-35ed8b725358"
   {:id "b8937176-f6d7-4370-aca2-35ed8b725358",
    :kind "code",
    :runtime [:runtime "72545a3d-110b-44f0-82ff-8c8e0a0824a9"]},
   "ba4fbe14-3d9d-436e-9409-1a13f2a72dbe"
   {:id "ba4fbe14-3d9d-436e-9409-1a13f2a72dbe", :kind "code-listing"},
   "c8a1cf62-a8eb-4ce4-bc7e-88733e109bdc"
   {:id "c8a1cf62-a8eb-4ce4-bc7e-88733e109bdc", :kind "code-listing"},
   "ce124c08-d13b-4bbe-9f6f-d994f77cf2ca"
   {:id "ce124c08-d13b-4bbe-9f6f-d994f77cf2ca",
    :kind "code",
    :runtime [:runtime "72545a3d-110b-44f0-82ff-8c8e0a0824a9"]},
   "d005a0af-0d2d-4a5c-a060-8747158523db"
   {:id "d005a0af-0d2d-4a5c-a060-8747158523db", :kind "code-listing"},
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
   "f35c6fbc-a3a1-4c5a-898e-b502bf0707ef"
   {:id "f35c6fbc-a3a1-4c5a-898e-b502bf0707ef", :kind "code-listing"},
   "f8294e3d-5a0a-48ee-a108-c7950ab1488a"
   {:id "f8294e3d-5a0a-48ee-a108-c7950ab1488a", :kind "code-listing"}},
  :nextjournal/id #uuid "e2c4e8ea-dd45-4bea-b788-9257c5408659",
  :article/change
  {:nextjournal/id #uuid "60b4a47f-f3df-40e3-acf2-a48a2e6fc9b9"}}}

```
</details>
