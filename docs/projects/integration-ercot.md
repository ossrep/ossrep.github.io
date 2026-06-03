# integration-ercot

Standalone Quarkus library for integrating with ERCOT public data endpoints.

## Technology Stack

| Technology   | Purpose                      |
| ------------ | ---------------------------- |
| Quarkus 3.33 | CDI / bean discovery         |
| Java 25      | Language (latest LTS)        |
| Jackson      | JSON parsing                 |

## Architecture

This is a **library JAR** (not an application). It provides CDI beans that are automatically discovered when added as a dependency to any Quarkus project.

- **Core** (`com.ossrep.integration.ercot`) -- ERCOT MIS client, configuration, exceptions
- **ESIID** (`com.ossrep.integration.ercot.esiid`) -- Report 203 document service, CSV parser, record

All ERCOT MIS endpoints are public -- no authentication required.

## Supported Reports

| Report Type ID | Name                | Class                  | Description                              |
| -------------- | ------------------- | ---------------------- | ---------------------------------------- |
| 203            | TDSP ESI ID Extract | `EsiidDocumentService` | Daily and monthly ESIID files per TDSP   |

## Key Classes

### ErcotClient

Core HTTP client for the ERCOT MIS API. Provides:

- `listDocuments(int reportTypeId)` -- lists available documents for a report type
- `downloadDocument(String docId)` -- downloads a document by ID as an `InputStream`

### EsiidDocumentService

Report 203-specific wrapper around `ErcotClient`. Provides:

- `listDocuments()` -- lists ESIID documents with parsed TDSP and file type metadata
- `downloadDocument(EsiidDocument)` -- downloads a specific ESIID document

### EsiidCsvParser

Streaming parser for ESIID ZIP/CSV files. Processes records in configurable batch sizes to support large files without excessive memory usage.

- `parseZipStream(InputStream, Consumer<List<EsiidRecord>>)` -- extracts and parses CSVs from a ZIP
- `parseZipStream(InputStream, Consumer<List<EsiidRecord>>, int batchSize)` -- with custom batch size

## Configuration

| Property           | Default                          | Description         |
| ------------------ | -------------------------------- | ------------------- |
| `ercot.mis-base-url` | `https://www.ercot.com/misapp` | ERCOT MIS base URL  |

## Usage

Add the dependency:

```xml
<dependency>
    <groupId>com.ossrep</groupId>
    <artifactId>integration-ercot</artifactId>
    <version>1.0.0-SNAPSHOT</version>
</dependency>
```

Inject and use:

```java
@Inject EsiidDocumentService documentService;
@Inject EsiidCsvParser csvParser;

List<EsiidDocument> docs = documentService.listDocuments();
for (EsiidDocument doc : docs) {
    try (InputStream stream = documentService.downloadDocument(doc)) {
        csvParser.parseZipStream(stream, batch -> {
            // process batch of EsiidRecord
        });
    }
}
```

## Build

```shell
JAVA_HOME=/usr/lib/jvm/java-25-openjdk mvn install
```
