# VDA 231-301 PDF Package

A PDF/A-based exchange package combining a readable report, one main VDA 231-301
document, zero or more VDA 231-301 sub-documents, and binary attachments, linked
through a structured manifest.

This repository defines a draft container format for software developers and system
integrators who exchange VDA 231-301 data. It contains the format description, a JSON
Schema for the manifest, and example manifests.

## Status

This is a working draft. It is not an approved VDA recommendation or a released
interoperability specification. The repository does not yet provide a package writer,
reader, or PDF/A conformance validator.

The package adds a transport layer around VDA 231-301 payloads. The manifest contract
has draft version `0.1.0`. It does not replace the payload schemas. Payload references
to embedded attachments require an extension to `Generic.Attachment`; that extension
is not yet defined in this repository.

## Repository contents

| File | Purpose |
| --- | --- |
| [Manifest schema](schemas/manifest.schema.json) | JSON Schema Draft 2020-12 for manifest version `0.1.0`. |
| [Main project](examples/main-project.manifest.json) | One main project document without sub-projects or separate attachments. |
| [Main project with sub-projects](examples/main-with-sub-projects.manifest.json) | An initial sample inspection report and two lab reports. |
| [Projects with attachments](examples/with-attachments.manifest.json) | An initial sample inspection report, two lab reports, and two separately embedded files. |

The examples are manifest examples, not complete PDF packages. Their hashes, byte
lengths, UUIDs, and payload schema identifiers are illustrative values. The
`example.org` schema identifiers are explicit placeholders. Replace these values
with metadata from the actual files before creating a package. No example payloads
or binary files are included, and no file-integrity or payload-validation claim is
made for these examples.

## Purpose and scope

The package supports use cases in which one exchange object must serve both
human review and automated processing.

The visible PDF pages provide the human-readable report. Embedded VDA 231-301
JSON documents provide structured data for software systems. The manifest
identifies the embedded documents and attachments and supports package-level
integrity checks.

The package adds a container and transport layer. It does not replace the
VDA 231-301 Generic Schema, application-specific subschemas, payload validation,
or application-specific business rules.

### PDF/A background and boundaries

PDF/A is a family of PDF profiles for the long-term preservation of electronic
documents, defined in the ISO 19005 series.

PDF/A-3 permits embedded files of arbitrary types. PDF/A-4f provides an
alternative for embedded-file use cases based on PDF 2.0. These profiles are not
interchangeable without considering implementation and validation requirements.

The package specification needs to identify the complete required PDF/A profile,
not only the general designation "PDF/A".

The selected profile and package rules need to define:

- permitted embedded-file types;
- PDF-level associations, including applicable AFRelationship values;
- embedded-file metadata requirements; and
- the PDF/A validation procedure.

PDF-level associations do not replace the domain-specific attachment
relationships defined in the VDA payloads.

PDF/A conformance alone does not demonstrate:

- VDA 231-301 payload conformance;
- manifest or inventory correctness;
- agreement between the report and payloads;
- authenticity of the sender;
- safety of embedded files; or
- compliance with application-specific business rules.

A PDF/A declaration in metadata is not proof of conformance. The enclosing PDF
needs to be checked with a suitable validator for the selected profile.

### Related approach: ZUGFeRD / Factur-X

ZUGFeRD / Factur-X combines a human-readable PDF/A-3 invoice with embedded
structured XML invoice data.

The VDA 231-301 PDF Package uses a comparable hybrid-document principle, but
addresses a different domain and package structure:

- business data is represented by VDA 231-301 JSON payloads;
- a package can contain a main payload and multiple sub-project payloads;
- supporting attachments can be embedded separately; and
- a manifest inventories the embedded payloads and attachments.

This comparison is informative only. It does not imply ZUGFeRD / Factur-X
conformance or import its invoice-specific rules, precedence rules, or legal
treatment into this draft.

VDA-specific consistency rules, attachment references, and validation
requirements need to be defined independently.

### Terminology: payload

A payload is a concrete VDA 231-301 JSON document containing the business data
being exchanged. Both the main project document and each sub-project document
are payloads.

A payload is not the manifest, a JSON Schema, the visible report, or a separately
embedded binary attachment.

