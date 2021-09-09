# Learn XTDB Datalog Today

Learn XTDB Datalog Today is derived from the classic [learndatalogtoday.org](http://learndatalogtoday.org) tutorial materials, but adapted to focus on the [xtdb](https://xtdb.com) API and unique Datalog semantics, and extended with additional topics and exercises. It is an interactive tutorial designed to teach you the xtdb dialect of Datalog. Datalog is a declarative database query language with roots in logic programming. Datalog has similar expressive power to SQL.

![XTDB Datalog Icon](https://github.com/xtdb/learn-xtdb-datalog-today/raw/main/images/find-where-icon.png)

You can follow along by hitting the "Remix" button on the toolbar to start your own interactive session running entirely on the Nextjournal platform, or you can copy the steps into a Clojure REPL running locally on your machine. You could also use curl (or similar) to send the data and queries in this tutorial via HTTP to a [pre-built Docker image](https://xtdb.com/reference/installation.html#docker).

The `master.md` source file for this notebook is available on [GitHub](https://github.com/xtdb/learn-xtdb-datalog-today).

# Runtime Setup

You need to get xtdb running before you can use it. Here we are using Clojure on the JVM and running xtdb locally, as an embedded in-memory library. However, the Datalog query API is identical for all clients and all these queries would work equally well over the network via HTTP and the Java/Clojure client library.

```edn no-exec id=d2a8a640-910f-4566-94fb-6479ed92f7b9
{:deps
 {org.clojure/clojure {:mvn/version "1.10.0"}
  org.clojure/tools.deps.alpha
  {:git/url "https://github.com/clojure/tools.deps.alpha.git"
   :sha "f6c080bd0049211021ea59e516d1785b08302515"}
  com.xtdb/xtdb-core {:mvn/version "dev-SNAPSHOT"}} ;; "RELEASE"

  :mvn/repos
  {"snapshots" {:url "https://s01.oss.sonatype.org/content/repositories/snapshots"}}}
```

```clojure id=d3b699f1-702e-4aad-8657-4c419e14e88d
(require '[xtdb.api :as xt])
```

Next we need some data. XTDB interprets maps as "documents." These require no pre-defined schema -- they only need a valid ID attribute. In the data below (which is defined using edn, and fully explained in the section) we are using negative integers as IDs. Any top-level attributes that refer to these integers can be interpreted in an ad-hoc way and traversed by the query engine -- this capability is known as "schema-on-read".

This vector of maps contains two kinds of documents: documents relating to people (actors and directors) and documents relating to movies. As a convention to aid human interpretation, all persons have IDs like `-1XX` and all movies have IDs like `-2XX`. Many ID value types are supported, such as strings and UUIDs, which may be more appropriate in a real application.

```clojure id=e2682518-935d-4a09-af39-210cf8286d33
(def my-docs
  [{:person/name "James Cameron",
    :person/born #inst "1954-08-16T00:00:00.000-00:00",
    :xt/id -100}
   {:person/name "Arnold Schwarzenegger",
    :person/born #inst "1947-07-30T00:00:00.000-00:00",
    :xt/id -101}
   {:person/name "Linda Hamilton",
    :person/born #inst "1956-09-26T00:00:00.000-00:00",
    :xt/id -102}
   {:person/name "Michael Biehn",
    :person/born #inst "1956-07-31T00:00:00.000-00:00",
    :xt/id -103}
   {:person/name "Ted Kotcheff",
    :person/born #inst "1931-04-07T00:00:00.000-00:00",
    :xt/id -104}
   {:person/name "Sylvester Stallone",
    :person/born #inst "1946-07-06T00:00:00.000-00:00",
    :xt/id -105}
   {:person/name "Richard Crenna",
    :person/born #inst "1926-11-30T00:00:00.000-00:00",
    :person/death #inst "2003-01-17T00:00:00.000-00:00",
    :xt/id -106}
   {:person/name "Brian Dennehy",
    :person/born #inst "1938-07-09T00:00:00.000-00:00",
    :xt/id -107}
   {:person/name "John McTiernan",
    :person/born #inst "1951-01-08T00:00:00.000-00:00",
    :xt/id -108}
   {:person/name "Elpidia Carrillo",
    :person/born #inst "1961-08-16T00:00:00.000-00:00",
    :xt/id -109}
   {:person/name "Carl Weathers",
    :person/born #inst "1948-01-14T00:00:00.000-00:00",
    :xt/id -110}
   {:person/name "Richard Donner",
    :person/born #inst "1930-04-24T00:00:00.000-00:00",
    :xt/id -111}
   {:person/name "Mel Gibson",
    :person/born #inst "1956-01-03T00:00:00.000-00:00",
    :xt/id -112}
   {:person/name "Danny Glover",
    :person/born #inst "1946-07-22T00:00:00.000-00:00",
    :xt/id -113}
   {:person/name "Gary Busey",
    :person/born #inst "1944-07-29T00:00:00.000-00:00",
    :xt/id -114}
   {:person/name "Paul Verhoeven",
    :person/born #inst "1938-07-18T00:00:00.000-00:00",
    :xt/id -115}
   {:person/name "Peter Weller",
    :person/born #inst "1947-06-24T00:00:00.000-00:00",
    :xt/id -116}
   {:person/name "Nancy Allen",
    :person/born #inst "1950-06-24T00:00:00.000-00:00",
    :xt/id -117}
   {:person/name "Ronny Cox",
    :person/born #inst "1938-07-23T00:00:00.000-00:00",
    :xt/id -118}
   {:person/name "Mark L. Lester",
    :person/born #inst "1946-11-26T00:00:00.000-00:00",
    :xt/id -119}
   {:person/name "Rae Dawn Chong",
    :person/born #inst "1961-02-28T00:00:00.000-00:00",
    :xt/id -120}
   {:person/name "Alyssa Milano",
    :person/born #inst "1972-12-19T00:00:00.000-00:00",
    :xt/id -121}
   {:person/name "Bruce Willis",
    :person/born #inst "1955-03-19T00:00:00.000-00:00",
    :xt/id -122}
   {:person/name "Alan Rickman",
    :person/born #inst "1946-02-21T00:00:00.000-00:00",
    :xt/id -123}
   {:person/name "Alexander Godunov",
    :person/born #inst "1949-11-28T00:00:00.000-00:00",
    :person/death #inst "1995-05-18T00:00:00.000-00:00",
    :xt/id -124}
   {:person/name "Robert Patrick",
    :person/born #inst "1958-11-05T00:00:00.000-00:00",
    :xt/id -125}
   {:person/name "Edward Furlong",
    :person/born #inst "1977-08-02T00:00:00.000-00:00",
    :xt/id -126}
   {:person/name "Jonathan Mostow",
    :person/born #inst "1961-11-28T00:00:00.000-00:00",
    :xt/id -127}
   {:person/name "Nick Stahl",
    :person/born #inst "1979-12-05T00:00:00.000-00:00",
    :xt/id -128}
   {:person/name "Claire Danes",
    :person/born #inst "1979-04-12T00:00:00.000-00:00",
    :xt/id -129}
   {:person/name "George P. Cosmatos",
    :person/born #inst "1941-01-04T00:00:00.000-00:00",
    :person/death #inst "2005-04-19T00:00:00.000-00:00",
    :xt/id -130}
   {:person/name "Charles Napier",
    :person/born #inst "1936-04-12T00:00:00.000-00:00",
    :person/death #inst "2011-10-05T00:00:00.000-00:00",
    :xt/id -131}
   {:person/name "Peter MacDonald", :xt/id -132}
   {:person/name "Marc de Jonge",
    :person/born #inst "1949-02-16T00:00:00.000-00:00",
    :person/death #inst "1996-06-06T00:00:00.000-00:00",
    :xt/id -133}
   {:person/name "Stephen Hopkins", :xt/id -134}
   {:person/name "Ruben Blades",
    :person/born #inst "1948-07-16T00:00:00.000-00:00",
    :xt/id -135}
   {:person/name "Joe Pesci",
    :person/born #inst "1943-02-09T00:00:00.000-00:00",
    :xt/id -136}
   {:person/name "Ridley Scott",
    :person/born #inst "1937-11-30T00:00:00.000-00:00",
    :xt/id -137}
   {:person/name "Tom Skerritt",
    :person/born #inst "1933-08-25T00:00:00.000-00:00",
    :xt/id -138}
   {:person/name "Sigourney Weaver",
    :person/born #inst "1949-10-08T00:00:00.000-00:00",
    :xt/id -139}
   {:person/name "Veronica Cartwright",
    :person/born #inst "1949-04-20T00:00:00.000-00:00",
    :xt/id -140}
   {:person/name "Carrie Henn", :xt/id -141}
   {:person/name "George Miller",
    :person/born #inst "1945-03-03T00:00:00.000-00:00",
    :xt/id -142}
   {:person/name "Steve Bisley",
    :person/born #inst "1951-12-26T00:00:00.000-00:00",
    :xt/id -143}
   {:person/name "Joanne Samuel", :xt/id -144}
   {:person/name "Michael Preston",
    :person/born #inst "1938-05-14T00:00:00.000-00:00",
    :xt/id -145}
   {:person/name "Bruce Spence",
    :person/born #inst "1945-09-17T00:00:00.000-00:00",
    :xt/id -146}
   {:person/name "George Ogilvie",
    :person/born #inst "1931-03-05T00:00:00.000-00:00",
    :xt/id -147}
   {:person/name "Tina Turner",
    :person/born #inst "1939-11-26T00:00:00.000-00:00",
    :xt/id -148}
   {:person/name "Sophie Marceau",
    :person/born #inst "1966-11-17T00:00:00.000-00:00",
    :xt/id -149}
   {:movie/title "The Terminator",
    :movie/year 1984,
    :movie/director -100,
    :movie/cast [-101 -102 -103],
    :movie/sequel -207,
    :xt/id -200}
   {:movie/title "First Blood",
    :movie/year 1982,
    :movie/director -104,
    :movie/cast [-105 -106 -107],
    :movie/sequel -209,
    :xt/id -201}
   {:movie/title "Predator",
    :movie/year 1987,
    :movie/director -108,
    :movie/cast [-101 -109 -110],
    :movie/sequel -211,
    :xt/id -202}
   {:movie/title "Lethal Weapon",
    :movie/year 1987,
    :movie/director -111,
    :movie/cast [-112 -113 -114],
    :movie/sequel -212,
    :xt/id -203}
   {:movie/title "RoboCop",
    :movie/year 1987,
    :movie/director -115,
    :movie/cast [-116 -117 -118],
    :xt/id -204}
   {:movie/title "Commando",
    :movie/year 1985,
    :movie/director -119,
    :movie/cast [-101 -120 -121],
    :trivia
    "In 1986, a sequel was written with an eye to having\n  John McTiernan direct. Schwarzenegger wasn't interested in reprising\n  the role. The script was then reworked with a new central character,\n  eventually played by Bruce Willis, and became Die Hard",
    :xt/id -205}
   {:movie/title "Die Hard",
    :movie/year 1988,
    :movie/director -108,
    :movie/cast [-122 -123 -124],
    :xt/id -206}
   {:movie/title "Terminator 2: Judgment Day",
    :movie/year 1991,
    :movie/director -100,
    :movie/cast [-101 -102 -125 -126],
    :movie/sequel -208,
    :xt/id -207}
   {:movie/title "Terminator 3: Rise of the Machines",
    :movie/year 2003,
    :movie/director -127,
    :movie/cast [-101 -128 -129],
    :xt/id -208}
   {:movie/title "Rambo: First Blood Part II",
    :movie/year 1985,
    :movie/director -130,
    :movie/cast [-105 -106 -131],
    :movie/sequel -210,
    :xt/id -209}
   {:movie/title "Rambo III",
    :movie/year 1988,
    :movie/director -132,
    :movie/cast [-105 -106 -133],
    :xt/id -210}
   {:movie/title "Predator 2",
    :movie/year 1990,
    :movie/director -134,
    :movie/cast [-113 -114 -135],
    :xt/id -211}
   {:movie/title "Lethal Weapon 2",
    :movie/year 1989,
    :movie/director -111,
    :movie/cast [-112 -113 -136],
    :movie/sequel -213,
    :xt/id -212}
   {:movie/title "Lethal Weapon 3",
    :movie/year 1992,
    :movie/director -111,
    :movie/cast [-112 -113 -136],
    :xt/id -213}
   {:movie/title "Alien",
    :movie/year 1979,
    :movie/director -137,
    :movie/cast [-138 -139 -140],
    :movie/sequel -215,
    :xt/id -214}
   {:movie/title "Aliens",
    :movie/year 1986,
    :movie/director -100,
    :movie/cast [-139 -141 -103],
    :xt/id -215}
   {:movie/title "Mad Max",
    :movie/year 1979,
    :movie/director -142,
    :movie/cast [-112 -143 -144],
    :movie/sequel -217,
    :xt/id -216}
   {:movie/title "Mad Max 2",
    :movie/year 1981,
    :movie/director -142,
    :movie/cast [-112 -145 -146],
    :movie/sequel -218,
    :xt/id -217}
   {:movie/title "Mad Max Beyond Thunderdome",
    :movie/year 1985,
    :movie/director [-142 -147],
    :movie/cast [-112 -148],
    :xt/id -218}
   {:movie/title "Braveheart",
    :movie/year 1995,
    :movie/director [-112],
    :movie/cast [-112 -149],
    :xt/id -219}])
```

Note that xtdb also has a JSON-over-HTTP API that naturally supports JSON documents, this is possible because JSON can be very simply mapped to a subset of edn.

To start an in-memory instance of xtdb, you can use the `start-node` function like so:

```clojure id=ec1250a9-3a5f-4ef8-a19f-ab2ac32f281b
(def my-node (xt/start-node {}))
```

Loading the small amount of data we defined under `my-docs` above can be comfortably done in a single transaction. In practice you will often find benefit to batch `put` operations into groups of 1000 at a time. The following code maps over the docs to generate a single transaction containing one `:xtdb.api/put` operation per document, then submits the transaction. Finally we call the `sync` function to ensure that the documents are fully indexed (and that the transaction has succeeded) before we attempt to run any queries -- this is necessary because of xtdb's asynchronous design.

```clojure id=6b4e42ab-8163-4d9d-b775-2691256d875c
(xt/submit-tx my-node (for [doc my-docs]
                          [:xtdb.api/put doc]))

(xt/sync my-node)
```

With xtdb running and the data loaded, you can now execute a query, which is a Clojure map, by passing it to xtdb's `q` API, which takes the result of a `db` call as it's first value. The meaning of this query will become apparent very soon!

```clojure id=8871da49-dac6-469b-a273-e4a2a87ff848
(xt/q (xt/db my-node)
      '{:find [title]
        :where [[_ :movie/title title]]})
```

To simplify this `xt/q` call throughout the rest of the tutorial we can define a new `q` function that saves us a few characters and visual clutter.

```clojure id=19dd5ef4-1841-4287-8c56-82543565182a
(defn q [query & args]
  (apply xt/q (xt/db my-node) query args))
```

Queries can then be executed trivially:

```clojure id=db46f3e2-70a3-4768-bef5-2a48d246be9e
(q '{:find [title]
     :where [[_ :movie/title title]]})
```

# Extensible Data Notation

In xtdb, a Datalog query is written in [extensible data notation (edn)](http://edn-format.org). Edn is a data format similar to JSON, but it:

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

```edn no-exec id=ab2ea701-a11a-4ef9-a3c8-51227e81ef1c
    {:find [title]
     :where [[_ :movie/title title]]}
```

Note that the query is a map with two key-value pairs:

* the `:find` vector containing the symbol `title`
* the `:where` vector containing a single query clause `[_ :movie/title title]` (which is also a vector)

XTDB also supports queries in an alternative "vector" format:

```edn no-exec id=fceef220-fe9c-4da0-80c5-1ab5b6f6a084
    [:find ?title
     :where
     [_ :movie/title ?title]]
```

However, in this tutorial we will use the map format. Also note that xtdb does not require logical variables to be preceded by `?`, although you may use this convention if you wish.

## Exercises

Q1. Find all the movie titles in the database

```clojure id=b66fbac8-5ef0-4f69-beee-152c2b9f4ac7
;; remove '#_' to uncomment the query
#_(q '{:find ... })
```

## Solutions

A1.

```clojure id=c6b02497-6495-477a-ac9b-c3a1438ede77
(q '{:find [title]
     :where [[_ :movie/title title]]})
```

# Basic Queries

The example database we'll use contains *movies* mostly, but not exclusively, from the 80s. You'll find information about movie titles, release year, directors, cast members, etc. As the tutorial advances we'll learn more about the contents of the database and how it's organized.

The data model in xtdb is based around *atomic collections of facts.* Those atomic collections of facts are called *documents.* The facts are called triples. A triple is a 3-tuple consisting of:

* Entity ID
* Attribute
* Value

Although it is the document which is atomic in xtdb (and not the triple), you can think of the database as a flat **set of triples** of the form:

```no-exec id=75319a3b-f798-4f22-80be-3be49cc69e2d
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

```clojure id=85e976a8-d317-48fc-aa2b-558cc71b4e5d
(q '{:find [e]
     :where [[e :person/name "Ridley Scott"]]})
```

The simplest data pattern is a triple with some parts replaced with logic variables. It is the job of the query engine to figure out every possible value of each of the logic variables and return the ones that are specified in the `:find` clause.

The symbol `_` can be used as a wildcard for the parts of the data pattern that you wish to ignore. You can also elide trailing values in a data pattern. Therefore, the following two queries are equivalent.

```clojure id=4966a282-80c4-4402-ba58-8d2e3d05f4d2
(q '{:find [e]
     :where [[e :person/name _]]})
```

```clojure id=db7c287c-4e6b-47ae-8716-2254b37981a1
(q '{:find [e]
     :where [[e :person/name]]})
```

## Exercises

Q1. Find the entity ids of movies made in 1987

```clojure id=8f6facdd-9f71-4681-98ff-8a1613b70a8b
;; remove '#_' to uncomment the query
#_(q '{:find [e]
     :where ...})
