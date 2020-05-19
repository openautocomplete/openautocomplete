# OpenAutoComplete Specification

Version 0.1 (development branch)

The OpenAutoComplete Specification is licensed under
[Creative Commons Attribution 4.0 International (CC BY 4.0)](https://creativecommons.org/licenses/by/4.0/).

## Status

This page presents the development version of OpenAutoComplete Specification. This version MUST NOT be considered
stable and can be changed in any time without any notification.

## Introduction

The OpenAutoComplete Specification (OAC) defines a standard, language-agnostic interface for command-line
applications, which allows other applications, like shells and SDKs to perform CLI argument introspection in a language-
and platform-independent way.

## Conventions

The key words "MUST", "MUST NOT", "REQUIRED", "SHALL", "SHALL NOT", "SHOULD", "SHOULD NOT", "RECOMMENDED",
"NOT RECOMMENDED", "MAY", and "OPTIONAL" in this document are to be interpreted as described in
[RFC8174](https://tools.ietf.org/html/rfc8174) when, and only when, they appear in all capitals, as shown here.

## Versioning

This specification follows [Semantic Versioning 2.0.0](https://semver.org/spec/v2.0.0.html).

Major version denotes backwards-incompatible changes, like changing the format,
deleting certain fields and other changes, that MAY lead to incompatibilities of tools.
Minor version SHALL add new features to this Standard without making backwards-incompatible changes.
Patch version SHOULD be ignored by the tools.

## Definitions

**Runtime** -- an application, that interprets OAC document and uses its data to construct shell autocompletions or
help text or documentation of some kind.

## Document Structure

This section describes the structure of OAC document. OAC document is a JSON document
[RFC8259](https://tools.ietf.org/html/rfc8259). This JSON document MUST use a JSON object for its root element. This
object defines a document’s “top level”.

### Top level

Every OAC document MUST contain the following fields:

- `openautocompletion`: **[REQUIRED]** object, contatining a text field `version` with *major.minor* portion of OAC
specification. Example: `"openautocompletion": {"version": "0.1"}`
- `components`: **[REQUIRED]** object, containing `Components` object.
- `cli`: **[REQUIRED]** object, containing `Cli` object.

### Components object

`Components` object is a root for all reusable components for use across OAC document. These components are allowed
to be referenced

This object MAY contain the following fields:

- `arguments`: object, cointatining arbitrary values as keys, and `Argument` objects as values.
- `options`: object, cointatining arbitrary values as keys, and `Option` objects as values.
- `commands`: object, cointatining arbitrary values as keys, and `Command` objects as values.

### Argument object

`Argument` object contains the following fields:

- `name`: string, the name, that identifies this argument.
- `description`: string, contains description of this item.

### Option object

`Option` object contains the following fields:

- `names_short`: array, contains short option names as they appear in CLI, without prefix. Example: ["s"]
- `names_long`: array, contains long option names as they appear in CLI, without prefix.

  This array SHOULD contain:
  - POSIX long options (long, double-dashed).
  - Legacy (long, single-dashed)
  - Windows-style (long and short, starting with forward slash "/")
  - PowerShell-style options (long, single-dashed)

  Example: ["simulate", "just-print", "dry-run", "recon", "no-act"]
- `description`: string, contains description of this item.

At least one of [`names_short`, `names_long`] MUST be included into `Option` object, and at least one element MUST be
present in either of these arrays.

Names MUST NOT include their prefixes (typically, dash '-', double dash '--' or slash '/').

### Command object

`Command` object contains the following fields:

- `names`: **[REQUIRED]** array, contains various names, that are used .
- `description`: string, contains description of this item.

At least one of [`names_short`, `names_long`] MUST be included into `Option` object, and at least one element must be
present in either of these arrays.

## Enumerations

`storage-volume`,
`path`,
