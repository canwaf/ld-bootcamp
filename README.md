# Linked Data Bootcamp

A self-paced course that teaches Linked Data the native way: starting from something every analyst already does, joining tables on a key, and following that idea until it becomes URIs, authority, RDF and SPARQL. The course is published as a small website written in plain HTML and CSS, and every page is itself a piece of Linked Data. The course describes its own modules, activities and quiz questions in RDFa, so the site a learner reads is also the first graph they get to inspect.

The course is published at <https://canwaf.github.io/ld-bootcamp/>.

## Syllabus

The course runs as nine modules, each building directly on the last. Each module has a short reading, worked examples, and a marked quiz.

Two worked examples run through the course and meet at one URI. The first is a water quality sampling point, the samples taken there, the observations made on those samples, the concepts that classify each of them, and the catalogue that publishes them. The second is a water discharge permit, the discharge point it covers and the holder it was issued to, where the discharge point's effluent is sampled at the same sampling point, `AN-CORBY`, Corby sewage treatment works' final effluent. Both follow the patterns used to publish environmental data on `environment.data.gov.uk` and the Defra domain model, so what learners practise on is the shape of real published data.

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

The graph stays fixed while the syntax changes. Learners start with Turtle, the format they read and write most, built up step by step from a single full triple: `@prefix` and `@base`, `;` and `,` for repeated subjects and predicates, `a`, typed and language-tagged literals, and `[ ]` for the result's blank node, ending with the whole `AN-CORBY` graph as one document. N-Triples follows as Turtle with every shorthand removed, one full triple per line for programs and bulk loading. RDF/XML is shown element by element against Turtle, then converted back to Turtle with `rdfpipe`, the one-line converter that comes with `rdflib`, so learners see that the same graph written differently means the same thing. JSON-LD is introduced as the native format of the RESTful Linked Data APIs: ordinary JSON with short keys and nesting, plus an `@context` that maps each key to its URI and sets datatypes, so a JSON developer can ignore the `@` keys and still have usable data. RDFa is shown through this course's own page source. Every serialisation on the page is checked with an RDF parser to hold the same ten triples.

The second half covers content negotiation against the live Water Quality service with `curl`: one URI serving JSON-LD or GeoJSON for a sampling point, CSV and JSON Lines for a list of observations, the `content-type`, `vary` and `link` response headers, and the error that lists the formats an endpoint offers. It closes on the file-extension form, where a suffix such as `.json` or `.ttl` gives each format a URI of its own, set against the service's choice of one URI per thing with the `Accept` header choosing the format.

### Module 5: Reading a vocabulary

Module 3 showed which vocabulary answers which question. This module teaches learners to read one on their own, using the Defra regulation and water ontologies that describe permits. A term is a URI, so reading it starts by following it: learners dereference terms from SKOS, SOSA and Dublin Core, then read each one twice, on its publisher's documentation page and as triples, matching every field on the page to its predicate. The Defra terms have no documentation page, so learners read them straight from the published Turtle files, block by block, the same way. For each term they find its label, its definition, `rdfs:isDefinedBy`, and where it sits in the class hierarchy: `defra-water:WaterDischargePermit` is a `defra-reg:Permit`, which is a `dcat:Resource`; `defra-reg:DischargePoint` is a `defra-core:Site`, which is a `geo:Feature`. Learners sort terms into classes, properties and SKOS concepts, which keeps the schema a vocabulary defines apart from the controlled values it points at.

The second half covers what a property's domain and range do to data that uses it. They are rules for drawing conclusions: a triple using `defra-water:monitoredAt` makes its subject a `defra-reg:DischargePoint` and its object a `sosa:FeatureOfInterest`, whatever the author meant. Learners work out by hand what a handful of triples make true, including `defra-core:hasClassification`, whose range of `skos:Concept` is the intended conclusion, and `defra-reg:regulatedProperty`, whose domain is a union of two classes. By the end, learners can look up any term, say what kind of term it is, and predict the types their own data will acquire from the properties it uses.

### Module 6: Making instances

With a vocabulary read, learners describe one real thing with it by hand. The thing is a water discharge permit, the discharge point it covers, and its holder: the permit is a `defra-water:WaterDischargePermit`, linked to its discharge point with `defra-reg:permitSite` and to its holder with `defra-reg:permitHolder`; the discharge point is a `defra-reg:DischargePoint` linked to `AN-CORBY` with `defra-water:monitoredAt`. The permit number becomes both the notation in the permit's URI and a `defra-core:Identifier` with its scheme and issuer, and the permit's type is a SKOS concept through `defra-core:hasClassification`. Each step applies Module 2: mint the URI where you are responsible for the thing, and reuse the URI where someone else is. Learners mint the permit, discharge point and holder under `https://data.example/`, a namespace reserved for examples, using readable patterns built from each thing's own key, and reuse the Environment Agency's URI for `AN-CORBY`.

Learners write the result as Turtle and check it parses with `rdfpipe`. They then run the `owlrl` reasoner over it together with the three Defra ontologies, and compare the types they stated with the types that follow from the vocabulary, before converting the same graph to JSON-LD with `rdfpipe`. The file they finish with is the target for Modules 7 and 8: the graph one row of a permit table should become.

### Module 7: Describing tables with CSV on the Web