```

Q2. Find the entity-id and titles of movies in the database

```clojure id=2d8b5c40-b932-4c4e-951c-228155f8796d
;; remove '#_' to uncomment the query
#_(q '{:find [e title]
     :where ...})
```

Q3. Find the name of all people in the database

```clojure id=d8dcf3db-a888-46ac-ad35-942c7374e223
;; remove '#_' to uncomment the query
#_(q '{:find ...})
```

## Solutions

A1.

```clojure id=11a5b364-f0a3-4a51-a7a0-41eb313f0b10
(q '{:find [e]
     :where [[e :movie/year 1987]]})
```

A2.

```clojure id=95566a23-02bc-4969-93b8-31ed19e026d4
(q '{:find [e]
     :where [[e :movie/title title]]})
```

A3.

```clojure id=8d7e7afe-edb4-4466-8810-23bdfd49ea91
(q '{:find [name]
     :where [[_ :person/name name]]})
```

# Data patterns

In the previous chapter, we looked at **data patterns**, i.e., vectors within the `:where` vector, such as `[e :movie/title "Commando"]`. There can be many data patterns in a `:where` clause:

```clojure id=f7fcf3ff-359e-4115-abda-7324679d4da0
(q '{:find [title]
     :where [[e :movie/year 1987]
             [e :movie/title title]]})
