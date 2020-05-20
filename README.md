# OpenAutoComplete :pencil:

> Shell-agnostic, declarative CLI autocomplete specification

![markdownlint](https://github.com/GamePad64/openautocomplete/workflows/markdownlint/badge.svg)

The spec itself: [here](SPECIFICATION.md)

Nowadays, app developers have to provide autocompletion scripts for their applications for every shell possible by
themselves. Usually, they provide autocompletions for bash, zsh and fish, and other shells have to support these
completions (and this task is not easy, if even possible).

We want to have a single way an application could provide its autocompletion specification for the user's shell.

## Basic principles

### Declarative

The spec should have means to create CLI autocompletion in a declarative way, *mostly* without using shell scripting.

### Shell-agnostic

There should not be a great problem to write autocompletion module for every shell available with minimum usage of
non-standard tools. Our target shells: bash, zsh, fish, PowerShell and others.

### Standards-based
 
OAC uses JSON, so the **Runtime** implementers can use `jq` for Unix shells, `ConvertFrom-Json` in PowerShell and other
common approaches to parse OAC document.

### Backwards-compatible with legacy applications

The spec should have means to describe most of the today's application CLIs.

## Roadmap

Every single item in the roadmap is a subject for discussion

### Enumerations support

Argument completion with values, being a part of some sort of enumeration:

- path (file/directory) (with regex/wildcard filter)
- storage-volume -- mounted volume. On Unix-like: "/home", "/" (use `df`?)
- packages (installed/not installed). Tricky one.
- PIDs
- process names
- something else

### Enumerations from command output

This is for commands, like tar, thar can provide some machine-readable list, that could be parsed using a simple regex,
and turned into an enumeration list. Also, this could be used for commands, that can provide their own autocompletion
lists, like `dotnet`.

### Delegation to another OAC instance

This could be used for delegating command autocompletion to another instance.
Better with an example.

Consider a Python Django package.
`python -m django <TAB><TAB>`

We provide python autocompletion by ourselves, and after `-m <arg>` it contains *delegation* pattern. When the user
invokes autocompletion (double TAB), the Runtime gets definitions for `python`, then proceeds to a command inside
*delegation*, and tries to get OAC definitions for django package (how?). If successful, it returns a list of currently
available django commands:
`[check, compilemessages, createcachetable, dbshell, diffsettings, dumpdata, flush, inspectdb, ...]`

### shell-dependent scripting for delegations or enumerations
Well, to make completions more powerful, some sort of shell-dependent scriptiong would be useful (but discouraged). We should think about safety considerations, though.
