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

## Report and payload consistency

The sender is responsible for consistency between the main report and the payloads.
A file hash can detect a change to file bytes. It cannot show that a report and a
payload describe the same result.

This draft does not establish which representation takes precedence when their
contents conflict. A receiving system must report such a conflict for resolution.
It must not silently replace one representation with the other.

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

## License

This repository uses the [MIT License](LICENSE).