```

The important thing to note here is that the logic variable `e` is used in both data patterns. When a logic variable is used in multiple places, the query engine requires it to be bound to the same value in each place. Therefore, this query will only find movie titles for movies made in 1987.

The order of the data patterns does not matter. XTDB ignores the user-provided clause ordering so the query engine can optimize query execution. Thus, the previous query could just as well have been written this way:

```clojure id=6bc9f6ea-e5a9-4aaf-9f5f-c1c8b4c31066
(q '{:find [title]
     :where [[e :movie/title title]
             [e :movie/year 1987]]})
```

In both cases, the result set will be exactly the same.

Let's say we want to find out who starred in "Lethal Weapon". We will need three data patterns for this. The first one finds the entity ID of the movie with "Lethal Weapon" as the title:

```no-exec id=5b589a91-5fd8-456e-ab74-3d36ce3ea3eb
[m :movie/title "Lethal Weapon"]
```

Using the same entity ID at `m`, we can find the cast members with the data pattern:

```no-exec id=7b740297-1079-467f-acdf-499da02c6d20
[m :movie/cast p]
```

In this pattern, `p` will now be (the entity ID of) a person entity, so we can grab the actual name with:

```no-exec id=a40bc6d0-77d1-4cce-9c3e-cc6fa7c24be7
[p :person/name name]
```

The query will therefore be:

```no-exec id=a46ab456-d97b-4434-a0b7-fc9f07c482e6
{:find [name]
 :where [[m :movie/title "Lethal Weapon"]
         [m :movie/cast p]
         [p :person/name name]]}
```

## Exercises

Q1. Find movie titles made in 1985

```clojure id=ea0ed77b-5507-4c8f-93b2-e06fc7a213fd
;; remove '#_' to uncomment the query
#_(q '{:find [title]
     :where ...})
```

Q2. What year was "Alien" released?

```clojure id=f91a07ba-5871-42e6-8da6-763d5053b225
;; remove '#_' to uncomment the query
#_(q '{:find [year]
     :where ...})
```

Q3. Who directed RoboCop? You will need to use `[<movie-eid> :movie/director <person-eid>]` to find the director for a movie.

```clojure id=37564289-6807-4c6f-bf23-dfbe7d58a696
;; remove '#_' to uncomment the query
#_(q '{:find [name]
     :where ...})
```

Q4. Find directors who have directed Arnold Schwarzenegger in a movie.

```clojure id=14eb2f22-05fc-4a4a-9b72-6fe1a1debbba
;; remove '#_' to uncomment the query
#_(q '{:find [name]
     :where ...})
```

## Solutions

A1.

```clojure id=53279af1-08aa-42ba-8744-8ce8c16da680
(q '{:find [title]
     :where [[m :movie/title title]
             [m :movie/year 1985]]})
```

A2.

```clojure id=614e6e34-ffb0-48de-8b64-b76f0fb48810
(q '{:find [year]
     :where [[m :movie/title "Alien"]
             [m :movie/year year]]})
```

A3.

```clojure id=acda780d-9072-4c3d-8a62-8365d40fcab0
(q '{:find [name]
     :where [[m :movie/title "RoboCop"]
             [m :movie/director d]
             [d :person/name name]]})
```

A4.

```clojure id=1bcad698-29ad-41c9-8d4e-4131fd28441e
(q '{:find [name]
     :where [[p :person/name "Arnold Schwarzenegger"]
             [m :movie/cast p]
             [m :movie/director d]
             [d :person/name name]]})
```

# Parameterized queries

Looking at this query:

```clojure id=3dcd9665-291d-42c2-82e4-342d04f48ed4
(q '{:find [title]
     :where [[p :person/name "Sylvester Stallone"]
             [m :movie/cast p]
             [m :movie/title title]]})
```

It would be great if we could reuse this query to find movie titles for any actor and not just for "Sylvester Stallone". This is possible with an `:in` clause, which provides the query with input parameters, much in the same way that function or method arguments do in your programming language.

Here's that query with an input parameter for the actor:

```clojure id=d092a2fb-9540-4f6b-b747-77ff36fad5ac
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

```clojure id=2261fff1-957b-4083-bc59-275fb281dbc3
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

```no-exec id=cc8883de-ad05-4b62-897b-ffe5b754b061
:in [director actor]
```

## Collections

You can use collection destructuring to implement a kind of *logical OR* in your query. Say you want to find all movies directed by either James Cameron **or** Ridley Scott:

```clojure id=58615e62-1b6d-45c9-b881-80f29852d4e7
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

```no-exec id=6ea741d1-73e7-43a0-9dad-d6cd29c4be85
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

```clojure id=4e836a89-6931-4900-852d-f0bed55973ba
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

```clojure id=281ea83d-606a-4498-8385-bcbcdfb8620b
;; remove '#_' to uncomment the query
#_(q '{:find [title]
     :in [year]
     :where ...})
```

Q2. Given a list of movie titles, find the title and the year that movie was released.

```clojure id=736d1574-844c-485f-aadb-d64190640dd7
;; remove '#_' to uncomment the query
#_(q '{:find [title year]
     :in ...
     :where ...})
```

Q3. Find all movie `title`s where the `actor` and the `director` has worked together

```clojure id=00afe771-ce3f-4545-b055-8e49319dcf46
;; remove '#_' to uncomment the query
#_(q '{:find [title]
     :in [actor director]
     :where ...})
```

Q4. Write a query that, given an actor name and a relation with movie-title/rating, finds the movie titles and corresponding rating for which that actor was a cast member.

```clojure id=34015421-038d-4daf-928f-869ea07d717c
;; remove '#_' to uncomment the query
#_(q '{:find [title rating]
     :in ...
     :where ...})
```

## Solutions

A1.

```clojure id=708797d2-acc3-4f59-8025-b499da4f9716
(q '{:find [title]
     :in [year]
     :where [[m :movie/year year]
             [m :movie/title title]]}
   1988)
```

A2.

```clojure id=8fffb866-4d1d-4353-8748-f0e9d4c770c1
(q '{:find [title year]
     :in [[title ...]]
     :where [[m :movie/title title]
             [m :movie/year year]]}
   ["Lethal Weapon" "Lethal Weapon 2" "Lethal Weapon 3"])
```

