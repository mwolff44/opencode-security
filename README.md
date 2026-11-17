# opencode-security

Exploring a dedicated OpenCode security plugin for outbound-secret protection
and defense against prompt injection.

**Status: discovery only. No security plugin or runtime protection is
implemented in this repository yet.**

## What

A proposed plugin that reduces accidental disclosure of credentials to model
providers and helps constrain actions induced by hostile content.
The Wayfinder effort will decide its scope, architecture, measurable guarantees,
and residual risks before implementation.
Detection alone is not enforcement or proof that content is safe.

## Why

File permissions and pre-commit secret scanning protect different boundaries:
neither proves that an assembled model request is free of secrets.
Prompt injection can also redirect an agent through repository files, Skills,
web pages, or tool results.
The project investigates complementary controls rather than replacing native
permissions, policies, or system isolation.

## Who

Initially built around the maintainer's OpenCode and OpenChamber workflow,
with reuse in mind.
Broader client, platform, provider, and transport compatibility must be
demonstrated rather than assumed.

## Where

Plugin source, research, and compatibility tests belong in this independent
repository.
[GitHub Issues](https://github.com/mwolff44/opencode-security/issues) holds the
Wayfinder map and its decision tickets.
`ia-dev-tools` may consume the eventual plugin; the plugin must not require that
configuration kit.
The initial threat model assumes a trusted host and runtime, but potentially
hostile repository, Skill, web, and tool-result content.
Resistance to an already compromised plugin or OS account is outside that
boundary.

## When

First chart and resolve the Wayfinder decisions, then produce a specification
ready for implementation tickets.
Do not install or rely on this repository as a security control today.
Publication of a release will require explicit coverage and limitation evidence.

## Discovery and contribution

Start with
[Wayfinder: define verifiable OpenCode security protections](https://github.com/mwolff44/opencode-security/issues/1).
Research must distinguish verified facts from hypotheses, cite source revisions,
and use synthetic data only.
Implementation is outside the current discovery effort.
All project artifacts are written in English.

## License

See `LICENSE`.
