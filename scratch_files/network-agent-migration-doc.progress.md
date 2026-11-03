# Progress: kTranslate → New Relic Network Agent migration page

Target: `docs-website/src/content/docs/network-performance-monitoring/get-started/migrate-to-network-agent.mdx` (branch `npm/network-agent-migration-guide`, not committed)
Approved outline: `.superpowers/brainstorm/18229-1792677713/content/outline.html` + `docker-steps-v2.html`

## Decisions from the user
- Audience: existing NPM customers, sent here by an email / AE outreach about the networking overhaul.
- Lives on docs.newrelic.com; drafted in docs-site MDX format.
- Image `newrelic/network-agent:latest` (GA will publish `latest`); re-create containers to get updates; never promise auto-update.
- Podman and Linux package installs are not supported; those customers move to Docker. Quay.io option removed.
- Deadline: 6 months from announcement (placeholder). Don't mention the Kentik contract or how long old images stay hosted; urgency from the date + "updates only ship in the new image".
- Docker steps: stop old → start new (new name) → verify → remove old (rollback possible until removal; `--restart always` caveat).
- Non-goal: renaming kTranslate across the rest of the NPM docs.

## Steps
- [x] 1. Setup (branch, progress file)
- [x] 2. Research exact commands/facts for each section
  - Find containers: `docker ps -a --format … | grep ktranslate` (docs' `--filter ancestor=…:v2` misses pinned tags). Recover settings with `docker inspect` (.Config.Cmd, .Config.Env, .HostConfig.Binds/PortBindings/NetworkMode/RestartPolicy).
  - snmp-base.yaml: `mib_profile_dir`/`mibs_db` under `global:` → `/etc/network-agent/…` (old paths make SNMP init fail: leveldb read-only open + os.ReadDir). Custom profiles mount → `:/etc/network-agent/profiles`.
  - All docs flags still exist (incl. `-metrics=jchf`, modes nr1.snmp/flow/syslog); `-nr_region` lowercased (EU/GOV fine), EU/JP auto-detected from key.
  - Linux→Docker: `/etc/ktranslate/snmp-base.yaml`, `/etc/default/ktranslate.env` (KT_FLAGS); trap.listen 162→1620 (`-p 162:1620/udp`), `-syslog.source` 514→5143 (`-p 514:5143/udp`); container user can't bind <1024; `systemctl disable --now ktranslate`; `yum remove` / `apt-get remove`.
  - Entities unchanged (synthesis keys: provider kentik-*, instrumentation.provider=kentik, device_name). Old agent also sets collector.name (='ktranslate') on metrics; heartbeat provider still `kentik-agent` → tracking query works for both. Log prefix KTranslate→NetworkAgent ("NetworkAgent Running -- version X").
  - Internal (not for the page): built-in `ktranslate-health` parsing rule grok expects "ktranslate"; guided install still generates kentik commands.
- [x] 3. Skeleton: frontmatter, intro, deadline callout, headings
- [x] 4. What's changing
- [x] 5. Before you begin
- [x] 6. Migrate a Docker deployment: steps
- [x] 7. Migrate a Docker deployment: per-job commands
- [x] 8. Migrate a Docker deployment: pipelines, custom profiles, future updates
- [x] 9. Move from Podman or a Linux package
- [x] 10. Update queries, alerts, and dashboards
- [x] 11. Track your migration
- [x] 12. Troubleshooting
- [x] 13. Timeline + related docs
- [x] 14. Nav entry (`src/nav/network-performance-monitoring.yml`, after "Get started")
- [x] 15. Verify — `node scripts/verify_mdx.js <page>` passes (checker confirmed to catch a broken copy); nav YAML parses; local preview compiled with the repo's MDX (`/tmp/nrna-review/preview.cjs`)
- [x] 16. Review against the conversation's decisions — added "future fixes ship only in the new agent", rollback restart-policy note, crash-loop wording for all container types
- [x] 17. Wrap up — companion left running for the preview (auto-stops after 4h idle)

Status: draft complete (2026-10-22), uncommitted on branch `npm/network-agent-migration-guide`.

## Follow-up (2026-10-23): HTML reference instead of a repo change
User isn't editing the docs; they hand reference material to the docs writer. Nothing may live inside `docs-website`.
- [x] Saved draft to `/tmp/nrna-review/migrate-to-network-agent.mdx`; removed the page, reverted the nav, deleted the branch (`docs-website` clean on `develop`).
- [x] Links aligned with names (4).json renames (`network-agent-container-management`, `network-agent-container-health`).
- [x] Built `network-agent-migration-guide.html` (Documentation folder) via `/tmp/nrna-review/build-html.cjs`: writer notes (`/tmp/nrna-review/writer-notes.html`) + rendered draft + MDX source.
- [x] Verified: in-page links resolve, docs links absolute, light/dark render, no overflow at narrow width; companion server stopped.
- [x] Added Docker step 6 "reports every device the old one did" (NRQL by device_name, split by collector.name, filtered on tags.container_service; KFlow / Log variants); removal is now step 7; tip + Podman/Linux final steps point to it. MDX check passes; HTML rebuilt.
- [x] 2026-10-29: `/tmp` working files were cleared by macOS; recovered MDX + notes from the HTML (rebuild was byte-identical) into `.superpowers/network-agent-migration/` (with `build-html.cjs`). Rebuild: `node .superpowers/network-agent-migration/build-html.cjs docs-website <mdx> <notes> network-agent-migration-guide.html`.
- [x] 2026-11-03: REVERTED (user's prompts were questions, not change requests) — region note + Linux `NR_REGION` line, and the differences-only step 6 query + its writer note. Guide is back to the post-device-check version (58.3 KB).
- [x] 2026-11-03: Restored the "What's coming" roadmap callout + its writer note, added on 2026-10-29 by another session (b5deedb2) directly in the HTML and clobbered by the Nov 3 revert rebuild. Now in the sources (MDX + notes); rebuild verified byte-identical to "Nov 3 version + that session's 3 edits". `build-html.cjs` now refuses to overwrite an output changed since its last build (`last-build.json`; `--force` to override).
- [~] (reverted) Region: example-commands note covers `-nr_region` / `NR_REGION`, flag must precede the mode arg; Linux step carries over `NR_REGION` from the env file.
- [~] (reverted) Step 6 query now shows only missing devices (nested aggregation: inner per-device counts by agent, outer `WHERE old_points > 0 AND new_points = 0`); container name copied from tracking query; writer note added to test it on live data.
- [x] 2026-11-03: Engineering input (`scratch_files/Scratch.md`) reviewed against agent HEAD 01b4f0d + upstream kentik/ktranslate. Guide and progress file now live in `docs-website/scratch_files/` (user moved them; build guard re-keyed to that path). Applied: Docker steps reordered to pull → stop → edit paths → start (Podman: stop before edit; step-1 refs → step 3); step 6 query gets `AND provider != 'kentik-agent'` (Metric only; flow/syslog variants leave it out); tracking query `LIKE '%Running --%'`; `/etc/ktranslate` flag/mount paths → `/etc/network-agent` (step 4, What stays the same, example-commands intro, pipelines bullet, Linux bullet); new "Find the agent's logs" section before Update queries; writer notes updated. Pushed back: pull stays before stop (shorter gap); "discovery undoes the edit" reason is wrong (both agents re-read `global` before saving); log lines show `network-agent/<service_name>`, not `<msg>`; LIKE is already case-insensitive per NRQL docs. MDX check passes; HTML 65.4 KB.
- [x] 2026-11-03: User deleted the MDX source ("it will never be used. Make all changes to the HTML file only"). Removed `.superpowers/network-agent-migration/migrate-to-network-agent.mdx` after confirming the HTML embeds it byte for byte. From now on edit `network-agent-migration-guide.html` directly; `build-html.cjs`, `writer-notes.html`, and `last-build.json` are retired.