The manifest describes the package inventory. The applicable VDA 231-301 schema
defines the structure of each payload. These are separate validation layers.

## Package structure

A package is one PDF document. Its pages contain the main report. Its embedded files
contain one manifest, exactly one main VDA 231-301 document, zero or more VDA 231-301
sub-documents, and any binary attachments. Each VDA document is a JSON payload.

| Part | Purpose |
| --- | --- |
| Main report | Present the report in a form that a person can read. |
| Manifest | Identify and describe the embedded payloads and attachments. |
| Main project document | Provide the single VDA 231-301 JSON document for the main project. |
| Sub-project document | Provide a VDA 231-301 JSON document for one sub-project of the main project. |
| Binary attachment | Provide supporting content, such as an image or raw measurement data. |

The main report is the page content of the package itself. It is not a second PDF
embedded in the package. The report must be understandable without extracting the
embedded files.

The project structure has two levels: one main project and its sub-projects. For
example, the main document can be an initial sample inspection report
(Erstmusterprüfbericht), with tensile-test and emission-test lab reports as
sub-documents. A package with no sub-projects still has one main project document.

```text
Main project: initial sample inspection report (VDA JSON)
    +-- Sub-project: tensile-test lab report (VDA JSON)
    +-- Sub-project: emission-test lab report (VDA JSON)
```

The main project document is embedded JSON. The main report is the readable PDF page
content. These are distinct parts of the package.

The physical file structure is flat. A package does not use nested PDF packages.
The manifest identifies the main document and groups the sub-documents under it.
It does not support further nested sub-projects. Relationships between domain objects
and attachments belong in the payloads. Each separately embedded attachment retains
its original file bytes.

This draft supports embedded files only. It does not define external attachment
URLs or automatic downloads. A payload schema identifier identifies a schema; it
is not an external attachment location.

## Authoring principle

> Proposed guidance for project-group review.

The preferred authoring approach is to generate the visible report and the
VDA 231-301 payloads from the same authoritative source data and the same
revision of that data.

Report and payload should not be maintained independently where this can
reasonably be avoided.

This approach reduces the risk of inconsistencies but does not prove that the
representations agree. Rendering, mapping, rounding, and data-selection errors
can still occur.

The report may summarize information contained in several payloads. It does not
need to reproduce every payload field. The scope of the summary and the
relationship to the underlying documents should be clear.

## Report and payload consistency

> Proposed guidance for project-group review. This section does not establish
> legal precedence between the report and the payloads.

Receiving systems use the validated VDA 231-301 payloads for automated data
processing. The visible report provides the human-readable representation
and context.

Where both representations describe the same business information, they should
agree in meaning. Consistency does not require identical text formatting:
decimal separators, unit conversions, rounding, and summaries may differ if
their interpretation is defined and does not change the result.

Depending on the use case, consistency checks may cover:

- report identifiers and revisions;
- component and specimen identifiers;
- test methods and conditions;
- reported values and units;
- target values or limits;
- assessments or verdicts; and
- qualifications, deviations, and limitations relevant to interpretation.

A technically valid payload does not automatically override a conflicting
report. If a relevant conflict is detected, the receiving system should:

1. identify and report the conflict;
2. prevent automatic acceptance of the affected result;
3. preserve both representations without silently overwriting either; and
4. route the conflict through the applicable clarification process.

A file hash verifies file integrity against the recorded digest. It does not
prove semantic consistency between the report and the payloads.

Use-case-specific rules should define which information must agree, how report
content maps to payload content, and which differences are acceptable. A
universal automated comparison is not defined by this draft.

### Validation levels and reporting

Validation results should identify which checks were performed. Passing one
validation level does not imply that the other levels have passed.

| Level | Scope |
|---|---|
| PDF/A conformance | Conformance of the enclosing PDF to the selected PDF/A profile |
| Manifest conformance | Validation of manifest.json against the manifest schema |
| Inventory integrity | Resolution of embedded files and verification of uniqueness, decoded sizes, media types, and hashes |
| Payload conformance | Validation of each payload against its declared VDA schema |
| Attachment-reference integrity | Resolution of payload attachment references once the required Generic Schema extension is defined |
| Semantic consistency | Agreement of overlapping business information in report and payloads |
| Business validation | Application-specific completeness, plausibility, and acceptance rules |
| Authenticity and trust | Sender or signature verification where required by the receiving process |

