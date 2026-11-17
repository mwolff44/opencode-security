# Reusable security mechanisms and adversarial evidence

Research resolution for
[Assess reusable mechanisms and adversarial evidence](https://github.com/mwolff44/opencode-security/issues/3).

## Resolution

The strongest candidates are a request-boundary secret control, structured-field
awareness, native-authority baselines, minimal diagnostics, and adversarial
tests that inspect actual boundary effects.
Do not transplant either project wholesale.
Recommendations below are inputs to the live policy and architecture decisions,
not approved implementation scope.

Research was read-only. Tests were not executed, external code was not installed
or executed, and no real credentials or private transcripts were inspected.

## Source basis

Pinned repositories:

- **Pi**: [pi-secured-setup at 0d531a4100e50318833c258e0a9609b903cda0a7](https://github.com/mwolff44/pi-secured-setup/tree/0d531a4100e50318833c258e0a9609b903cda0a7).
- **Scrubber**: [prompt-scrubber at 8eee448af8cf9f2ac6bcbacc81dc40a6c7815285](https://github.com/Nano-Collective/prompt-scrubber/tree/8eee448af8cf9f2ac6bcbacc81dc40a6c7815285).
- **ia-dev-tools**: local source at `2f1c4223d787584f61d5eb427defea8b9d5991db`;
  the parent verified that revision before this research.

Important primary files:

- [Pi secret scanner](https://github.com/mwolff44/pi-secured-setup/blob/0d531a4100e50318833c258e0a9609b903cda0a7/lib/secret-scanner.ts),
  [injection scanner](https://github.com/mwolff44/pi-secured-setup/blob/0d531a4100e50318833c258e0a9609b903cda0a7/lib/injection-scanner.ts),
  [guard pipeline](https://github.com/mwolff44/pi-secured-setup/blob/0d531a4100e50318833c258e0a9609b903cda0a7/lib/guard-pipeline.ts).
- [Scrubber secret detector](https://github.com/Nano-Collective/prompt-scrubber/blob/8eee448af8cf9f2ac6bcbacc81dc40a6c7815285/src/detectors/secret.ts),
  [structured JSON walker](https://github.com/Nano-Collective/prompt-scrubber/blob/8eee448af8cf9f2ac6bcbacc81dc40a6c7815285/src/proxy/json-scrub.ts),
  [default detectors](https://github.com/Nano-Collective/prompt-scrubber/blob/8eee448af8cf9f2ac6bcbacc81dc40a6c7815285/src/core/scrub.ts).

Function names below are symbol locators within those immutable files.

## Verified findings

### Secret detection and structured fields

Pi's `walkAndRedact` scans string values, including system content, at
`before_provider_request`. It replaces findings irreversibly and emits
pattern names and lengths rather than secret values.
It does not use the parent field name to classify an otherwise opaque value.

Pi's pattern named `high-entropy` does not calculate Shannon entropy:
it recognizes a keyword-adjacent value of sufficient length.
Its `sk-` pattern requires contiguous alphanumerics after the prefix and misses
the intermediate-hyphen shape used by `sk-proj-` unless another contextual
pattern catches it.
These are reasons to adapt the mechanism rather than inherit its completeness.

Scrubber has layered pattern/key-name/entropy detection and whole-value
handling for recognized credential-named structured fields.
Its JSON mapping supports key-path context and JSON encoded inside tool-argument
strings. Those concepts address important gaps in a string-only walk.
False positives remain possible for identifiers and high-entropy non-secrets.

Correction to an earlier conversation hypothesis: Pi calls
`isPlaceholder(match[0])`. A vendor-prefixed match starts with the vendor prefix,
not `test` or `example`; the placeholder prefixes do not automatically bypass
those patterns. Value-only patterns require separate analysis.
See [Pi secret-scanner assertions](https://github.com/mwolff44/pi-secured-setup/blob/0d531a4100e50318833c258e0a9609b903cda0a7/test/secret-scanner.test.ts)
and [Scrubber secret-detector assertions](https://github.com/Nano-Collective/prompt-scrubber/blob/8eee448af8cf9f2ac6bcbacc81dc40a6c7815285/tests/detectors/secret.test.ts).

### Coverage limits and refusal

Both walkers leave deeply nested subtrees unprocessed at their safety cap:
Pi beyond depth 50 and Scrubber's JSON walker beyond 64.
Neither is general decoding of arbitrary base64, hex, or fragmented secrets.
Matching a complete credential-bearing URL is different from decoding every
encoded value.

Both projects replace detected secrets rather than necessarily refusing a
request containing one. However, Scrubber also has real strict-mode refusal
for unsupported request bodies and routes.
It must not be described as incapable of refusal:
[proxy tests: unknown-route passthrough versus strict refusal](https://github.com/Nano-Collective/prompt-scrubber/blob/8eee448af8cf9f2ac6bcbacc81dc40a6c7815285/tests/proxy/proxy-server.test.ts#L212-L257)
assert an unchanged body in ordinary mode and zero upstream requests in strict
mode. Those assertions were read, not run here.
Strict mode does not prove every field was scanned.

### Injection detection is not provenance or action enforcement

Pi marks matching strings with untrusted-content delimiters and notifies.
It does not block a provider request on this basis.
Its scanner excludes structural system carriers and roles and skips comment
lines. A role or key called `system` is not proof that every value originated
with a trusted authority.

The [Pi injection tests](https://github.com/mwolff44/pi-secured-setup/blob/0d531a4100e50318833c258e0a9609b903cda0a7/test/injection-scanner.test.ts)
contain attack and benign-control cases, carrier exclusions, tool-result
coverage, and a non-blocking contract.
They do not prove that a model obeys the markers or that unmatched content is
safe.

Action control lives in the separate guard pipeline.
Pi's boundary intentionally does not constrain Bash paths:
[ADR: no boundary enforcement for Bash](https://github.com/mwolff44/pi-secured-setup/blob/0d531a4100e50318833c258e0a9609b903cda0a7/docs/adr/0003-no-boundary-enforcement-for-bash.md).
Its non-TUI confirmation rule must not be copied literally into OpenChamber:
a remote web client may provide a valid human approval channel.

### Baseline, diagnostics, and existing local coverage

Pi restricts project-layer weakening of selected baseline rules:
[baseline-authority ADR](https://github.com/mwolff44/pi-secured-setup/blob/0d531a4100e50318833c258e0a9609b903cda0a7/docs/adr/0009-project-layer-config-cannot-weaken-baseline.md).
This is not a universal invariant: the project-level external-path list is
additive and can broaden permitted paths.

The [Pi integration test](https://github.com/mwolff44/pi-secured-setup/blob/0d531a4100e50318833c258e0a9609b903cda0a7/test/integration.test.ts)
registers real handlers on a mock ExtensionAPI.
Assertions inspect redaction, action blocks, pattern-only diagnostics, and
absence of verbatim attack/secret text in the audit.
This is useful composition evidence, not a real Pi or OpenCode provider capture.

ia-dev-tools already has permissions, restricted specialist agents, staged
gitleaks checks, deployment hashes, and delegation quality-control code.
Those are not interchangeable with a pre-provider filter.
Its `scripts/verify_agent_permissions.py` provides an additional useful pattern:
an isolated runtime configuration verifier with a sabotage mode that must
detect the deliberately removed deny. The script was read, not run.
Even that verifier checks resolved permissions, not every actual tool or network
effect.

## Candidate decisions: adopt, adapt, or defer

| Mechanism | Recommendation, not a policy ruling | Benefit | Cost and residual risk |
| --- | --- | --- | --- |
| Request-boundary secret transformation | Adapt Pi's boundary concept to proven OpenCode seams | Covers secrets in otherwise allowed content | Must prove all selected calls and failure paths; replacement can alter useful code. |
| Layered patterns, structured keys, and JSON-in-string handling | Adapt Scrubber concepts | Better coverage than a bare recursive regex walk | Key-name drift, parser/escaping complexity, identifier false positives. |
| Silent depth-limit passthrough | Do not inherit silently | Makes coverage gaps observable | Whether to refuse, ask, or warn is a live risk decision. |
| Native-authority baseline | Prefer OpenCode's native controls when verified | Avoids a second configuration-permission engine | Version/precedence proof and minimal invariant selection are necessary. |
| Pattern/count-only diagnostics | Adopt as a design constraint | Reduces log amplification and secondary disclosure | Metadata and paths can still reveal context; retention also needs a decision. |
| Injection marking | Experimental, narrow, advisory unless an action gate is proved | Recognizes known suspicious forms | Evasion, benign examples, carrier spoofing, and context/model degradation. |
| Bash parser | Reuse scenarios, not a second parser by default | Tests compound-action hazards | Duplicate parsers diverge; native scanner already exists. |
| Approval capability and technical call caps | Evaluate separately | Bounds unattended actions and runaway work | Web approval is not TUI; caps can stop valid investigations. |
| Skill hash/status display | Reuse drift visibility only if needed | Shows changed context | A recorded Deny that does not remove the Skill is not enforcement. |
| HMAC audit | Defer pending a concrete requirement | Detects some local edits | Key and writer in the same OS trust domain are not independent protection. |
| Broad PII defaults | Do not inherit for the initial scope | Preserves coding context | Paths, URLs, names, and IPs may be diagnostically necessary. |
| Rehydration/session maps | Keep outside the current destination | Avoids extra secret-bearing state and tool-output rewriting | This is a scope choice, not a claim that Scrubber lacks encrypted storage. |
| Full proxy | Keep as an architectural alternative, not a selected module | May provide a different enforcement boundary | Protocol coverage and fail-open defaults must be evaluated; not automatically excluded if plugin seams fail. |

## Storage correction

Scrubber's threat-model prose is stale about encryption.
The parent directly read
[storage.ts:93-165](https://github.com/Nano-Collective/prompt-scrubber/blob/8eee448af8cf9f2ac6bcbacc81dc40a6c7815285/src/session/storage.ts#L93-L165):
encrypted sessions are supported; plaintext is possible when encryption is not
enabled, and an already encrypted session is not silently downgraded.
Do not reject rehydration on the false premise that all maps are necessarily
plaintext. It remains outside this map's current scope for complexity and
data-minimization reasons.

## Prioritized synthetic scenario corpus

Construct synthetic credential-shaped values at test runtime; do not use real
credentials or private prompts.

1. **Outbound boundary:** vendor-prefix variants and overlength values;
   structured opaque passwords; JSON-in-string arguments; complete multiline
   private-key blocks; credential URLs; every auxiliary request and retry;
   unsupported encodings and deep nesting.
2. **Action boundary:** hostile tool/web/repository content requesting a harmless
   sentinel action; configured deny versus hook modification; absent approval
   client; policy weakening by a repository; malicious plugin discovery treated
   as crossing the trusted-runtime assumption.
3. **Injection signals:** direct role override, fake role carriers, multiline
   and comment-hidden instructions, benign security documentation, ordinary
   user instructions, and nested JSON fields named `system`.
4. **Shell negatives and adversarial forms:** substitutions, pipes in both
   directions, newlines, heredocs, brace expansion, and unknown commands.
   Test the actual native gate rather than assuming Pi's expected verdicts.
5. **Context preservation and diagnostics:** benign identifiers, hashes,
   connection examples without credentials, file paths and URLs; no raw
   synthetic secret or attacker text in logs, notices, or error messages.
6. **Robustness:** scanner budgets, oversized/deep payloads, malformed JSON,
   hook ordering, concurrent sessions, and failure injection.

## License and reuse constraints

Directly verified:

- [Pi LICENSE](https://github.com/mwolff44/pi-secured-setup/blob/0d531a4100e50318833c258e0a9609b903cda0a7/LICENSE): MIT, Mathias WOLFF.
- [Scrubber LICENSE](https://github.com/Nano-Collective/prompt-scrubber/blob/8eee448af8cf9f2ac6bcbacc81dc40a6c7815285/LICENSE): MIT, Nano Collective.
- This repository contains a GPL-3.0 license.
- ia-dev-tools has a proprietary all-rights-reserved license.

MIT source reuse requires preserving applicable notices and attribution.
Do not assume authorization to redistribute ia-dev-tools source merely because
it is locally readable; conceptual adaptation is distinct from code copying,
and the owner can make an explicit licensing decision if needed.
This is not legal advice and does not settle the future package/dependency
licensing decisions.

## Uncertainty and handoff

This research does not prove OpenCode compatibility or model resistance.
OpenCode interception is addressed by the sibling research ticket.
The next live decisions must choose refusal versus masking, incomplete-scan
behavior, provenance boundaries, injection-signal use, and acceptable context
damage. Those questions are already covered by the map's policy, architecture,
and acceptance tickets.

The reading was targeted, not a complete audit of every file or detector.
Read test assertions support statements about intended behavior, not passing
test results. No coverage percentage is claimed.

## Archive context

Local branch: `research/reusable-mechanisms`.
Setup baseline: `4c51cd0`.
The issue resolution is canonical; this file is its local research archive.
[Canonical resolution comment](https://github.com/mwolff44/opencode-security/issues/3#issuecomment-6947099964).
