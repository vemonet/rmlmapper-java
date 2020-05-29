# RMLMapper

The RMLMapper execute RML rules to generate Linked Data.
It is a Java library, which is available via the command line ([API docs online](https://rmlio.github.io/rmlmapper-java/apidocs/)).
The RMLMapper loads all data in memory, so be aware when working with big datasets.

## GitHub Action usage

```yaml
steps:
- uses: vemonet/rmlmapper-java@4.8
  with:
    mapping: mapping.rml.ttl
    output: rdf_output.nq
    serialization: nquads
```

> Serialization format can be `nquads` (default), `trig`, `trix`, `jsonld`, `hdt`.

## Table of contents

- [Features](#features)
- [Build](#build)
- [Usage](#usage)
  - [CLI](#cli)
  - [Library](#library)
  - [Docker](#docker)
  - [Including functions](#including-functions)
- [Testing](#testing)
- [Dependencies](#dependencies)
- [Remarks](#remarks)
- [Documentation](#documentation)
  - [UML Diagrams](#uml-diagrams)

## Features

### Supported
- local data sources:
 - CSV files (including CSVW)
 - JSON files (JSONPath)
 - XML files (XPath)
- remote data sources:
 - relational databases (MySQL, PostgreSQL, and SQLServer)
 - SPARQL endpoints
 - files via HTTP urls (via GET)
  - CSV files
  - JSON files (JSONPath)
  - XML files (XPath)
- functions (most cases)
- configuration file
- metadata generation
- output formats: nquads (default), turtle, trig, trix, jsonld, hdt
- join conditions

### Future
- functions (all cases)
- conditions (all cases)
- data sources:
   - NoSQL databases
   - web APIs
   - TPF servers

## Build
The RMLMapper is build using Maven: `mvn install`.
A standalone jar can be found in `/target`.

## Usage

### CLI
The following options are most common.

- `-m, --mapping <arg>`: one or more mapping file paths and/or strings (multiple values are concatenated).
- `-o, --output <arg>`: path to output file
- `-s,--serialization <arg>`: serialization format (nquads (default), trig, trix, jsonld, hdt)

All options can be found when executing `java -jar rmlmapper.jar --help`,
that output is found below.

```
usage: java -jar mapper.jar <options>
options:
 -c,--configfile <arg>            path to configuration file
 -d,--duplicates                  remove duplicates in the output
 -dsn,--r2rml-jdbcDriver <arg>    DSN of the database when using R2RML rules
 -e,--metadatafile <arg>          path to output metadata file
 -f,--functionfile <arg>          one or more function file paths (dynamic functions with relative paths are found relative to the cwd)
 -h,--help                        show help info
 -l,--metadataDetailLevel <arg>   generate metadata on given detail level
                                  (dataset - triple - term)
 -m,--mappingfile <arg>           one or more mapping file paths and/or
                                  strings (multiple values are
                                  concatenated). r2rml is converted to rml 
                                  if needed using the r2rml arguments.
 -o,--outputfile <arg>            path to output file (default: stdout)
 -p,--r2rml-password <arg>        password of the database when using R2RML rules
 -s,--serialization <arg>         serialization format (nquads (default),
                                  turtle, trig, trix, jsonld, hdt)
 -t,--triplesmaps <arg>           IRIs of the triplesmaps that should be
                                  executed in order, split by ',' (default
                                  is all triplesmaps)
 -u,--r2rml-username <arg>        username of the database when using R2RML rules
 -v,--verbose                     show more details in debugging output
```

#### Accessing Oracle Database

You need to add the Oracle JDBC driver manually to the class path
if you want to access an Oracle Database.
The required driver is `ojdbc8`.

- Download `ojdbc8.jar` from [Oracle](https://www.oracle.com/database/technologies/jdbc-ucp-122-downloads.html).
- Execute the RMLMapper via 

```
java -cp 'rmlmapper.jar:ojdbc8-12.2.0.1.jar' be.ugent.rml.cli.Main -m rules.rml.ttl
```

The options do the following:

- `-cp 'rmlmapper.jar:ojdbc8-12.2.0.1.jar'`: Put the jar of the RMLMapper and JDBC driver in the classpath.
- `be.ugent.rml.cli.Main`: `be.ugent.rml.cli.Main` is the entry point of the RMLMapper.
- `-m rules.rml.ttl`: Use the RML rules in the file `rules.rml`.ttl.
The exact same options as the ones mentioned earlier are supported.

### Library

An example of how you can use the RMLMapper as an external library can be found
at [./src/test/java/be/ugent/rml/readme/ReadmeTest.java](https://github.com/RMLio/rmlmapper-java/blob/master/src/test/java/be/ugent/rml/readme/ReadmeTest.java)

### Docker

You can use Docker to run the RMLMapper by following these steps:

- Build the Docker image: `docker build -t rmlmapper .`.
- Run a Docker container: `docker run --rm -v $(pwd):/data rmlmapper -m mapping.ttl`.

The same parameters are available as via the CLI.
The RMLMapper is executed in the `/data` folder in the Docker container.

### Including functions

There are two ways to include (new) functions within the RML Mapper
  * dynamic loading: you add links to java files or jar files, and those files are loaded dynamically at runtime
  * preloading: you register functionality via code, and you need to rebuild the mapper to use that functionality

Registration of functions is done using a Turtle file, which you can find in `src/main/resources/functions.ttl`

The snippet below for example links an fno:function to a library, provided by a jar-file (`GrelFunctions.jar`).

```turtle
@prefix dcterms: <http://purl.org/dc/terms/> .
@prefix doap:    <http://usefulinc.com/ns/doap#> .
@prefix fno:     <https://w3id.org/function/ontology#> .
@prefix fnoi:    <https://w3id.org/function/vocabulary/implementation#> .
@prefix fnom:    <https://w3id.org/function/vocabulary/mapping#> .
@prefix grel:    <http://users.ugent.be/~bjdmeest/function/grel.ttl#> .
@prefix grelm:   <http://fno.io/grel/rmlmapping#> .
@prefix rdfs:    <http://www.w3.org/2000/01/rdf-schema#> .

grel:toUpperCase a fno:Function ;
  fno:name "to Uppercase" ;
  rdfs:label "to Uppercase" ;
  dcterms:description "Returns the input with all letters in upper case." ;
  fno:expects ( grel:valueParam ) ;
  fno:returns ( grel:stringOut ) .

grelm:javaString
    a                  fnoi:JavaClass ;
    doap:download-page "GrelFunctions.jar" ;
    fnoi:class-name    "io.fno.grel.StringFunctions" .

grelm:uppercaseMapping
    a                    fnoi:Mapping ;
    fno:function         grel:toUpperCase ;
    fno:implementation   grelm:javaString ;
    fno:methodMapping    [ a                fnom:StringMethodMapping ;
                           fnom:method-name "toUppercase" ] .
```

#### Dynamic loading

Just put the java or jar-file in the resources folder,
at the root folder of the jar-location,
or the parent folder of the jar-location,
it will be found dynamically.

> Note: the java or jar-files are found relative to the cwd.
You can change the functions.ttl path (or use multiple functions.ttl paths) using a commandline-option (`-f`).

#### Preloading

This overrides the dynamic loading.
An example of how you can use Preload a custom function can be found
at [./src/test/java/be/ugent/rml/readme/ReadmeFunctionTest.java](https://github.com/RMLio/rmlmapper-java/blob/master/src/test/java/be/ugent/rml/readme/ReadmeFunctionTest.java)

## Testing

Run the tests via `test.sh`.

### RDBs
Make sure you have [Docker](https://www.docker.com) running.

#### Problems
* A problem with Docker (can't start the container) causes the SQLServer tests to fail locally. These tests will always succeed locally.
* A problem with Docker (can't start the container) causes the PostgreSQL tests to fail locally on Windows 7 machines.

## Dependencies

|             Dependency             | License                                                            |
|:----------------------------------:|--------------------------------------------------------------------|
| com.spotify docker client          | Apache License 2.0                                                 |
| com.h2database h2                  | Eclipse Public License 1.0 & Mozilla Public License 2.0            |
| com.googlecode.zohhak              | GNU Lesser General Public License v3.0                             |
| com.microsoft.sqlserver mssql-jdbc | MIT                                                                |
| ch.vorbuger.mariaDB4j              | Apache License 2.0                                                 |
| mysql-connector-java               | GNU General Public License v2.0                                    |
| com.google.guava                   | Apache License 2.0                                                 |
| javax.xml.parsers jaxp-api         | Apache License 2.0                                                 |
| com.jayway.jsonpath                | Apache License 2.0                                                 |
| junit                              | Eclipse Public License 1.0                                         |
| org.eclipse.rdf4j rdf4j-runtime    | Eclipse Public License 1.0                                         |
| commons-cli                        | Apache License 2.0                                                 |
| commons-csv                        | Apache License 2.0                                                 |
| commons-lang                       | Apache License 2.0                                                 |
| ch.qos.logback                     | Eclipse Public License 1.0 & GNU Lesser General Public License 2.1 |
| org.rdfhdt.hdt-jena                | GNU Lesser General Public License v3.0                             |
| io.fno.grel                        | MIT                                                                |

## Commercial Support

Do you need...

-   training?
-   specific features?
-   different integrations?
-   bugfixes, on _your_ timeline?
-   custom code, built by experts?
-   commercial support and licensing?

You're welcome to [contact us](mailto:info@rml.io) in regards to
on-premise, enterprise, and internal installations, integrations, and deployments.

We have commercial support available.

We also offer consulting for all-things-RML.

## Remarks

### XML file parsing performance

The RMLMapper's XML parsing implementation (`javax.xml.parsers`) has been chosen to support full XPath.
This implementation causes a large memory consumption (up to ten times larger than the original XML file size).
However, the RMLMapper can be easily adapted to use a different XML parsing implementation that might be better suited for a specific use case.

### Language tag support

The processor checks whether correct language tags are not, using a regular expression.
The regex has no support for languages of length 5-8, but this currently only applies to 'qaa..qtz'.

## Documentation
Generate static files at /docs/apidocs with:
```
mvn javadoc:javadoc
```

### UML Diagrams

#### Architecture UML Diagram
##### How to generate with IntelliJ IDEA
(Requires Ultimate edition)

* Right click on package: "be.ugent.rml"
* Diagrams > Show Diagram > Java Class Diagrams
* Choose what properties of the classes you want to show in the upper left corner
* Export to file > .png  | Save diagram > .uml

#### Sequence Diagram
##### Edit on [draw.io](https://www.draw.io)
* Go to [draw.io](https://www.draw.io)
* Click on 'Open Existing Diagram' and choose the .html file
