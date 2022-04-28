# Comunica SPARQL Mapping Query Engine

[![npm version](https://badge.fury.io/js/%40comunica%2Fquery-sparql-mapping.svg)](https://www.npmjs.com/package/@comunica/query-sparql-mapping)
[![Docker Pulls](https://img.shields.io/docker/pulls/comunica/query-sparql-mapping.svg)](https://hub.docker.com/r/comunica/query-sparql-mapping/)

Comunica SPARQL Mapping is a SPARQL query engine for JavaScript that follows links to detect new sources.

This module is part of the [Comunica framework](https://comunica.dev/).

## Install

```bash
$ yarn add @comunica/query-sparql-mapping
```

or

```bash
$ npm install -g @comunica/query-sparql-mapping
```

## Install a prerelease

Since this package is still in testing phase, you may want to install a prerelease of this package, which you can do by appending `@next` to the package name during installation.

```bash
$ yarn add @comunica/query-sparql-mapping@next
```

or

```bash
$ npm install -g @comunica/query-sparql-mapping@next
```

## Usage

Show 100 triples from http://fragments.dbpedia.org/2015-10/en:

```bash
$ comunica-sparql-mapping https://www.rubensworks.net/ \
  "SELECT DISTINCT * WHERE {
       <https://www.rubensworks.net/#me> <http://xmlns.com/foaf/0.1/knows> ?p.
       <https://ruben.verborgh.org/profile/#me> <http://xmlns.com/foaf/0.1/knows> ?p.
       ?p <http://xmlns.com/foaf/0.1/name> ?name.
   }" --lenient
```

Show the help with all options:

```bash
$ comunica-sparql-mapping --help
```

Just like [Comunica SPARQL](https://github.com/comunica/comunica/tree/master/packages/query-sparql),
a [dynamic variant](https://github.com/comunica/comunica/tree/master/packages/query-sparql#usage-from-the-command-line) (`comunica-dynamic-sparql-mapping`) also exists.

_[**Read more** about querying from the command line](https://comunica.dev/docs/query/getting_started/query_cli/)._

### Usage within application

This engine can be used in JavaScript/TypeScript applications as follows:

```javascript
const QueryEngine = require('@comunica/query-sparql-mapping').QueryEngine;
const myEngine = new QueryEngine();

const bindingsStream = await myEngine.queryBindings(`
  SELECT DISTINCT * WHERE {
      <https://www.rubensworks.net/#me> <http://xmlns.com/foaf/0.1/knows> ?p.
      <https://ruben.verborgh.org/profile/#me> <http://xmlns.com/foaf/0.1/knows> ?p.
      ?p <http://xmlns.com/foaf/0.1/name> ?name.
  }`, {
    sources: ['https://www.rubensworks.net/'],
    lenient: true,
});

// Consume results as a stream (best performance)
bindingsStream.on('data', (binding) => {
    console.log(binding.toString()); // Quick way to print bindings for testing

    console.log(binding.has('s')); // Will be true

    // Obtaining values
    console.log(binding.get('s').value);
    console.log(binding.get('s').termType);
    console.log(binding.get('p').value);
    console.log(binding.get('o').value);
});
bindingsStream.on('end', () => {
    // The data-listener will not be called anymore once we get here.
});
bindingsStream.on('error', (error) => {
    console.error(error);
});

// Consume results as an array (easier)
const bindings = await bindingsStream.toArray();
console.log(bindings[0].get('s').value);
console.log(bindings[0].get('s').termType);
```

_[**Read more** about querying an application](https://comunica.dev/docs/query/getting_started/query_app/)._

### Usage as a SPARQL endpoint

Start a webservice exposing https://www.rubensworks.net/ via the SPARQL protocol, i.e., a _SPARQL endpoint_.

```bash
$ comunica-sparql-mapping-http https://www.rubensworks.net/
```

Show the help with all options:

```bash
$ comunica-sparql-mapping-http --help
```

The SPARQL endpoint can only be started dynamically.
An alternative config file can be passed via the `COMUNICA_CONFIG` environment variable.

Use `bin/http.js` when running in the Comunica monorepo development environment.

_[**Read more** about setting up a SPARQL endpoint](https://comunica.dev/docs/query/getting_started/setup_endpoint/)._
