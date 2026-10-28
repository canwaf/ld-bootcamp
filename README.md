# Linked Data Bootcamp

A self-paced course that teaches Linked Data the native way: starting from something every analyst already does, joining tables on a key, and following that idea until it becomes URIs, authority, RDF and SPARQL. The course is published as a small website written in plain HTML and CSS, and every page is itself a piece of Linked Data. The course describes its own modules, activities and quiz questions in RDFa, so the site a learner reads is also the first graph they get to inspect.

The course is published at <https://canwaf.github.io/ld-bootcamp/>.

## Syllabus

The course runs as five modules, each building directly on the last. Each module has a short reading, worked examples, and a marked quiz.

One worked example runs through all five: a water quality sampling point, the samples taken there, the observations made on those samples, the concepts that classify each of them, and the catalogue that publishes them. It follows the patterns used to publish environmental data on `environment.data.gov.uk`, so what learners practise on is the shape of real published data.

### Module 1: From keys to URIs

Learners start with three small tables, sampling points, samples and observations, joined on code columns. They then meet the cases where those joins quietly go wrong: the same code meaning different things in different tables, a code that only makes sense alongside a lookup file, a classification held as whatever label someone typed ("River", "river", "Rivers"). The module reframes a URI as a key whose scope is the whole web, carrying its own context with it, so the sampling point notation `AN-CORBY` becomes `https://environment.data.gov.uk/water-quality/sampling-point/AN-CORBY` and any dataset using that URI joins with any other dataset using it. The same move applies to classifications: a sampling point type or a determinand becomes the URI of a concept, so a filter matches the concept exactly and its label is free to change. The module closes on dereferencing. People care about resources, and a URI typed into a browser should tell them about the thing it names. By the end, learners can explain why a local key fails once it leaves its table, rewrite both identifier and classification columns as URIs, and follow a URI to find out what it identifies.

### Module 2: Authority, minting and persistence

If a URI is a global key, someone has to be responsible for it. This module covers how domain ownership gives an organisation authority over a namespace, and why `environment.data.gov.uk` is the right home for an Environment Agency sampling point and the wrong home for a local authority's air quality sensor. Identifiers come from the people who own the data: `AN-CORBY` is the provider's own notation for the sampling point, so the URI is built from it, and database row numbers stay inside the database. Good URIs name the thing directly in a path a person can read, `/water-quality/sampling-point/AN-CORBY`, so someone can navigate the service by editing the URL. Learners practise minting URIs under a namespace they control, reusing the URIs of published concept schemes for classifications, and writing prefixes so that long namespaces become readable shorthand (`sampling-point:AN-CORBY`).

Naming the parts of a thing is taught as one idea written in two styles. Nested paths name parts with exactly one parent, following the cardinality promise in the Water Quality pattern: `/sampling-point/AN-CORBY/sample/2119744/observation/0111`, where every sample belongs to one sampling point and a sample has one observation per determinand. Hash URIs name parts delivered together with their parent, such as a sample's `#sampling`, and this course's own activities.

The persistence topic takes its title from "Cool URIs don't change", and its argument is about discipline: change an identifier only when the thing it identifies has changed. ISO 3166 is the model. `GB` has identified the United Kingdom since the standard was first published, and a code that every border system, shipping form and dataset depends on stays put because changing it would break all of them at once. URIs deserve the same treatment. A renamed team, a new database or a rebuilt website leaves the sampling point exactly where it was, so its URI stays the same too. When a sampling point's record is replaced with a fresh one, its URI is carried over, and everything that joined on it keeps working. Learners practise designing patterns built only from the stable facts of the thing itself, such as its type and its notation.

### Module 3: Triples and vocabularies

