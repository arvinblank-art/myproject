# OpenClash and Shadowrocket production rules

These rules are mandatory for every OpenClash, Shadowrocket, DNS, Lucky, or routing task in this repository.

## Change discipline

1. Inspect live state read-only before proposing or making a change. Do not infer root cause from a browser error alone.
2. Trace the complete path when a hostname fails: client DNS answer -> AdGuard rewrite (domain, exact answer, enabled) -> active OpenClash YAML/core -> Lucky Host/SNI route -> backend health.
3. Record the observed failure and the hypothesis before editing. A hypothesis is not evidence; if the check disproves it, stop and update the diagnosis rather than adding a second speculative change.
4. Make one minimal, reversible change at a time. Never stack speculative DNS, Fake-IP, routing, proxy-group, certificate, or backend edits.
5. Do not switch the active OpenClash configuration, change global Fake-IP/DNS mode, or change proxy groups to repair one hostname unless the user explicitly approves the broader impact and a rollback plan exists.
6. Never add a `DIRECT` rule to a hostname that currently resolves to Fake-IP. First prove that the hostname returns its intended real LAN IP; otherwise the direct connection can be sent to the Fake-IP range and fail.
7. Before every production edit, create a versioned backup of the exact target. Preserve active traffic and unrelated services.
8. After every production edit, reload only the affected service and prove the requested endpoint works. Also regression-test Zashboard and any affected existing service.
9. A LuCI “saved” message, a YAML parse result, a configuration file on disk, or a screenshot is not completion. Confirm the running process loaded the intended configuration and verify end-to-end behavior from a real LAN client.
10. If an agent-added change worsens or plausibly causes the issue, back it out first from its versioned backup before making another change.
11. If browser automation cannot complete a state-changing click or is denied, do not bypass it through alternate browser/control paths. State exactly what remains unapplied and ask the user to perform that visible action; then verify the result independently.
12. Do not delete, overwrite, rename, or prune production configuration files as part of diagnosis. Cleanup is a separate release step after successful verification.

## OpenClash and Shadowrocket release rules

1. `D:\workspace\myproject` is the sole Git working baseline. Do not use other folders as competing sources of truth.
2. Before applying production changes: create candidate -> validate syntax/rule targets -> review delta -> commit and push Git -> upload/apply -> runtime verification.
3. Version names must be unique and explicit: `openclash-v<N>-<purpose>-YYYYMMDD.yaml` and `shadowrocket-v<N>-<purpose>-YYYYMMDD.conf`.
4. Keep exactly two live router configurations: the active version and immediately previous version. Keep older history in Git, not in the router configuration list.
5. Never reuse a version filename for different content. Never delete the previous version until the new version has passed all runtime checks.
6. Shadowrocket is separately generated but must match the approved OpenClash routing intent where supported.
7. Public Git files must not contain subscriptions, node endpoints, credentials, tokens, private host aliases, or personal identifiers.
8. Do not deploy a candidate merely because it imports or parses. Verify active core status, expected selectors/rules in Zashboard, target-domain behavior, and unaffected critical services before marking it released.
9. Do not silently sync a live router change into Git after the fact. Git must contain the reviewed candidate and backup/reference before the live apply.

## Internal HTTPS hostnames

1. For `*.802.viggosimple.top` services behind Lucky, verify the live AdGuard rewrite target is a valid LAN IP before modifying OpenClash.
2. Normally Lucky-facing services resolve to `192.168.50.2`; validate actual DNS and HTTPS Host/SNI routing after any change.
3. Do not change global Fake-IP mode merely to repair one internal hostname without explicit user approval and a rollback plan.
4. A malformed AdGuard rewrite answer can mimic an OpenClash/Fake-IP failure. Check the literal persisted `answer` value, not just that a rewrite row exists in the UI.
5. Validate both the intended hostname and at least one existing Lucky service after DNS, certificate, or reverse-proxy work. An HTTP `401` for an authenticated service is a healthy reachability result, not a proxy failure.
6. Before editing a container-backed service for HTTPS, back up its compose/configuration, validate the compose file, change only its canonical external URL, redeploy only the affected container, and test through the final hostname with Host/SNI. Do not treat a direct backend port test as proof of the HTTPS route.

## Router and system safety

1. Do not perform firmware upgrades, WAN/VLAN changes, dnsmasq port changes, or firewall changes during an application/DNS diagnosis. These are separate changes requiring their own confirmed rollback plan.
2. For router DNS changes, retain a working emergency DNS path and verify a real DHCP client after service restart.
3. Do not use a direct NAS address as a substitute for a Lucky hostname when certificate or Host/SNI routing matters.
4. Do not expose internal services to WAN merely to simplify testing. Preserve the LAN/WireGuard access posture unless the user explicitly asks for exposure and firewall/TLS review is complete.
