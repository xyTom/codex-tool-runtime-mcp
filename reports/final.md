# Current Integration Report

This report summarizes the recovered rollout and subsequent production-hardening checkpoints on branch `recover-rollout-2026-05-16`.

## Remote Checkpoint

- GitHub repo: `https://github.com/ytagent/codex-tool-runtime-mcp`
- Branch: `recover-rollout-2026-05-16`
- Latest pushed commit: `687509f7588c8b215b91b0af727052d7341c3c0b`
- Latest checkpoint tag: `v0.1.3-recovered-error-normalization`
- Latest GitHub Actions run: `https://github.com/ytagent/codex-tool-runtime-mcp/actions/runs/25993105983`
- CI conclusion: `success`

## Current Evidence

- `make lint`: passed.
- `make typecheck`: passed.
- `make test`: passed, `65` tests.
- `make ci`: passed locally with lint, typecheck, unittest discovery, protocol/integration gates, docs/schema gates, dogfood smoke, latency benchmark, and SWE-bench preflight.
- `make compliance`: passed, `65` tests, `suite: all`.
- GitHub Actions on the pushed SHA passed lint, typecheck, unit discovery, protocol, integration, docs, schema drift, dogfood smoke, latency benchmark, SWE-bench preflight, full compliance refresh, and artifact upload.
- Deterministic Codex-on-MCP dogfood conclusion: `PASS`, with direct bypass `False`.
- MCP runtime latency benchmark conclusion: `PASS`.
- SWE-bench smoke conclusion: `PREFLIGHT_ONLY`.
- Explicit official SWE-bench attempt conclusion: `BLOCKED`.

## Primary Artifacts

- Compliance report: [compliance/latest.md](compliance/latest.md)
- Dogfood report: [dogfood/codex-on-mcp.md](dogfood/codex-on-mcp.md)
- Dogfood transcript: [../docs/dogfood/codex-on-mcp-transcript.json](../docs/dogfood/codex-on-mcp-transcript.json)
- MCP latency benchmark: [benchmark/mcp-latency.md](benchmark/mcp-latency.md)
- SWE-bench preflight: [benchmark/swebench-regression.md](benchmark/swebench-regression.md)
- SWE-bench official attempt: [benchmark/swebench-official-attempt.md](benchmark/swebench-official-attempt.md)
- SWE-bench official raw logs: [benchmark/swebench-official-attempt/raw](benchmark/swebench-official-attempt/raw)

## Implemented Improvements

- Recovered the prior runtime hardening rollout and regenerated subagent/report artifacts.
- Added JSON-RPC envelope validation, params-object validation, initialize protocol-version validation, initialized-state enforcement, stdio session continuity, cancellation notifications, and structured redacted trace logging.
- Hardened HTTP transport handling with media-type validation, bounded request bodies, valid `Content-Length` checks, batch-size caps, exact loopback Origin validation, unknown session-id rejection, and normalized `id: null` pre-dispatch errors.
- Strengthened execution security with shell-expansion gating, setuid/setgid executable rejection, secret-value environment filtering, Linux Landlock filesystem confinement, direct syscall denial coverage, cancellation/kill cleanup, watchdog-backed session deadlines, bounded session buffers, and reader-thread drain on process exit.
- Added adversarial protocol/security tests for malformed envelopes, invalid params, pre-initialize calls, stdio pre-initialize rejection, cancellation notifications, content-type/body limits, batch limits, Origin prefix bypasses, unknown sessions, and pre-dispatch error IDs.
- Added an MCP runtime latency benchmark comparing MCP HTTP calls to native local developer-tool baselines.
- Updated GitHub Actions to Node 24-backed action releases and verified the deprecation warning is gone.
- Expanded CI to run lint, typecheck, unittest discovery, protocol/integration gates, docs-required, schema-drift, dogfood smoke, latency benchmark, SWE-bench preflight, full compliance refresh, and artifact upload.

## SWE-bench Status

The official harness was genuinely attempted with:

```bash
python benchmarks/swebench/run_smoke.py --install-swebench --run-evaluation --allow-placeholder-evaluation --instance-id sympy__sympy-12419 --max-workers 1 --report-json reports/benchmark/swebench-official-attempt.json --report-md reports/benchmark/swebench-official-attempt.md
```

Result: `BLOCKED`.

Observed blockers:

- Docker executable is not available in this container.
- `pip install swebench` completed, but `python -m swebench.harness.run_evaluation --help` failed due a `pyarrow` API mismatch in the installed dependency set.
- Prediction files remain placeholders, so they cannot support score claims.

## Remaining Risks And Requirements

- The full 7-hour productionization objective is still active; this report is a verified checkpoint, not a claim that the time-boxed final goal is complete.
- Do not claim SWE-bench pass until Docker-backed official harness results exist with real baseline and MCP-candidate predictions and parsed resolved counts.
- A future production deployment should still add rate limiting, stronger process isolation on non-Linux platforms, and persistent trend storage for latency/benchmark reports.