A3.

```clojure id=b16b103d-ff19-4d85-a3e4-3013be6894a8
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

```clojure id=f9e67c81-c05b-4278-976d-24726430a8b0
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

```clojure id=2397ec93-bfb7-4cc3-8bc6-87974aa7565e
(q '{:find [title]
     :where [[m :movie/title title]
             [m :movie/year year]
             [(< year 1984)]]})
```

The last clause, `[(< year 1984)]`, is a predicate clause. The predicate clause filters the result set to only include results for which the predicate returns a "truthy" (non-nil, non-false) value. You can use any Clojure function as a predicate function:

```clojure id=6aacc5ea-937b-4303-ab4c-2e2ae7f83476
(q '{:find [name]
     :where [[p :person/name name]
             [(clojure.string/starts-with? name "M")]]})
```

Clojure functions must be fully namespace-qualified, so if you have defined your own predicate `awesome?` you must write it as `(my.namespace/awesome? movie)`. All `clojure.core/*` functions may be used as predicates without namespace qualification: `<, >, <=, >=, =, not=` and so on. XTDB provides a ["Predicate AllowList"](https://xtdb.com/reference/query-configuration.html#fn-allowlist) feature to restrict the exact set of predicates available to queries.

## Exercises

Q1. Find movies older than a certain year (inclusive)

```clojure id=509c034b-b023-49b7-bfc2-5d803fc61eeb
;; remove '#_' to uncomment the query
#_(q '{:find [title]
     :in [year]
     :where ...})
```

Q2. Find actors older than Danny Glover

```clojure id=54ff648c-1691-4691-8f10-06bc674d7ffb
;; remove '#_' to uncomment the query
#_(q '{:find [actor]
     :where ...})
```

Q3. Find movies newer than `year` (inclusive) and has a `rating` higher than the one supplied

```clojure id=ba036603-2fa1-4a4c-bc2b-d88017b97c7c
;; remove '#_' to uncomment the query
#_(q '{:find [title]
     :in [year rating [[title r]]]
     :where ...})
```

## Solutions

A1.

```clojure id=8ece8520-e980-4682-a44f-becb64840c7c
(q '{:find [title]
     :in [year]
     :where [[m :movie/title title]
             [m :movie/year y]
             [(<= y year)]]}
   1979)
```

A2.

```clojure id=573ea588-990e-457e-b1f6-a5435141880d
(q '{:find [actor]
     :where [[d :person/name "Danny Glover"]
             [d :person/born b1]
             [e :person/born b2]
             [_ :movie/cast e]
             [(< b2 b1)]
             [e :person/name actor]]})
```

A3.

```clojure id=eea2322c-b011-4d79-bedd-1bdb35deee34
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

```clojure id=ab46712b-c4cd-4ffd-af74-baee71533aee
(defn age [^java.util.Date birthday ^java.util.Date today]
  (quot (- (.getTime today)
          (.getTime birthday))
        (* 1000 60 60 24 365)))
```

With this function, we can now calculate the age of a person **inside the query itself**:

```clojure id=da8488cb-de60-4c7a-9706-849e775102a3
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

```no-exec id=bb9a6631-d31a-4585-a1ac-4fbd3dd16659
[(f (g x)) a]
```

Instead, you must bind intermediate results in temporary logic variables:

```no-exec id=ec915b4e-615e-4ee8-a110-f701fe42b677
[(g x) t]
[(f t) a]
```

## Exercises

Q1. Find people by age. Use the function `user/age` to find the age given a birthday and a date representing "today".

```clojure id=efe159a6-33df-481f-ad9c-751f4afebf48
;; remove '#_' to uncomment the query
#_(q '{:find [name]
     :in [age today]
     :where ...})
```

Q2. Find people younger than Bruce Willis and their ages.

```clojure id=b94a242d-969e-40df-8a4f-f37325738cd3
;; remove '#_' to uncomment the query
#_(q '{:find [name age]
     :in [today]
     :where ...})
```

## Solutions

A1.

```clojure id=11018ebc-c60e-4b86-8aba-29d8bfb8358b
(q '{:find [name]
     :in [age today]
     :where [[p :person/name name]
             [p :person/born born]
             [(user/age born today) age]]}
   63
   #inst "2013-08-02T00:00:00.000-00:00")
```

A2.

```clojure id=ad3acca5-10ed-4e6b-83cf-1f2ebce40f77
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

Aggregate functions such as `sum`, `max` etc. are readily available in xtdb's Datalog implementation. They are written in the `:find` clause in your query:

```no-exec id=c1bab13d-ff15-43a5-bbc3-b9dd242d10e6
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

```clojure id=09998a40-7b35-462a-92f4-387706b0d559
(q '{:find [(count m)]
     :where [[m :movie/title]]})
```

A2.

```clojure id=fe781e94-0e50-4dd6-90c8-0df43dfa9ec1
(q '{:find [(min date)]
     :where [[_ :person/born date]]})
```

A3.

```clojure id=78896ccc-7cab-49c4-a0ad-9d9fe90ca5a5
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

```no-exec id=6ba3ebd7-017c-46a0-b620-9f379968149a
[p :person/name name]
[m :movie/cast p]
[m :movie/title title]
```

**Rules** are the means of abstraction in Datalog. You can abstract away reusable parts of your queries into rules, give them meaningful names and forget about the implementation details, just like you can with functions in your favorite programming language. Let's create a rule for the three lines above:

```no-exec id=dbe5cfa2-9a01-4c1c-b0ec-eda50ffec27a
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

```clojure id=873b47ea-ff23-4ba4-bee4-da4aac072efb
(q '{:find [name]
     :where [[p :person/name name]
             [m :movie/cast p]
             [m :movie/title "The Terminator"]]})
```

Now becomes:

```clojure id=c2bce90a-ce64-42a2-9e70-bdd65bc53308
(q '{:find [name]
     :where [(actor-movie name "The Terminator")]
     :rules [[(actor-movie name title)
              [p :person/name name]
              [m :movie/cast p]
              [m :movie/title "The Terminator"]]]})
```

You can write any number of rules, collect them in a vector, and pass them to the query engine using the `:rules` key, as above.

```no-exec id=b2e2c9e1-4597-4b19-bb4f-e83a0a45e8bd
[[(rule-a a b)
  ...]
 [(rule-b a b)
  ...]
 ...]
```

You can use [data patterns](/chapter/2), [predicates](/chapter/5), [transformation functions](/chapter/6) and calls to other rules in the body of a rule.

Rules can also be used as another tool to write *logical OR* queries, as the same rule name can be used several times:

```no-exec id=2a7e1dd3-27ba-498f-a44a-a4da3b79d42a
[[(associated-with person movie)
  [movie :movie/cast person]]
 [(associated-with person movie)
  [movie :movie/director person]]]
```

Subsequent rule definitions will only be used if the ones preceding it aren't satisfied.

Using this rule, we can find both directors and cast members very easily:

```clojure id=2b823a94-2659-4e91-8263-8e28b8401386
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

```clojure id=ff921ee6-94b1-4e88-843b-ecdb0fb442f9
;; remove '#_' to uncomment the query
#_(q '{:find [title]
     :where [(movie-year title 1991)]
     :rules [[(movie-year title year)
              ...]]})
```

Q2. Two people are friends if they have worked together in a movie. Write a rule `(friends p1 p2)` where `p1` and `p2` are person entities. Try with a few different `name` inputs to make sure you got it right. There might be some edge cases here.

```clojure id=490e3a0b-4a69-4777-914e-bbe26cf3e71c
;; remove '#_' to uncomment the query
#_(q '{:find [friend]
     :in [name]
     :where [[p1 :person/name name]
             (friends p1 p2)
             [p2 :person/name friend]]
     :rules [[(friends p1 p2)
              ...]]})
```

## Solutions

A1.

```clojure id=6f2472b3-4d89-4e95-89f3-348417e86617
(q '{:find [title]
     :where [(movie-year title 1991)]
     :rules [[(movie-year title year)
              [m :movie/title title]
              [m :movie/year year]]]})
```

A2.

```clojure id=29464535-aa4b-489f-948d-cf293f3cab23
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
              [?m :movie/director ?p2]
              [(not= ?p1 ?p2)]]]}
  "Sigourney Weaver")