With identifiers settled, learners assemble statements from them, using the real graph around `AN-CORBY`. Triples are introduced as three-column tables of subject, predicate and object before any syntax, and a row from Module 1's Samples table becomes three triples about two things, the sample and the act of sampling it. Triples that share a URI join into a graph, drawn for the observation of ammoniacal nitrogen in sample `2119744`, which makes the Module 1 join a matter of following arrows. Literals carry datatypes (`xsd:dateTime`, `xsd:date`, `xsd:decimal`) and language tags, with notations kept free of language tags. The result `<0.132` mg/l is the worked example of a value with structure: a `qudt:QuantityValue` with an upper bound and a unit, held in a blank node reached through its observation.

The module then introduces the vocabularies the DSP Application Profile uses, each answering one question: SKOS for concepts, notations and labels; SOSA/SSN for the chain from sampling point to sampling, sample and observation; QUDT for quantities and units; GeoSPARQL for geometry and `geo:sfWithin`; DCAT for catalogues and datasets, including `dcat:themeTaxonomy`; Dublin Core for publication terms such as title, publisher and licence; and RDFS and OWL for defining terms of your own, as Defra's domain model does under `defra-core:`. It closes on classes and classifications: `AN-CORBY` is typed as both `sosa:FeatureOfInterest` and `geo:Feature`, and `defra-core:hasClassification` adds a finer kind as a SKOS concept without a new class, so the River Irwell is a water body classified as a river, and `AN-CORBY` is classified by its sampling point type. The predicate's range, `skos:Concept`, is what suits it to that job.

### Module 4: Serialisation and content negotiation

The graph stays fixed while the syntax changes. Learners start with Turtle, the format they read and write most, built up step by step from a single full triple: `@prefix` and `@base`, `;` and `,` for repeated subjects and predicates, `a`, typed and language-tagged literals, and `[ ]` for the result's blank node, ending with the whole `AN-CORBY` graph as one document. N-Triples follows as Turtle with every shorthand removed, one full triple per line for programs and bulk loading. RDF/XML is shown element by element against Turtle, then converted back to Turtle with a one-line `rdflib` command, so learners see that the same graph written differently means the same thing. JSON-LD is introduced as the native format of the RESTful Linked Data APIs: ordinary JSON with short keys and nesting, plus an `@context` that maps each key to its URI and sets datatypes, so a JSON developer can ignore the `@` keys and still have usable data. RDFa is shown through this course's own page source. Every serialisation on the page is checked with an RDF parser to hold the same ten triples.

The second half covers content negotiation against the live Water Quality service with `curl`: one URI serving JSON-LD or GeoJSON for a sampling point, CSV and JSON Lines for a list of observations, the `content-type`, `vary` and `link` response headers, and the error that lists the formats an endpoint offers. It closes on the file-extension form, where a suffix such as `.json` or `.ttl` gives each format a URI of its own, set against the service's choice of one URI per thing with the `Accept` header choosing the format.

### Module 5: Querying with SPARQL

The final module teaches learners to ask questions of a graph. It starts with `SELECT` and a single triple pattern in the `WHERE` clause, then builds up to the join from observation to sample to sampling point across patterns that share a variable, which brings the course back to where it started: a shared variable is a join on a URI. From there it introduces filtering observations by concept URI, discovering through `dcat:themeTaxonomy` which concept schemes classify a dataset, `FILTER`, creating new values with `BIND`, string handling with `STR`, `SUBSTR`, `STRSTARTS` and `CONCAT` (for example, extracting the notation `AN-CORBY` from the end of its URI), combining alternative patterns with `UNION`, and keeping rows where some data is missing with `OPTIONAL`. Exercises present a small graph in Turtle alongside a query and ask the learner to predict the result rows, which trains careful reading of what each clause does. Learners who want to run queries can load the same Turtle into any SPARQL endpoint, such as Apache Jena Fuseki.

### Later modules

GeoSPARQL follows as the next extension, building on Module 5: sampling point locations as WKT geometries with explicit CRS URIs, British National Grid (EPSG:27700) and its axis order, spatial relationships between sampling points and the catchments they sit in, and requesting geometries in a chosen CRS with an `Accept-Crs` header. CSV on the Web and SHACL provide material for further modules after that.

