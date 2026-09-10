# Fleet safety retrofit

This tool audits recognized generated-source patterns without printing CLIs or
changing catalog indexes. Run it from the repository root:

```sh
GO111MODULE=off go run ./tools/fleet-safety-audit
GO111MODULE=off go test ./tools/fleet-safety-audit
```

The audit exits nonzero when it finds a candidate. `-verbose` lists candidates;
`-write` applies the mechanical repairs and creates per-CLI reprint guards.
Inspect the candidate diff before committing. A zero result covers the patterns
this tool recognizes, not every possible bespoke implementation.

The retry repair preserves the existing retry budget for authentication refresh
and rate-limit responses. Only transport errors and server errors are restricted
to GET, HEAD, OPTIONS, or explicit read-only intent. Existing guarded retry
implementations and fixed-method helpers are left alone. Soccer Goat's separate
cross-source failover boundary requires a manual repair and executable tests.

The other repairs encode path parameters (including dot-only segments), remove
constant-true parameter guards, and report zero stored rows when a batch rolls
back. They do not advance sync watermarks or change MCP tool registration.

For a staged candidate, independently replay every mechanical source edit from
the review base and require a tracked reprint guard for every changed source:

```sh
FLEET_AUDIT_BASE=upstream/main GO111MODULE=off go test ./tools/fleet-safety-audit -count=1 -v
git diff --cached --check
```

## Issue 1977 integration

The retrofit was applied directly to upstream/main
`ce7f84011f58333dc2a89ba3a9b382dcf13ef696`. It does not carry the old branch's
library diff. This base produced 361 mechanical retry-client changes, 265 path
helpers, 27 parameter files, and 346 stores. Soccer Goat adds one manually
reviewed client change. FedEx and Paperclip's already-safe retry policies remain
unchanged. Paperclip's three requested guard records are tracked explicitly.

Representative checks use local fixtures and temporary configuration directories,
not production credentials or provider requests. Amazon Ads tests exercise write
401 refresh, bounded 429 recovery, ambiguous write failures, read-only POST
recovery, and rollback counts. Soccer Goat tests cover cross-source replay.
The full fleet replay complements representative execution; it does not claim
that every CLI's test suite was executed.