```

# Conclusion

Congratulations for making it through the tutorial - we hope this knowledge helps you in your Datalog journey! Any and all feedback is appreciated, as are new contributions, please email [hello@xtdb.com](mailto:hello@xtdb.com) or open an issue via [GitHub](https://github.com/xtdb/learn-xtdb-datalog-today)

# Copyright & License

The MIT License (MIT)

Copyright © 2013 - 2021 Jonas Enlund

Permission is hereby granted, free of charge, to any person obtaining a copy of this software and associated documentation files (the "Software"), to deal in the Software without restriction, including without limitation the rights to use, copy, modify, merge, publish, distribute, sublicense, and/or sell copies of the Software, and to permit persons to whom the Software is furnished to do so, subject to the following conditions:

The above copyright notice and this permission notice shall be included in all copies or substantial portions of the Software.

THE SOFTWARE IS PROVIDED "AS IS", WITHOUT WARRANTY OF ANY KIND, EXPRESS OR IMPLIED, INCLUDING BUT NOT LIMITED TO THE WARRANTIES OF MERCHANTABILITY, FITNESS FOR A PARTICULAR PURPOSE AND NONINFRINGEMENT. IN NO EVENT SHALL THE AUTHORS OR COPYRIGHT HOLDERS BE LIABLE FOR ANY CLAIM, DAMAGES OR OTHER LIABILITY, WHETHER IN AN ACTION OF CONTRACT, TORT OR OTHERWISE, ARISING FROM, OUT OF OR IN CONNECTION WITH THE SOFTWARE OR THE USE OR OTHER DEALINGS IN THE SOFTWARE.

# Thank You

Thank you Jonas and contributors for freely licensing your excellent materials!


<details id="com.nextjournal.article">
<summary>This notebook was exported from <a href="https://nextjournal.com/a/PFvcmonXcQS6w4eRjvLY9?change-id=CzkmoRCLzAWWZFjJAw2gxP">https://nextjournal.com/a/PFvcmonXcQS6w4eRjvLY9?change-id=CzkmoRCLzAWWZFjJAw2gxP</a></summary>

```edn nextjournal-metadata
{:article
 {:nodes
  {"00afe771-ce3f-4545-b055-8e49319dcf46"
   {:compute-ref #uuid "a31838e9-ccba-47e0-9725-39d57f89fd41",
    :exec-duration 56,
    :id "00afe771-ce3f-4545-b055-8e49319dcf46",
    :kind "code",
    :output-log-lines {},
    :runtime [:runtime "b87e8658-2485-4254-8a4b-ccd6e512b1e0"]},
   "09998a40-7b35-462a-92f4-387706b0d559"
   {:compute-ref #uuid "23f34a76-3c7a-4c70-b4c3-9a2627063f59",
    :exec-duration 66,
    :id "09998a40-7b35-462a-92f4-387706b0d559",
    :kind "code",
    :output-log-lines {},
    :runtime [:runtime "b87e8658-2485-4254-8a4b-ccd6e512b1e0"]},
   "11018ebc-c60e-4b86-8aba-29d8bfb8358b"
   {:compute-ref #uuid "4d3cb3ac-1343-4c2a-8131-7c3f82a06062",
    :exec-duration 112,
    :id "11018ebc-c60e-4b86-8aba-29d8bfb8358b",
    :kind "code",
    :output-log-lines {},
    :runtime [:runtime "b87e8658-2485-4254-8a4b-ccd6e512b1e0"]},
   "11a5b364-f0a3-4a51-a7a0-41eb313f0b10"
   {:compute-ref #uuid "07ca6283-6464-470f-ab61-f878b9474e2c",
    :exec-duration 53,
    :id "11a5b364-f0a3-4a51-a7a0-41eb313f0b10",
    :kind "code",
    :output-log-lines {},
    :runtime [:runtime "b87e8658-2485-4254-8a4b-ccd6e512b1e0"]},
   "14eb2f22-05fc-4a4a-9b72-6fe1a1debbba"
   {:compute-ref #uuid "8b7044c0-cfe5-435e-8350-03c96c623b5b",
    :exec-duration 56,
    :id "14eb2f22-05fc-4a4a-9b72-6fe1a1debbba",
    :kind "code",
    :output-log-lines {},
    :runtime [:runtime "b87e8658-2485-4254-8a4b-ccd6e512b1e0"]},
   "19dd5ef4-1841-4287-8c56-82543565182a"
   {:compute-ref #uuid "247652d8-e725-4892-b160-482fefa1f445",
    :exec-duration 59,
    :id "19dd5ef4-1841-4287-8c56-82543565182a",
    :kind "code",
    :output-log-lines {},
    :runtime [:runtime "b87e8658-2485-4254-8a4b-ccd6e512b1e0"]},
   "1bcad698-29ad-41c9-8d4e-4131fd28441e"
   {:compute-ref #uuid "d0de03c4-a05e-49f1-b7e1-e2c724376ae0",
    :exec-duration 111,
    :id "1bcad698-29ad-41c9-8d4e-4131fd28441e",
    :kind "code",
    :output-log-lines {},
    :runtime [:runtime "b87e8658-2485-4254-8a4b-ccd6e512b1e0"]},
   "2261fff1-957b-4083-bc59-275fb281dbc3"
   {:compute-ref #uuid "e4a1605c-d7f5-4e6b-8cac-36f4c3001c84",
    :exec-duration 166,
    :id "2261fff1-957b-4083-bc59-275fb281dbc3",
    :kind "code",
    :output-log-lines {},
    :runtime [:runtime "b87e8658-2485-4254-8a4b-ccd6e512b1e0"]},
   "2397ec93-bfb7-4cc3-8bc6-87974aa7565e"
   {:compute-ref #uuid "11ad8b72-4ad5-4204-8780-57098a591358",
    :exec-duration 123,
    :id "2397ec93-bfb7-4cc3-8bc6-87974aa7565e",
    :kind "code",
    :output-log-lines {},
    :runtime [:runtime "b87e8658-2485-4254-8a4b-ccd6e512b1e0"]},
   "281ea83d-606a-4498-8385-bcbcdfb8620b"
   {:compute-ref #uuid "27caf5ea-82e9-4412-80e5-156333a54935",
    :exec-duration 49,
    :id "281ea83d-606a-4498-8385-bcbcdfb8620b",
    :kind "code",
    :output-log-lines {},
    :runtime [:runtime "b87e8658-2485-4254-8a4b-ccd6e512b1e0"]},
   "29464535-aa4b-489f-948d-cf293f3cab23"
   {:compute-ref #uuid "ccc4db2d-4c2f-4540-9d68-ccb8f5f6667c",
    :exec-duration 844,
    :id "29464535-aa4b-489f-948d-cf293f3cab23",
    :kind "code",
    :output-log-lines {},
    :runtime [:runtime "b87e8658-2485-4254-8a4b-ccd6e512b1e0"]},
   "2a7e1dd3-27ba-498f-a44a-a4da3b79d42a"
   {:id "2a7e1dd3-27ba-498f-a44a-a4da3b79d42a", :kind "code-listing"},
   "2b823a94-2659-4e91-8263-8e28b8401386"
   {:compute-ref #uuid "7922f68e-d956-42b6-b54b-c12887e01fa4",
    :exec-duration 418,
    :id "2b823a94-2659-4e91-8263-8e28b8401386",
    :kind "code",
    :output-log-lines {},
    :runtime [:runtime "b87e8658-2485-4254-8a4b-ccd6e512b1e0"]},
   "2d8b5c40-b932-4c4e-951c-228155f8796d"
   {:compute-ref #uuid "fa69870a-24ee-4123-bb94-fb6c4ee9c3de",
    :exec-duration 57,
    :id "2d8b5c40-b932-4c4e-951c-228155f8796d",
    :kind "code",
    :output-log-lines {},
    :runtime [:runtime "b87e8658-2485-4254-8a4b-ccd6e512b1e0"]},
   "34015421-038d-4daf-928f-869ea07d717c"
   {:compute-ref #uuid "a5417402-f55d-410b-8713-c481e25d7ffb",
    :exec-duration 53,
    :id "34015421-038d-4daf-928f-869ea07d717c",
    :kind "code",
    :output-log-lines {},
    :runtime [:runtime "b87e8658-2485-4254-8a4b-ccd6e512b1e0"]},
   "37564289-6807-4c6f-bf23-dfbe7d58a696"
   {:compute-ref #uuid "c93143f4-5c4d-4a1a-8ea5-8a5d0b716875",
    :exec-duration 51,
    :id "37564289-6807-4c6f-bf23-dfbe7d58a696",
    :kind "code",
    :output-log-lines {},
    :runtime [:runtime "b87e8658-2485-4254-8a4b-ccd6e512b1e0"]},
   "3dcd9665-291d-42c2-82e4-342d04f48ed4"
   {:compute-ref #uuid "de982188-0215-47f8-b654-17c719635cc3",
    :exec-duration 90,
    :id "3dcd9665-291d-42c2-82e4-342d04f48ed4",
    :kind "code",
    :output-log-lines {},
    :runtime [:runtime "b87e8658-2485-4254-8a4b-ccd6e512b1e0"]},
   "490e3a0b-4a69-4777-914e-bbe26cf3e71c"
   {:compute-ref #uuid "5b8d7c67-cf11-4f1b-bc64-943e55ce644c",
    :exec-duration 65,
    :id "490e3a0b-4a69-4777-914e-bbe26cf3e71c",
    :kind "code",
    :output-log-lines {},
    :runtime [:runtime "b87e8658-2485-4254-8a4b-ccd6e512b1e0"]},
   "4966a282-80c4-4402-ba58-8d2e3d05f4d2"
   {:compute-ref #uuid "0bda1d79-747e-4c65-a154-37eeedff244c",
    :exec-duration 163,
    :id "4966a282-80c4-4402-ba58-8d2e3d05f4d2",
    :kind "code",
    :output-log-lines {},
    :runtime [:runtime "b87e8658-2485-4254-8a4b-ccd6e512b1e0"]},
   "4e836a89-6931-4900-852d-f0bed55973ba"
   {:compute-ref #uuid "288816fa-4a8a-453f-b654-491cc980a78e",
    :exec-duration 136,
    :id "4e836a89-6931-4900-852d-f0bed55973ba",
    :kind "code",
    :output-log-lines {},
    :runtime [:runtime "b87e8658-2485-4254-8a4b-ccd6e512b1e0"]},
   "509c034b-b023-49b7-bfc2-5d803fc61eeb"
   {:compute-ref #uuid "e77d3523-79df-499a-9bfb-a4744883eabb",
    :exec-duration 50,
    :id "509c034b-b023-49b7-bfc2-5d803fc61eeb",
    :kind "code",
    :output-log-lines {},
    :runtime [:runtime "b87e8658-2485-4254-8a4b-ccd6e512b1e0"]},
   "53279af1-08aa-42ba-8744-8ce8c16da680"
   {:compute-ref #uuid "2e1b695c-0b16-4d9c-964e-1fc50497b13c",
    :exec-duration 57,
    :id "53279af1-08aa-42ba-8744-8ce8c16da680",
    :kind "code",
    :output-log-lines {},
    :runtime [:runtime "b87e8658-2485-4254-8a4b-ccd6e512b1e0"]},
   "54ff648c-1691-4691-8f10-06bc674d7ffb"
   {:compute-ref #uuid "045840f0-355a-4627-83c3-6238f3e4ccc4",
    :exec-duration 49,
    :id "54ff648c-1691-4691-8f10-06bc674d7ffb",
    :kind "code",
    :output-log-lines {},
    :runtime [:runtime "b87e8658-2485-4254-8a4b-ccd6e512b1e0"]},
   "573ea588-990e-457e-b1f6-a5435141880d"
   {:compute-ref #uuid "04aea8fa-a644-45eb-acce-2396f1500947",
    :exec-duration 109,
    :id "573ea588-990e-457e-b1f6-a5435141880d",
    :kind "code",
    :output-log-lines {},
    :runtime [:runtime "b87e8658-2485-4254-8a4b-ccd6e512b1e0"]},
   "58615e62-1b6d-45c9-b881-80f29852d4e7"
   {:compute-ref #uuid "b04ab0de-6647-4241-a2e6-ce0d6108bcd2",
    :exec-duration 79,
    :id "58615e62-1b6d-45c9-b881-80f29852d4e7",
    :kind "code",
    :output-log-lines {},
    :runtime [:runtime "b87e8658-2485-4254-8a4b-ccd6e512b1e0"]},
   "5b589a91-5fd8-456e-ab74-3d36ce3ea3eb"
   {:id "5b589a91-5fd8-456e-ab74-3d36ce3ea3eb", :kind "code-listing"},
   "614e6e34-ffb0-48de-8b64-b76f0fb48810"
   {:compute-ref #uuid "18300cf6-4dce-4ada-a8a0-51467e88ecf3",
    :exec-duration 103,
    :id "614e6e34-ffb0-48de-8b64-b76f0fb48810",
    :kind "code",
    :output-log-lines {},
    :runtime [:runtime "b87e8658-2485-4254-8a4b-ccd6e512b1e0"]},
   "6aacc5ea-937b-4303-ab4c-2e2ae7f83476"
   {:compute-ref #uuid "f8f3a811-8632-4bc1-942f-6dcae42f0532",
    :exec-duration 104,
    :id "6aacc5ea-937b-4303-ab4c-2e2ae7f83476",
    :kind "code",
    :output-log-lines {},
    :runtime [:runtime "b87e8658-2485-4254-8a4b-ccd6e512b1e0"]},
   "6b4e42ab-8163-4d9d-b775-2691256d875c"
   {:compute-ref #uuid "dcd5a2dd-8311-4873-bcd3-e25055ed9765",
    :exec-duration 611,
    :id "6b4e42ab-8163-4d9d-b775-2691256d875c",
    :kind "code",
    :output-log-lines {},
    :runtime [:runtime "b87e8658-2485-4254-8a4b-ccd6e512b1e0"]},
   "6ba3ebd7-017c-46a0-b620-9f379968149a"
   {:id "6ba3ebd7-017c-46a0-b620-9f379968149a", :kind "code-listing"},
   "6bc9f6ea-e5a9-4aaf-9f5f-c1c8b4c31066"
   {:compute-ref #uuid "e22ecace-89df-41e3-903d-f37110b2bf8f",
    :exec-duration 81,
    :id "6bc9f6ea-e5a9-4aaf-9f5f-c1c8b4c31066",
    :kind "code",
    :output-log-lines {},
    :runtime [:runtime "b87e8658-2485-4254-8a4b-ccd6e512b1e0"]},
   "6ea741d1-73e7-43a0-9dad-d6cd29c4be85"
   {:id "6ea741d1-73e7-43a0-9dad-d6cd29c4be85", :kind "code-listing"},
   "6f2472b3-4d89-4e95-89f3-348417e86617"
   {:compute-ref #uuid "71581e23-aa86-42d7-9d24-cb5254a7fefb",
    :exec-duration 50,
    :id "6f2472b3-4d89-4e95-89f3-348417e86617",
    :kind "code",
    :output-log-lines {},
    :runtime [:runtime "b87e8658-2485-4254-8a4b-ccd6e512b1e0"]},
   "708797d2-acc3-4f59-8025-b499da4f9716"
   {:compute-ref #uuid "4e8f54e3-a140-4c5f-98fc-1633bcba4682",
    :exec-duration 76,
    :id "708797d2-acc3-4f59-8025-b499da4f9716",
    :kind "code",
    :output-log-lines {},
    :runtime [:runtime "b87e8658-2485-4254-8a4b-ccd6e512b1e0"]},
   "736d1574-844c-485f-aadb-d64190640dd7"
   {:compute-ref #uuid "5af9880f-8ce8-4e59-905d-6de7077ff11e",
    :exec-duration 48,
    :id "736d1574-844c-485f-aadb-d64190640dd7",
    :kind "code",
    :output-log-lines {},
    :runtime [:runtime "b87e8658-2485-4254-8a4b-ccd6e512b1e0"]},
   "75319a3b-f798-4f22-80be-3be49cc69e2d"
   {:id "75319a3b-f798-4f22-80be-3be49cc69e2d", :kind "code-listing"},
   "78896ccc-7cab-49c4-a0ad-9d9fe90ca5a5"
   {:compute-ref #uuid "e8fd935c-509a-4363-8bbd-d030d37d927d",
    :exec-duration 160,
    :id "78896ccc-7cab-49c4-a0ad-9d9fe90ca5a5",
    :kind "code",
    :output-log-lines {},
    :runtime [:runtime "b87e8658-2485-4254-8a4b-ccd6e512b1e0"]},
   "7b740297-1079-467f-acdf-499da02c6d20"
   {:id "7b740297-1079-467f-acdf-499da02c6d20", :kind "code-listing"},
   "85e976a8-d317-48fc-aa2b-558cc71b4e5d"
   {:compute-ref #uuid "6266646a-5edb-46a1-89b5-66123a10bcfc",
    :exec-duration 78,
    :id "85e976a8-d317-48fc-aa2b-558cc71b4e5d",
    :kind "code",
    :output-log-lines {},
    :runtime [:runtime "b87e8658-2485-4254-8a4b-ccd6e512b1e0"]},
   "873b47ea-ff23-4ba4-bee4-da4aac072efb"
   {:compute-ref #uuid "22687665-fa6a-4a78-b65f-dba96b5d8be7",
    :exec-duration 60,
    :id "873b47ea-ff23-4ba4-bee4-da4aac072efb",
    :kind "code",
    :output-log-lines {},
    :runtime [:runtime "b87e8658-2485-4254-8a4b-ccd6e512b1e0"]},
   "8871da49-dac6-469b-a273-e4a2a87ff848"
   {:compute-ref #uuid "5d05e819-3d94-483e-802d-35f035c9ac60",
    :exec-duration 366,
    :id "8871da49-dac6-469b-a273-e4a2a87ff848",
    :kind "code",
    :output-log-lines {},
    :runtime [:runtime "b87e8658-2485-4254-8a4b-ccd6e512b1e0"]},
   "8d7e7afe-edb4-4466-8810-23bdfd49ea91"
   {:compute-ref #uuid "0e986d13-f908-4d2e-ad3f-068bc440aed1",
    :exec-duration 141,
    :id "8d7e7afe-edb4-4466-8810-23bdfd49ea91",
    :kind "code",
    :output-log-lines {},
    :runtime [:runtime "b87e8658-2485-4254-8a4b-ccd6e512b1e0"]},
   "8ece8520-e980-4682-a44f-becb64840c7c"
   {:compute-ref #uuid "b0737457-f87d-4d51-a60e-6414367ec9ea",
    :exec-duration 60,
    :id "8ece8520-e980-4682-a44f-becb64840c7c",
    :kind "code",
    :output-log-lines {},
    :runtime [:runtime "b87e8658-2485-4254-8a4b-ccd6e512b1e0"]},
   "8f6facdd-9f71-4681-98ff-8a1613b70a8b"
   {:compute-ref #uuid "87c7d39f-fe72-4c81-b313-bb3f394da992",
    :exec-duration 49,
    :id "8f6facdd-9f71-4681-98ff-8a1613b70a8b",
    :kind "code",
    :output-log-lines {},
    :runtime [:runtime "b87e8658-2485-4254-8a4b-ccd6e512b1e0"]},
   "8fffb866-4d1d-4353-8748-f0e9d4c770c1"
   {:compute-ref #uuid "134e8966-a65d-4f43-a4a9-114b782a9198",
    :exec-duration 73,
    :id "8fffb866-4d1d-4353-8748-f0e9d4c770c1",
    :kind "code",
    :output-log-lines {},
    :runtime [:runtime "b87e8658-2485-4254-8a4b-ccd6e512b1e0"]},
   "95566a23-02bc-4969-93b8-31ed19e026d4"
   {:compute-ref #uuid "e0b6409d-0166-45ee-96b0-d66ff4d6e2d4",
    :exec-duration 90,
    :id "95566a23-02bc-4969-93b8-31ed19e026d4",
    :kind "code",
    :output-log-lines {},
    :runtime [:runtime "b87e8658-2485-4254-8a4b-ccd6e512b1e0"]},
   "a40bc6d0-77d1-4cce-9c3e-cc6fa7c24be7"
   {:id "a40bc6d0-77d1-4cce-9c3e-cc6fa7c24be7", :kind "code-listing"},
   "a46ab456-d97b-4434-a0b7-fc9f07c482e6"
   {:id "a46ab456-d97b-4434-a0b7-fc9f07c482e6", :kind "code-listing"},
   "ab2ea701-a11a-4ef9-a3c8-51227e81ef1c"
   {:id "ab2ea701-a11a-4ef9-a3c8-51227e81ef1c", :kind "code-listing"},
   "ab46712b-c4cd-4ffd-af74-baee71533aee"
   {:compute-ref #uuid "721c7a43-7d43-40c6-abe2-c1d06f825e03",
    :exec-duration 47,
    :id "ab46712b-c4cd-4ffd-af74-baee71533aee",
    :kind "code",
    :output-log-lines {},
    :runtime [:runtime "b87e8658-2485-4254-8a4b-ccd6e512b1e0"]},
   "acda780d-9072-4c3d-8a62-8365d40fcab0"
   {:compute-ref #uuid "7d66867b-9363-4cbc-a462-ec1e52a39e49",
    :exec-duration 87,
    :id "acda780d-9072-4c3d-8a62-8365d40fcab0",
    :kind "code",
    :output-log-lines {},
    :runtime [:runtime "b87e8658-2485-4254-8a4b-ccd6e512b1e0"]},
   "ad3acca5-10ed-4e6b-83cf-1f2ebce40f77"
   {:compute-ref #uuid "31269a78-00c2-4a88-8a85-83f3d5127344",
    :exec-duration 139,
    :id "ad3acca5-10ed-4e6b-83cf-1f2ebce40f77",
    :kind "code",
    :output-log-lines {},
    :runtime [:runtime "b87e8658-2485-4254-8a4b-ccd6e512b1e0"]},
   "b16b103d-ff19-4d85-a3e4-3013be6894a8"
   {:compute-ref #uuid "483e7bd0-1c62-452f-9226-b4902b9ffc10",
    :exec-duration 65,
    :id "b16b103d-ff19-4d85-a3e4-3013be6894a8",
    :kind "code",
    :output-log-lines {},
    :runtime [:runtime "b87e8658-2485-4254-8a4b-ccd6e512b1e0"]},
   "b2e2c9e1-4597-4b19-bb4f-e83a0a45e8bd"
   {:id "b2e2c9e1-4597-4b19-bb4f-e83a0a45e8bd", :kind "code-listing"},
   "b66fbac8-5ef0-4f69-beee-152c2b9f4ac7"
   {:compute-ref #uuid "b4f3cfb8-5e0b-4337-9e0e-38b79ec17c4e",
    :exec-duration 52,
    :id "b66fbac8-5ef0-4f69-beee-152c2b9f4ac7",
    :kind "code",
    :output-log-lines {},
    :runtime [:runtime "b87e8658-2485-4254-8a4b-ccd6e512b1e0"]},
   "b87e8658-2485-4254-8a4b-ccd6e512b1e0"
   {:environment
    [:environment
     {:article/nextjournal.id
      #uuid "5b45eb52-bad4-413d-9d7f-b2b573a25322",
      :change/nextjournal.id
      #uuid "5f045c36-90bd-428b-a26c-b59fa0a2e1db",
      :node/id "0ae15688-6f6a-40e2-a4fa-52d81371f733"}],
    :id "b87e8658-2485-4254-8a4b-ccd6e512b1e0",
    :kind "runtime",
    :language "clojure",
    :type :nextjournal,
    :runtime/mounts
    [{:src [:node "d2a8a640-910f-4566-94fb-6479ed92f7b9"],
      :dest "/deps.edn"}]},
   "b94a242d-969e-40df-8a4f-f37325738cd3"
   {:compute-ref #uuid "71343f1f-587d-44ea-9076-837ca4ea35b0",
    :exec-duration 51,
    :id "b94a242d-969e-40df-8a4f-f37325738cd3",
    :kind "code",
    :output-log-lines {},
    :runtime [:runtime "b87e8658-2485-4254-8a4b-ccd6e512b1e0"]},
   "ba036603-2fa1-4a4c-bc2b-d88017b97c7c"
   {:compute-ref #uuid "18e743b8-a3f4-4bb4-8f03-38055a5e97c4",
    :exec-duration 49,
    :id "ba036603-2fa1-4a4c-bc2b-d88017b97c7c",
    :kind "code",
    :output-log-lines {},
    :runtime [:runtime "b87e8658-2485-4254-8a4b-ccd6e512b1e0"]},
   "bb9a6631-d31a-4585-a1ac-4fbd3dd16659"
   {:id "bb9a6631-d31a-4585-a1ac-4fbd3dd16659", :kind "code-listing"},
   "c1bab13d-ff15-43a5-bbc3-b9dd242d10e6"
   {:id "c1bab13d-ff15-43a5-bbc3-b9dd242d10e6", :kind "code-listing"},
   "c2bce90a-ce64-42a2-9e70-bdd65bc53308"
   {:compute-ref #uuid "55de08c0-ab7e-42d1-86b7-5b6a56e04df3",
    :exec-duration 87,
    :id "c2bce90a-ce64-42a2-9e70-bdd65bc53308",
    :kind "code",
    :output-log-lines {},
    :runtime [:runtime "b87e8658-2485-4254-8a4b-ccd6e512b1e0"]},
   "c6b02497-6495-477a-ac9b-c3a1438ede77"
   {:compute-ref #uuid "b13a9253-3f65-4cbe-af9a-8ffa863642e4",
    :exec-duration 79,
    :id "c6b02497-6495-477a-ac9b-c3a1438ede77",
    :kind "code",
    :output-log-lines {},
    :runtime [:runtime "b87e8658-2485-4254-8a4b-ccd6e512b1e0"]},
   "cc8883de-ad05-4b62-897b-ffe5b754b061"
   {:id "cc8883de-ad05-4b62-897b-ffe5b754b061", :kind "code-listing"},
   "d092a2fb-9540-4f6b-b747-77ff36fad5ac"
   {:compute-ref #uuid "3c783e7b-b374-4138-b05a-f8ec5c6ba726",
    :exec-duration 87,
    :id "d092a2fb-9540-4f6b-b747-77ff36fad5ac",
    :kind "code",
    :output-log-lines {},
    :runtime [:runtime "b87e8658-2485-4254-8a4b-ccd6e512b1e0"]},
   "d2a8a640-910f-4566-94fb-6479ed92f7b9"
   {:id "d2a8a640-910f-4566-94fb-6479ed92f7b9",
    :kind "code-listing",
    :name "deps.edn"},
   "d3b699f1-702e-4aad-8657-4c419e14e88d"
   {:compute-ref #uuid "ccc5c3c4-0eb2-4042-a08a-0dc96904e440",
    :exec-duration 13560,
    :id "d3b699f1-702e-4aad-8657-4c419e14e88d",
    :kind "code",
    :output-log-lines {},
    :runtime [:runtime "b87e8658-2485-4254-8a4b-ccd6e512b1e0"]},
   "d8dcf3db-a888-46ac-ad35-942c7374e223"
   {:compute-ref #uuid "4e564ec9-4645-44ae-8f82-9a0eb0892a22",
    :exec-duration 46,
    :id "d8dcf3db-a888-46ac-ad35-942c7374e223",
    :kind "code",
    :output-log-lines {},
    :runtime [:runtime "b87e8658-2485-4254-8a4b-ccd6e512b1e0"]},
   "da8488cb-de60-4c7a-9706-849e775102a3"
   {:compute-ref #uuid "e7ae67c4-a9dd-41ca-aee9-9987d426d089",
    :exec-duration 76,
    :id "da8488cb-de60-4c7a-9706-849e775102a3",
    :kind "code",
    :output-log-lines {},
    :runtime [:runtime "b87e8658-2485-4254-8a4b-ccd6e512b1e0"]},
   "db46f3e2-70a3-4768-bef5-2a48d246be9e"
   {:compute-ref #uuid "86dfeb3a-198d-4410-ae3b-6c7502ab4e56",
    :exec-duration 75,
    :id "db46f3e2-70a3-4768-bef5-2a48d246be9e",
    :kind "code",
    :output-log-lines {},
    :runtime [:runtime "b87e8658-2485-4254-8a4b-ccd6e512b1e0"]},
   "db7c287c-4e6b-47ae-8716-2254b37981a1"
   {:compute-ref #uuid "9427014d-fcb0-4531-aba0-2ffc5be5c64f",
    :exec-duration 181,
    :id "db7c287c-4e6b-47ae-8716-2254b37981a1",
    :kind "code",
    :output-log-lines {},
    :runtime [:runtime "b87e8658-2485-4254-8a4b-ccd6e512b1e0"]},
   "dbe5cfa2-9a01-4c1c-b0ec-eda50ffec27a"
   {:id "dbe5cfa2-9a01-4c1c-b0ec-eda50ffec27a", :kind "code-listing"},
   "e2682518-935d-4a09-af39-210cf8286d33"
   {:compute-ref #uuid "bb6a28e8-d7d6-43d9-bcee-9f791d7b274b",
    :exec-duration 385,
    :id "e2682518-935d-4a09-af39-210cf8286d33",
    :kind "code",
    :output-log-lines {},
    :runtime [:runtime "b87e8658-2485-4254-8a4b-ccd6e512b1e0"]},
   "ea0ed77b-5507-4c8f-93b2-e06fc7a213fd"
   {:compute-ref #uuid "f2cc59dd-55cc-4296-80ef-e115d4222bbb",
    :exec-duration 49,
    :id "ea0ed77b-5507-4c8f-93b2-e06fc7a213fd",
    :kind "code",
    :output-log-lines {},
    :runtime [:runtime "b87e8658-2485-4254-8a4b-ccd6e512b1e0"]},
   "ec1250a9-3a5f-4ef8-a19f-ab2ac32f281b"
   {:compute-ref #uuid "47806a9a-4ffd-4c76-960b-717562d099e9",
    :exec-duration 6291,
    :id "ec1250a9-3a5f-4ef8-a19f-ab2ac32f281b",
    :kind "code",
    :output-log-lines {},
    :runtime [:runtime "b87e8658-2485-4254-8a4b-ccd6e512b1e0"]},
   "ec915b4e-615e-4ee8-a110-f701fe42b677"
   {:id "ec915b4e-615e-4ee8-a110-f701fe42b677", :kind "code-listing"},
   "eea2322c-b011-4d79-bedd-1bdb35deee34"
   {:compute-ref #uuid "45622eb1-4d64-4b5f-a13e-a29546e03785",
    :exec-duration 136,
    :id "eea2322c-b011-4d79-bedd-1bdb35deee34",
    :kind "code",
    :output-log-lines {},
    :runtime [:runtime "b87e8658-2485-4254-8a4b-ccd6e512b1e0"]},
   "efe159a6-33df-481f-ad9c-751f4afebf48"
   {:compute-ref #uuid "681d87e0-e87d-4f8d-88b3-bbe9693065ec",
    :exec-duration 43,
    :id "efe159a6-33df-481f-ad9c-751f4afebf48",
    :kind "code",
    :output-log-lines {},
    :runtime [:runtime "b87e8658-2485-4254-8a4b-ccd6e512b1e0"]},
   "f7fcf3ff-359e-4115-abda-7324679d4da0"
   {:compute-ref #uuid "a84de897-8778-41dc-b8f1-4f11eea0e43e",
    :exec-duration 78,
    :id "f7fcf3ff-359e-4115-abda-7324679d4da0",
    :kind "code",
    :output-log-lines {},
    :runtime [:runtime "b87e8658-2485-4254-8a4b-ccd6e512b1e0"]},
   "f91a07ba-5871-42e6-8da6-763d5053b225"
   {:compute-ref #uuid "4b765011-2a6b-4766-b05a-0d091f306a8d",
    :exec-duration 52,
    :id "f91a07ba-5871-42e6-8da6-763d5053b225",
    :kind "code",
    :output-log-lines {},
    :runtime [:runtime "b87e8658-2485-4254-8a4b-ccd6e512b1e0"]},
   "f9e67c81-c05b-4278-976d-24726430a8b0"
   {:compute-ref #uuid "7043646f-1f55-4571-b315-cfe386bf0e26",
    :exec-duration 89,
    :id "f9e67c81-c05b-4278-976d-24726430a8b0",
    :kind "code",
    :output-log-lines {},
    :runtime [:runtime "b87e8658-2485-4254-8a4b-ccd6e512b1e0"]},
   "fceef220-fe9c-4da0-80c5-1ab5b6f6a084"
   {:id "fceef220-fe9c-4da0-80c5-1ab5b6f6a084", :kind "code-listing"},
   "fe781e94-0e50-4dd6-90c8-0df43dfa9ec1"
   {:compute-ref #uuid "a3a7cf6b-a706-4ead-95d9-12c08352c5b5",
    :exec-duration 76,
    :id "fe781e94-0e50-4dd6-90c8-0df43dfa9ec1",
    :kind "code",
    :output-log-lines {},
    :runtime [:runtime "b87e8658-2485-4254-8a4b-ccd6e512b1e0"]},
   "ff921ee6-94b1-4e88-843b-ecdb0fb442f9"
   {:compute-ref #uuid "808b49a2-909d-4004-81d4-fa304ce032d8",
    :exec-duration 51,
    :id "ff921ee6-94b1-4e88-843b-ecdb0fb442f9",
    :kind "code",
    :output-log-lines {},
    :runtime [:runtime "b87e8658-2485-4254-8a4b-ccd6e512b1e0"]},
   "ffb88b44-0bc5-4d58-beea-87b1c7f2b879"
   {:id "ffb88b44-0bc5-4d58-beea-87b1c7f2b879", :kind "file"}},
  :nextjournal/id #uuid "031b8f79-51ec-425e-b6ab-da02b443e162",
  :article/change
  {:nextjournal/id #uuid "6124e557-2d8f-4230-9f30-f24930be8870"}}}

```
</details>
