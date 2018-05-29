# Mu Query Rewriter Sandbox

This repository provides a very simple UI for writing and testing Query Rewriter plugins for use with the [Mu Query Rewriter](https://github.com/big-data-europe/mu-query-rewriter).

The [graph-acl-basics](https://github.com/big-data-europe/graph-acl-basics/) repository provides a full working example for experimentation.

## Sample Users

Users are fetched from the database with this query, which is run on the mu-query-rewriter backend. 

```
PREFIX muauth: <http://mu.semte.ch/vocabularies/authorization/>
PREFIX dct: <http://purl.org/dc/terms/>
SELECT DISTINCT ?user ?role ?name
WHERE {
 { ?user muauth:has-role ?role }
 UNION
 { ?user muauth:has-role/dct:title ?role }

 FILTER (isLiteral(?role))
 
 { ?user dct:title ?name }
 UNION
 { ?user foaf:name ?name }
}
```

To customize it, replace the loading function at the bottom of assets/sandbox.js.

## Example docker-compose file

```
version: "2"
services:
  db:
    image: tenforce/virtuoso:1.0.0-virtuoso7.2.4
    environment:
      SPARQL_UPDATE: "true"
      DEFAULT_GRAPH: "http://mu.semte.ch/application"
    ports:
      - "8890:8890"
    volumes:
      - ./data/db:/data
  rewriter:
    image: nathanielrb/mu-graph-rewriter
    links:
      - db:database
    environment:
      DEBUG_LOGGING: "true"
      PLUGIN: "authorization"
    volumes:
      - ./config/rewriter:/config
    ports:
      - "4027:8890"
  my-service:
    image: my/service
    links:
      - rewriter:database
  sandbox:
    image: semtech/ember-proxy-service
    ports:
      - "9000:80"
    links:
      - identifier:backend
    volumes:
      - /path/to/mu-query-rewriter-sandbox:/app
```
