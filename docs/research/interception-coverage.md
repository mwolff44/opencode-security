# OpenCode interception and enforcement coverage

Research resolution for
[Establish OpenCode interception and enforcement coverage](https://github.com/mwolff44/opencode-security/issues/2).

## Resolution

OpenCode exposes useful request-transformation and action-control seams, but
source inspection does not establish complete interception of the maintainer's
running OpenCode + OpenChamber stack.
A plugin design is plausible; a universal confidentiality or injection-prevention
guarantee is not yet supported.

Three distinctions matter:

1. Detection is not enforcement, but verified request rewriting can enforce
   removal on the paths it actually covers. It is not merely an alert.
2. A typed tool-rejection channel is demonstrated. A general typed provider-call
   rejection channel is not demonstrated by the inspected hook contract.
   This does not prove that every thrown exception fails open.
3. Native permission decisions and policies remain necessary. A plugin cannot
   sandbox code already executing inside the trusted server.

No runtime acceptance probes were executed. No provider was contacted, service
started, external code installed or executed, credential store read, or machine
configuration changed.

## Version and evidence

The parent's local `opencode --version` command returned `opencode v2.3.0`.
This identifies the CLI, not necessarily the already-running server.

The GitHub tag resolves to commit
`a6b15d33daf6ab7d35569f6d711fdb630b9f04e7`:
[tag reference](https://api.github.com/repos/anomalyco/opencode/git/ref/tags/v2.3.0).
Source links below use that immutable commit.

Current documentation was read on 2026-11-17. It is not a frozen v2.3.0 contract:

- [Server plugin APIs and hooks](https://opencode.ai/docs/build/plugins/server)
- [Permissions](https://opencode.ai/docs/permissions)
- [Policies and authority precedence](https://opencode.ai/docs/policies)

Pinned sources independently checked by the parent:

- [Hook runner and failure-channel declaration](https://github.com/anomalyco/opencode/blob/a6b15d33daf6ab7d35569f6d711fdb630b9f04e7/packages/core/src/plugin/hooks.ts)
- [Permission evaluation, saved approvals, and unattended settlement](https://github.com/anomalyco/opencode/blob/a6b15d33daf6ab7d35569f6d711fdb630b9f04e7/packages/core/src/permission.ts)

Evidence levels: **S** = pinned source; **D** = current official documentation;
**U** = unresolved runtime or version-specific coverage.
None of the rows below is a successful live security test.

## Coverage matrix

| Path | Candidate seam | Supported conclusion | Evidence / gap |
| --- | --- | --- | --- |
| User prompt admission | `session.hook("prompt")` | Mutates the draft before attachment/Skill resolution and durable admission. The docs expose no typed rejection API. | D; exceptions and preparation failure must not be repurposed into an undocumented blocking contract. |
| Primary model loop | `session.hook("context")` | Can transform assembled instructions, messages, tools, and request options. | D; not sufficient for auxiliary requests. |
| Title | `session.hook("title")` | Separate hook; docs allow setting `result` to supply the title and skip generation. | D; no agent/tools fields. |
| Compaction | `session.hook("compaction")` | Separate hook; docs allow supplying a result. | D; summary prompt is appended after these hooks, so hook timing matters. |
| Session transient generation | `session.hook("generate")` | Documents a separate session-generation path. | D; distinguish from no-session generation. |
| Direct plugin generation | `ctx.experimental.generate.*` | Exists without a session. | U: equivalent interception and auxiliary coverage are not established. |
| Subagents | Tool admission, child permissions, child model hooks | Native `subagent` permission can restrict target agents. Custom subagents have their own permissions. | D; prove model-hook reach and location scope for actual child sessions. |
| Native provider HTTP | `http.request` / `http.response` | Docs expose mutable requests/responses and `kind` for primary, title, compaction, and generate flows. Bodies are one-shot streams. | D; provider-driver coverage and retries need capture tests. |
| Native WebSocket | `experimental.ws.handshake/send/receive` | Separate experimental frame hooks; HTTP hooks do not cover this traffic. | D; version support, framing, and fallback paths require proof. |
| Retry | `session.hook("retry")` | Can veto a proposed retry or change delay; not the initial call. | D; overflow recovery through compaction is distinct. |
| Files and attachments | Prompt draft and later request transformation | Hook-added files undergo normal resolution. | D; do not assume OCR, images, binary data, or every attachment text source is covered. |
| Tool execution | `tool.hook("execute.before")` | The typed `Tool.Error` channel can reject execution before the call runs. | S; coverage of built-ins, MCP, nested Code Mode, and background tools remains U. |
| Tool results | `execute.after` | Can inspect successful or failed calls and transform successful results. | D; durable-history persistence and every asynchronous return path remain U. |
| Permission decisions | Native rules, then `permission.evaluate` | An already-configured deny returns before the hook. Allow/ask decisions can be changed by hooks. | S; finality depends on the resolved rules, not on the mere presence of a deny earlier in a file. |
| Hard baseline | Native `policies` | Documentation gives broader authority precedence and restrictions on providers, tools, Skills, MCP, and plugins. | D; validate the installed/server version and resolved policy behavior. |
| Shell | Native scanner and permission checks; `shell.create.before` | Hooks can transform shell inputs; native scanner/rules own their documented enforcement. | S/D; shell still has host-user authority and directory inference is best effort. |
| Web/MCP | Tool permissions and policies | Can limit native web tools and MCP tool use. | D; no general inspection of arbitrary networking inside plugin/MCP processes is established. |

## Ordering and failure semantics

The pinned hook runner appends callbacks by domain/name and invokes them
sequentially. Later hooks see mutable changes from earlier hooks.
Provider-scoped callbacks are skipped on a provider-ID mismatch.

Its `Failures` declaration explicitly gives a typed failure channel only to
`tool.execute.before`; session, permission, shell, and AISDK hook failures are
typed `never`. The runner itself does not catch callback defects.
The ultimate result of an unexpected exception must be verified at the caller:
do not label it either safe refusal or silent passthrough from this file alone.

The pinned permission engine:

- uses last-match-wins within the assembled rule list;
- returns early for a resolved configured deny;
- considers saved project approvals before invoking the evaluation hook;
- accepts the evaluation hook's final mutable effect;
- when an unattended ruleset is supplied, settles remaining asks as allowed
  only when that ruleset permits every checked resource; otherwise denies.

A later trusted plugin can overwrite an earlier evaluation hook's effect.
Native policy/rule controls and plugin order are therefore separate design
concerns. Exact global/project/agent/session assembly must be checked beyond
this permission evaluator.

Documentation distinguishes persisted prompt edits from outgoing-request
transformations: prompt edits become canonical user input, while model-request
edits do not rewrite the original history. Result-mutation persistence is not
settled by the inspected material.

## Threat-model implications

- Malicious repository text is in scope. A malicious auto-loaded repository
  plugin is executable code in the trusted server and crosses the assumed
  trust boundary; discovery/allowlisting policy must be considered explicitly.
- A system-role message is not sufficient evidence of trusted provenance.
- Synthetic messages, shell messages, compaction controls, and move controls
  do not all pass through prompt admission. A prompt-only filter is incomplete.
- Direct generation and networking performed by trusted plugins cannot be
  assumed to pass through session hooks.
- A request transform may reduce disclosure without a universal refusal API.
  The specification must choose what to do when parsing, scanning, or transport
  coverage is incomplete instead of inventing a guarantee.
- Restricting action consequences and removing outbound secrets are related
  but distinct protections. Neither establishes immunity to prompt injection.

## Proposed synthetic acceptance probes

These are questions for a separately authorized isolated experiment, not work
performed in this research:

1. A `Tool.Error` before-hook rejection must prevent a harmless sentinel tool
   effect; repeat for built-in, MCP, nested Code Mode, and background paths.
2. A resolved configured deny must remain denied despite an evaluation hook
   returning allow. Separately prove global policy precedence over a project.
3. Capture exactly what reaches a loopback fake provider for primary, child,
   title, compaction, transient generation, and retry requests.
4. Exercise native HTTP, native WebSocket, driver alternatives, and fallback.
   Check missing hooks as well as positive callbacks.
5. Compare session generation with no-session plugin generation.
6. Make each hook fail independently and observe the caller's actual result;
   prove whether the request is absent, modified, or forwarded.
7. Register a benign modifier before and after the security transform to test
   ordering and whether later changes can reintroduce a synthetic secret.
8. Check attachment formats, JSON-encoded tool arguments, tool-result
   persistence, streamed output, and non-prompt message inlets.
9. Test real client approval capability, including a disconnected web client
   and an explicitly unattended run; non-TUI is not equivalent to no human.

## Remaining decision questions

The next live decision tickets must choose guarantees and failure behavior,
not assume those choices have been settled by this research.
The architecture ticket already covers selection of hooks, native controls,
provenance, auxiliary paths, and a narrowed scope or alternative boundary.
The acceptance ticket must turn the probes above into a safe, version-pinned
experiment before any public runtime guarantee is made.

The running server version, every-path tool/model interception, no-session
generation, precise plugin/location scope, and hook-defect containment remain
explicitly unverified.

## Archive context

Local branch: `research/interception-coverage`.
Setup baseline: `4c51cd0`.
The issue resolution is canonical; this file is its local research archive.
[Canonical resolution comment](https://github.com/mwolff44/opencode-security/issues/2#issuecomment-6947098065).
