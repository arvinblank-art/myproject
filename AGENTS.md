# OpenClash and Shadowrocket production rules

These rules are mandatory for every OpenClash, Shadowrocket, DNS, Lucky, or routing task in this repository.

## Change discipline

1. Inspect live state read-only before proposing or making a change. Do not infer root cause from a browser error alone.
2. Trace the complete path when a hostname fails: client DNS answer -> AdGuard rewrite (domain, exact answer, enabled) -> active OpenClash YAML/core -> Lucky Host/SNI route -> backend health.
3. Make one minimal, reversible change at a time. Never stack speculative DNS, Fake-IP, routing, or proxy-group edits.
4. Before every production edit, create a versioned backup of the exact target. Preserve active traffic and unrelated services.
5. After every production edit, reload only the affected service and prove the requested endpoint works. Also regression-test Zashboard and any affected existing service.
6. A LuCI “saved” message, a YAML parse result, or a screenshot is not completion. Confirm the running process loaded the intended configuration and verify end-to-end behavior.
7. If an agent-added change worsens or plausibly causes the issue, back it out first from its versioned backup before making another change.

## OpenClash and Shadowrocket release rules

1. `D:\workspace\myproject` is the sole Git working baseline. Do not use other folders as competing sources of truth.
2. Before applying production changes: create candidate -> validate syntax/rule targets -> review delta -> commit and push Git -> upload/apply -> runtime verification.
3. Version names must be unique and explicit: `openclash-v<N>-<purpose>-YYYYMMDD.yaml` and `shadowrocket-v<N>-<purpose>-YYYYMMDD.conf`.
4. Keep exactly two live router configurations: the active version and immediately previous version. Keep older history in Git, not in the router configuration list.
5. Never reuse a version filename for different content. Never delete the previous version until the new version has passed all runtime checks.
6. Shadowrocket is separately generated but must match the approved OpenClash routing intent where supported.
7. Public Git files must not contain subscriptions, node endpoints, credentials, tokens, private host aliases, or personal identifiers.

## Internal HTTPS hostnames

1. For `*.802.viggosimple.top` services behind Lucky, verify the live AdGuard rewrite target is a valid LAN IP before modifying OpenClash.
2. Normally Lucky-facing services resolve to `192.168.50.2`; validate actual DNS and HTTPS Host/SNI routing after any change.
3. Do not change global Fake-IP mode merely to repair one internal hostname without explicit user approval and a rollback plan.