Most data arrives as tables, and CSV on the Web (CSVW) describes a table well enough for a machine to turn every row into the graph learners wrote by hand. Learners start with two CSV files, permits and discharge points, joined on the discharge point's code as in Module 1. They write a `csv-metadata.json` for them, one property at a time: `url` and `tableSchema` to say which file is described, `columns` with `titles` and `datatype`, and `aboutUrl` to mint each row's subject from its key under `https://data.example/`. `propertyUrl` chooses the predicate, `valueUrl` turns a key column into a link, so the discharge point code becomes the discharge point's URI and the sampling point notation becomes `https://environment.data.gov.uk/water-quality/sampling-point/AN-CORBY`, and virtual columns add statements every row shares, such as each permit's type.

The metadata file is JSON with an `@context`, so learners recognise it from Module 4's JSON-LD. Foreign keys are covered as what they are in CSVW: a check that every code in one table exists in the other, with the links themselves made by `valueUrl`. By the end, learners can read a CSV file and write metadata that maps each column to a term from Module 5.

### Module 8: From CSV to RDF

Learners run the conversion with Swirrl's `csv2rdf`, packaged as a container image and run with Docker or Podman on any operating system, pinned to a released version. They convert in standard mode first and find the extra statements CSVW adds about the table, each row and the conversion itself, then convert in minimal mode, which produces only the statements described by the metadata. Minimal mode is what the course uses from then on.

The check that closes the module returns to Module 4. Learners write their Module 6 target and the converted output as N-Triples, sort both, and compare them line by line, so any difference points at a column whose metadata needs fixing. Exercises break a working metadata file in small ways, a missing datatype, a wrong template, a typo in a prefix, and ask learners to predict the effect before running the conversion. By the end, learners can turn a table into a graph, check it against the graph they meant, and explain each difference.

### Module 9: Querying with SPARQL

The final module teaches learners to ask questions of a graph, and they now have two graphs of their own to ask: the sampling point graph from Module 4 and the permit graph from Module 8. It starts with `SELECT` and a single triple pattern in the `WHERE` clause, then builds up to the join from observation to sample to sampling point across patterns that share a variable, which brings the course back to where it started: a shared variable is a join on a URI. The two graphs meet at `AN-CORBY`, so the module's capstone query follows a permit to its discharge point, to the sampling point that monitors it, and on to the observations made there. From there it introduces filtering observations by concept URI, discovering through `dcat:themeTaxonomy` which concept schemes classify a dataset, `FILTER`, creating new values with `BIND`, string handling with `STR`, `SUBSTR`, `STRSTARTS` and `CONCAT` (for example, extracting the notation `AN-CORBY` from the end of its URI), combining alternative patterns with `UNION`, and keeping rows where some data is missing with `OPTIONAL`. Exercises present a small graph in Turtle alongside a query and ask the learner to predict the result rows, which trains careful reading of what each clause does. Learners run queries locally over the files they have made, with `rdflib` or any SPARQL engine that reads Turtle.

### Later modules

GeoSPARQL follows as the next extension, building on Module 9: sampling point and discharge point locations as WKT geometries with explicit CRS URIs, British National Grid (EPSG:27700) and its axis order, spatial relationships between points and the catchments they sit in, and requesting geometries in a chosen CRS with an `Accept-Crs` header. SHACL follows, describing the shape a permit graph must have and validating Module 8's output against it.

## How the site is built

The site is HTML and CSS following the GOV.UK Design System patterns: its layout grid, type scale, colours and focus states, and components such as the start button, phase banner, breadcrumbs, details, tables, summary lists, radios, tags and panels. The course carries its own header and name. Pages use semantic HTML for structure and modern CSS for everything interactive, so the source of every page is readable by a learner, which matters because reading the source is part of the course.

Quizzes are marked in the browser with CSS alone. Each question is a group of radio inputs; selecting an answer reveals its feedback through `:checked` and `:has()` selectors, and a CSS animation marks the answer right or wrong. Running scores use CSS counters that increment on each correct answer checked. Hints and model answers sit inside `<details>` and `<summary>` elements so learners reveal them deliberately. Because the correct answer is identified in the markup, the quizzes are designed for self-assessment.

Each page's URL is a resource URI. A module page is the document describing that module, and each activity or question on it is identified with a hash URI on the same page (for example `/modules/keys-to-uris/#q3`). Pages describe the course, its modules and its activities in RDFa using schema.org's learning vocabulary (`Course`, `LearningResource`, `Quiz`, `Question`, `Answer`, with `hasPart`, `isPartOf`, `teaches` and `assesses`) and Dublin Core terms for titles, modification dates and the licence. Any RDFa parser pointed at the site recovers the full course structure as a graph, which Modules 4 and 9 then use as practice data.

## Repository layout

| Path | Contents |
| --- | --- |
| `index.html` | The course landing page, which describes the course itself in RDFa |
| `assets/bootcamp.css` | The shared stylesheet, including the CSS that marks quizzes |
| `modules/keys-to-uris/` | Module 1: From keys to URIs |
| `modules/authority-and-persistence/` | Module 2: Authority, minting and persistence |
| `modules/triples-and-vocabularies/` | Module 3: Triples and vocabularies |
| `modules/serialisation-and-content-negotiation/` | Module 4: Serialisation and content negotiation |
| `modules/reading-a-vocabulary/` | Module 5: Reading a vocabulary |
| `modules/making-instances/` | Module 6: Making instances |
| `modules/csv-on-the-web/` | Module 7: Describing tables with CSV on the Web |
| `modules/csv-to-rdf/` | Module 8: From CSV to RDF |
| `modules/sparql/` | Module 9: Querying with SPARQL |
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
