# openautocomplete

## Motivation

Nowadays, app developers have to provide autocompletion scripts for their applications for every shell possible by
themselves. Usually, they provide autocompletions for bash, zsh and fish, and other shells have to support these
completions (and this task is not easy, if even possible). We want to have a single way an application could provide
its autocompletion specification for the user's shell.

## Basic principles

1. Declarative
1. Shell-agnostic: there should not be a great problem to write autocompletion module for every shell available with
minimum usage of non-standard tools. Our target shells: bash, zsh, fish, PowerShell and others.
1. Standards-based:
1. Backwards-compatible with legacy applications

## Enumerations

`storage-volume`,
`path`,
