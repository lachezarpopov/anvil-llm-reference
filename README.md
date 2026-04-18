# Anvil LLM Reference

This repository contains portable reference material for AI agents and developers working on Anvil applications stored as local project files.

The main reference set lives in [`anvil_reference/`](anvil_reference/), with agent instructions in [`AGENTS.md`](AGENTS.md). Together, they are designed to be copied into an Anvil app repository so an agent can quickly understand Anvil-specific structure, form YAML, client/server boundaries, Data Tables, layout behavior, custom components, routing, and verification practices without rediscovering the framework from scratch.

## Purpose

The goal of this repository is to provide a compact, app-neutral source of Anvil guidance that can travel with a project. It is not an Anvil app itself, and it is not a replacement for the target app's source code or official Anvil documentation.

How to use:

- Copy `anvil_reference/` into an Anvil app repository.
- Copy `AGENTS.md` into the same repository, or merge its instructions into an existing `AGENTS.md`.

Guidelines:

- Read `anvil_reference/README.md` for the reference index and recommended reading order.
- Follow the target app's actual conventions whenever they differ from these generic notes.
- Check official Anvil documentation, and when needed the open-source Anvil runtime, for behavior that may have changed.

## Repository Layout

- [`anvil_reference/`](anvil_reference/) contains the portable Anvil reference docs.
- [`anvil_reference/README.md`](anvil_reference/README.md) is the index for the reference set itself.
- [`AGENTS.md`](AGENTS.md) gives agent-facing instructions for working with this repository.

## License

This project is licensed under the MIT License. See [`LICENSE`](LICENSE).
