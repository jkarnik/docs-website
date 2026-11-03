# Progress: prepopulate New Relic Network Agent names

Target: `kentik-ktranslate-catalog-tagged.html` (no git; backup at `/tmp/nrna-work/catalog-tagged.orig.html`)
Source of truth for new names: https://github.com/newrelic-forks/newrelic-network-agent (HEAD 01b4f0d, 2026-10-13)

## Naming facts from the repo
- Product: "New Relic Network Agent"; short prose form "network agent"
- Binary / systemd unit / container user / collector.name: `network-agent`
- Docker image: `newrelic/network-agent` (Docker Hub, tags `latest` + SemVer; no quay.io mirror)
- Config dir: `/etc/network-agent/` (profiles, mibs.db, config.json)
- Env vars: `NETWORK_AGENT_*` (e.g. `NETWORK_AGENT_MERAKI_API_KEY`, `NETWORK_AGENT_PING_PRIV`); `KENTIK_*` kept as deprecated aliases
- Log prefix: `NetworkAgent` ("NetworkAgent Running -- …", "NetworkAgent CLI: …", "NetworkAgent>device …")
- SNMP profiles: `github.com/newrelic-forks/snmp-profiles` (folder `profiles/kentik_snmp` unchanged)
- Unchanged: `kentik.*` metrics, `instrumentation.provider=kentik`, Kentik API flags, upstream wiki links (new repo has no wiki)

## Steps
- [x] 1. Research new agent naming
- [x] 2. Extract every name-field key/occurrence from DATA — 475 fields (`/tmp/nrna-work/extract.js`)
- [x] 3. Build name proposals (rules + per-mention overrides) — `/tmp/nrna-work/propose.js` + `overrides.js` → `default-names.json` (475), `default-comments.json` (48)
- [x] 4. Embed proposals in the HTML as defaults (user edits still win) — `DEFAULT_NAMES`/`DEFAULT_COMMENTS` after `DATA`, `valueFor()` fallback, merged export, intro text, auto-height comment boxes
- [x] 5. Verify — headless Chrome: 475/475 fields + 48/48 comments match; typed and emptied values win; export merges

Status: complete (2026-10-13).
