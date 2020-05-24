# OpenAutoComplete Specification

Version 0.2 (development branch)

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

**Runtime**: an application, that interprets OAC document and uses its data to construct shell
autocompletions or help text or documentation of some kind.

This includes:

- shell plugins, that propose autocompletions
- shell plugins, that generate code for native shell autocompletion system on-the-fly
- stand-alone converters, that convert OAC document into a shell script

## Document Structure

This section describes the structure of OAC document. OAC document is a JSON document
[RFC8259](https://tools.ietf.org/html/rfc8259). This JSON document MUST use a JSON object for its root element. This
object defines a document’s “top level”.

### Top level

Every OAC document MUST contain the following fields:

- `openautocompletion` object, required: Contains a text field `version` with *major.minor* portion of OAC
specification. Example: `"openautocompletion": {"version": "0.1"}`
- `components` object, required: Contains `Components` object.
- `cli` object, required: Contains `Cli` object.

### Components object

`Components` object is a root for all reusable components for use across OAC document. These components are allowed
to be referenced

This object MAY contain the following fields:

- `arguments` object: Contains identifiers as keys, and `Argument` objects as values.
- `options` object: Contains identifiers as keys, and `Option` objects as values.
- `commands` object: Contains identifiers as keys, and `Command` objects as values.

**Runtime** MUST NOT rely on this object for getting a list of all command-line tokens, and it should be used only for
referencing purposes using `$ref` notation.

### Argument object

`Argument` object contains the following fields:

- `name` string: Name, that identifies this argument.
- `description` string: Description of this item.

### Option object

`Option` object contains the following fields:

- `names_short` array[string]: contains short option names as they appear in CLI, without prefix. Example: ["s"]
- `names_long` array[string]: contains long option names as they appear in CLI, without prefix.

  This array SHOULD contain:
  - POSIX long options (long, double-dashed).
  - Legacy (long, single-dashed)
  - Windows-style (long and short, starting with forward slash "/")
  - PowerShell-style options (long, single-dashed)

  Example: ["simulate", "just-print", "dry-run", "recon", "no-act"]
- `description` string: Description of this item.

At least one of [`names_short`, `names_long`] MUST be included into `Option` object, and at least one element MUST be
present in either of these arrays.

Names MUST NOT include their prefixes (typically, dash '-', double dash '--' or slash '/').

### Command object

`Command` object contains the following fields:

- `names` array[string], required:, contains various names, that are used ass command-line subcommands. This array MUST
contain at least. The first element SHOULD be the most unambigous command. For example, when `ci` and `clean-install`
have the same meaning, then the resulting array will be: `["clean-install", "ci"]`
- `description` string: Description of this item.

At least one of [`names_short`, `names_long`] MUST be included into `Option` object, and at least one element must be
present in either of these arrays.

### Patterns

Currently, PatternType values available are:

- `group`
- `command`
- `argument`
- `option`

When the **Runtime** encounters a pattern type, that is unknown or unprocessible, then the Runtime MUST skip it
gracefully, so the pattern list could be extended in future specifications.

Each pattern object contains following fields:

- `type` PatternType, required: a type of the pattern
- `repeated` bool, default: `false`: Marks the pattern, that it could be repeated multiple times
- `optional` bool, default: `false`: Marks the pattern, that it could be omitted from command

Field values, that are supposed to refer to an object in Components, MAY be referred using `$ref` notation, defined in
JSON Schema specification. The whole object, containing `$ref` field SHOULD be replaced inplace with a target object,
ignoring all other fields. References, that use `$ref` notation MUST be local.

Example: `{"$ref": "#/components/commands/install"}`

#### CommandPattern

Besides the basic pattern fields, this contains:

- `command` Command, required, reference: Command that this pattern represents

#### ArgumentPattern

Besides the basic pattern fields, this contains:

- `argument` Argument, required, reference: Argument that this pattern represents

#### OptionPattern

Besides the basic pattern fields, this contains:

- `separators_long` array[string], default: `["=", " "]`: separators, that should appear after long options.
- `separators_short` array[string], default: `[" ", ""]`: separators, that should appear after short options.
- `prefix_long` string, default: "--": prefix for long options.
- `prefix_short` string, default: "-": prefix for short options.
- `option` Option, required, reference: Option that this pattern represents
- `argument` ArgumentPattern: ArgumentPattern, that could be used to parse the argument for the option

#### GroupPattern

Besides the basic pattern fields, this contains:

- `exclusive` bool, default: `false`: If `true`, then all the group members are mutually exclusive. If `false`, then all
the group members are required (except for memnbers, explicitly marked as optional).
- `group` array[GroupPattern], required, reference: Option that this pattern represents

GroupPattern SHOULD be considered a recursive structure, representing a parsing DAG. It is not tree, because after
mutually a exclusive group, another required token might follow. Generally, during the parsing process, this DAG may be
converted to parsing tree.
Commands and Arguments in GroupPattern have strict order, and Options may follow in arbitrary order.

### Cli object

- `name` string, required: The command itself, as it appears in the shell. If this field is an array, then the
**Runtime** SHALL take the first element as a value.
- `pattern_groups` array[GroupPattern], required. This array MUST contain at least one GroupPattern. This array contains
all usage patterns for given command.

  Essentially,`pattern_groups` can be represented as a single GroupPattern with `exclusive: true` set and
  with all array items inside `group`.

## OAC-enabled app development considerations

For app to be OAC-enabled, it MUST support `--openautocomplete` option for its root command. The app, called with this
option MUST print a valid OAC document and exit immediately with code 0.

The easiest way to make the app OAC-enabled is to use an argument parser, that supports OpenAutoComplete, but there are
no parsers, that support it so far (you can make one!).