A validation report should record:

- the package identifier;
- the schemas and profiles used;
- the validators or checking tools and their versions;
- the result of each applicable validation level; and
- any failed, unsupported, or unperformed checks.

Checks that were not performed should not be reported as passed.
A package should not be described simply as "valid" without stating the
validation scope.

## Manifest

Embed exactly one UTF-8 JSON manifest under the reserved name `manifest.json`, with
media type `application/json`. This reserved name is the entry point for manifest
discovery. Do not use it for a payload or attachment. The example filenames in this
repository identify different scenarios; rename the selected file when embedding it.

The manifest does not list itself or the main report. This avoids a self-referential
hash: the report is part of the enclosing PDF, which also contains the manifest.

All properties in the following tables are required. Unknown properties are rejected
by the draft schema. Arrays can be empty only where stated.

| Property | Meaning |
| --- | --- |
| `schema_version` | Manifest contract version. The current value is `0.1.0`. |
| `package_uuid` | UUID of this package. Use a new UUID when creating a revised package. |
| `created_at` | Creation time in RFC 3339 form, including a UTC offset. |
| `main_project` | Exactly one file entry for the main VDA 231-301 document. |
| `sub_projects` | File entries for VDA 231-301 sub-documents of the main project. Use an empty array if there are none. |
| `attachments` | File entries for separately embedded attachments. Use an empty array if there are none. |

Each file entry has these properties:

| Property | Meaning |
| --- | --- |
| `file_uuid` | UUID that identifies the file entry within the package. |
| `name` | Human-readable filename. It is not a file lookup key. |
| `mime_type` | Media type without parameters, such as `image/png`. |
| `size` | Non-negative byte length of the original file. |
| `sha256` | SHA-256 digest as 64 lowercase hexadecimal characters. |

Each payload entry also has `schema` and `schema_version`. `schema` is an absolute
URI that identifies the payload schema. `schema_version` identifies its version.
The payload media type must be `application/json`. Manifest and payload schema
versions are independent. Use a trusted local schema registry when resolving schema
identifiers; the identifier does not require a network request.

File UUIDs must be unique across `main_project`, `sub_projects`, and `attachments`.
UUIDs use lowercase hyphenated form.
Record each distinct file once. Two entries must not have the same SHA-256 digest.
Several payloads can use the same attachment entry.

Except for the manifest, every embedded file must have exactly one inventory entry.
Every inventory entry must resolve to one embedded file. Match files by the SHA-256
digest of their decoded bytes, then check their size and media type. Reject missing,
ambiguous, unlisted, or mismatched files. Do not fall back to a filename match.

These cross-entry and file-content rules require package-level checks. JSON Schema
does not enforce them.

## Attachment references in payloads

The intended relationship is:

```text
VDA payload: Generic.Attachment
    -> attachment reference
    -> manifest attachment entry
    -> embedded file
```

The manifest does not assign attachments to payloads. The payload places each
attachment at the relevant domain object. Multiple attachment objects can refer to
the same embedded file, including objects in different payloads.

The intended generic-schema extension lets an attachment contain either inline
Base64 data or a reference to a separately stored file. In this profile, that file
must be embedded in the same PDF package. "External" in an attachment reference
would mean external to the JSON payload, not external to the PDF package.

The current generic schema in the VDA repository requires `Data` and
`Encoding: "base64"` in `Generic.Attachment`. It also rejects additional properties.
Omitting `Data` or adding a reference property therefore requires a compatible
revision of that schema. A manifest cannot relax those requirements.

The reference property name, its value structure, and its lookup key remain to be
defined in the generic schema. The manifest exposes a file UUID and a SHA-256 digest
for that work. This draft does not define a new payload property or claim that
reference-based attachments already validate against the current generic schema.
The example manifests show the file inventory only; they do not demonstrate the
pending payload extension.

## PDF/A basis

The initial proposal uses PDF/A-3 (ISO 19005-3). PDF/A-3 permits embedded files of
arbitrary types. PDF/A conformance applies to the PDF document; it does not prove
that an embedded JSON document conforms to a VDA schema.