## How the site is built

The site is hand-written HTML and CSS following the GOV.UK Design System patterns: its layout grid, type scale, colours and focus states, and components such as the start button, phase banner, breadcrumbs, details, tables, summary lists, radios, tags and panels. The course carries its own header and name. Pages use semantic HTML for structure and modern CSS for everything interactive, so the source of every page is readable by a learner, which matters because reading the source is part of the course.

Quizzes are marked in the browser with CSS alone. Each question is a group of radio inputs; selecting an answer reveals its feedback through `:checked` and `:has()` selectors, and a CSS animation marks the answer right or wrong. Running scores use CSS counters that increment on each correct answer checked. Hints and model answers sit inside `<details>` and `<summary>` elements so learners reveal them deliberately. Because the correct answer is identified in the markup, the quizzes are designed for self-assessment.

Each page's URL is a resource URI. A module page is the document describing that module, and each activity or question on it is identified with a hash URI on the same page (for example `/modules/keys-to-uris/#q3`). Pages describe the course, its modules and its activities in RDFa using schema.org's learning vocabulary (`Course`, `LearningResource`, `Quiz`, `Question`, `Answer`, with `hasPart`, `isPartOf`, `teaches` and `assesses`) and Dublin Core terms for titles, modification dates and the licence. Any RDFa parser pointed at the site recovers the full course structure as a graph, which Modules 4 and 5 then use as practice data.

## Repository layout

| Path | Contents |
| --- | --- |
| `index.html` | The course landing page, which describes the course itself in RDFa |
| `assets/bootcamp.css` | The shared stylesheet, including the CSS that marks quizzes |
| `modules/keys-to-uris/` | Module 1: From keys to URIs |
| `modules/authority-and-persistence/` | Module 2: Authority, minting and persistence |
| `modules/triples-and-vocabularies/` | Module 3: Triples and vocabularies |
| `modules/serialisation-and-content-negotiation/` | Module 4: Serialisation and content negotiation |
| `modules/sparql/` | Module 5: Querying with SPARQL |
| `404.html` | The page GitHub Pages shows for an address that doesn't exist |
| `.nojekyll` | Tells GitHub Pages to publish the files exactly as they are |
| `LICENSE` | The CC0 1.0 Universal legal code |

Each module is one directory with an `index.html`, so directory paths map directly onto the course's resource URIs. Links between pages are relative, so the site works from any web server or base path; `404.html` uses absolute paths under `/ld-bootcamp/`, because GitHub Pages serves it at whatever address was requested. To run the site locally:

```
python3 -m http.server
```

then open `http://localhost:8000/`.

## Publishing

The site is published with GitHub Pages from the `main` branch of `canwaf/ld-bootcamp`, using the repository root as the source. GitHub Pages serves a project site at a path named after its repository, which gives the course the address `https://canwaf.github.io/ld-bootcamp/`.

That address is also the base of every resource URI in the course’s RDFa, so it follows Module 2’s advice: choose it once and keep it.

## How this course was written

This course was written with Claude, the AI model made by Anthropic, working to the course author’s direction and review. The Water Quality examples were checked against the live Environment Agency service and the [DSP Application Profile](https://agrimetrics.github.io/application-profile/), and every serialisation in Module 4 was checked with an RDF parser.

## Licence

The course is dedicated to the public domain under [CC0 1.0 Universal](https://creativecommons.org/publicdomain/zero/1.0/). You can copy, modify, adapt and republish it, including for commercial use, without asking permission. The full legal code is in [`LICENSE`](LICENSE).

Water Quality data quoted in the course, such as the sampling point `AN-CORBY` and its samples and observations, is published by the Environment Agency under the [Open Government Licence v3.0](https://www.nationalarchives.gov.uk/doc/open-government-licence/version/3/). Contains public sector information licensed under the Open Government Licence v3.0.