Every embedded file must meet the PDF/A requirements for associated files. These
requirements include a media type and an `AFRelationship` value. The PDF Association
describes these requirements in its
[Associated Files application note](https://pdfa.org/download-area/publications/PDF20_AN002-AF.pdf).

The final PDF/A conformance level, the possible use of PDF/A-4f, and the mapping of
manifest, payloads, and attachments to `AFRelationship` values remain open for review.
The manifest schema alone does not settle these PDF-level requirements.


## Validation boundaries

A receiving system must check the following layers separately:

1. Check PDF/A conformance with a PDF/A validator, such as
   [veraPDF](https://verapdf.org/).
2. Validate the manifest against the schema for its declared version.
3. Check file identities, relationships, byte lengths, and hashes against the
   extracted files.
4. Validate each payload against its declared VDA 231-301 schema and version.
5. Check the applicable business rules and the consistency of the report and data.

JSON Schema checks the manifest structure and individual values. It does not inspect
the PDF, calculate file hashes, resolve cross-references, or validate embedded
payloads. A manifest that passes schema validation is not proof of a valid package.

Use a JSON Schema Draft 2020-12 validator with format checks enabled. The default
2020-12 dialect does not require `format` annotations to act as assertions. See the
[JSON Schema validation specification](https://json-schema.org/draft/2020-12/json-schema-validation).

## Reference package and interoperability testing

> Proposed acceptance criteria for a future interoperability release.

The repository should provide complete reference packages in addition to
illustrative manifest examples.

A reference package should contain:

- a readable report on the PDF pages;
- exactly one manifest.json;
- one main VDA 231-301 JSON payload;
- additional sub-project payloads where the scenario requires them;
- supporting attachments where the scenario requires them;
- actual file sizes and SHA-256 digests;
- resolvable package-internal file relationships; and
- identified payload schemas without illustrative placeholder identifiers.

Validation evidence should accompany each reference package. It should identify
the PDF/A profile, schema versions, checking tools, and expected results.

A baseline reference package can demonstrate the main payload without separately
referenced attachments.

A reference package demonstrating payload-to-attachment references depends on
the corresponding Generic Schema extension. Until that extension is available,
such a scenario should be labelled as a proposal and should not claim
conformance with an unchanged schema that does not support it.

Interoperability testing should include extraction and processing by a reader
implemented independently of the package writer.

Tests should verify that the reader can:

1. discover the manifest;
2. resolve the embedded files using the defined matching rules;
3. verify the inventory and file integrity;
4. validate the payloads;
5. resolve supported attachment references; and
6. report relevant consistency or business-rule failures.

Negative test cases should include missing files, unlisted files, incorrect
hashes, ambiguous file matches, invalid payloads, and relevant report-to-payload
conflicts.

Successful processing by the writer's own reader alone is not sufficient
evidence of interoperability.

## Integrity and safe processing

Calculate SHA-256 hashes over the original file bytes after extraction and PDF stream
decoding. Do not hash the compressed PDF stream. Do not normalize JSON whitespace,
line endings, or character encoding before calculating a hash.

A hash detects a mismatch against the manifest. It does not authenticate the sender:
an attacker can replace both a file and its recorded hash. Signature requirements
and trust policies are outside the current draft.

Treat embedded files as untrusted input. Do not execute their contents. Apply local
limits to package size, extracted file size, file count, and parser resource use.
Treat embedded filenames as display labels, not as filesystem paths. Do not use
them directly to write files outside a controlled extraction directory.

## Proposed decisions for project-group review

The following points should be resolved or explicitly deferred before an
interoperability release:

- confirm the complete PDF/A profile and PDF-level association rules;
- define the handling of relevant conflicts between report and payloads;
- define use-case-specific consistency checks;
- complete the Generic Schema extension for separately embedded attachments;
- document compatibility with existing inline Base64 attachments;
- define validation evidence and reference-package acceptance criteria; and
- demonstrate interoperability with an independently implemented reader.

These proposals do not change the current manifest contract, the two-level
project structure, the flat embedded-file structure, or the existing
hash-based file-matching rules.

## License

This repository uses the [MIT License](LICENSE).
