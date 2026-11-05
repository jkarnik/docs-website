:announcement: **kTranslate Network Monitoring Agent End-of-Life,** effective **[DEADLINE DATE]** :announcement:

Effective **[DEADLINE DATE]**, kTranslate (`kentik/ktranslate`), the agent that collects data for New Relic network monitoring (NPM), reaches end of life (EOL). New Relic now builds, publishes, and supports its replacement, the **New Relic Network Agent**, available on Docker Hub from **[ANNOUNCEMENT DATE]**. Because the new agent is built from kTranslate and reads the same configuration files, customers migrate by swapping the image, updating two paths, and re-creating each container. From launch, fixes, security updates, and improvements ship only in the New Relic Network Agent.

**What's changing?**
- **Image:** `kentik/ktranslate:v2` becomes `newrelic/network-agent:latest`, on Docker Hub only (no Quay.io mirror).
- **Installation methods:** Docker only. Podman and Linux package (yum/apt) installs aren't supported, so those customers move to Docker.
- **Paths:** `/etc/ktranslate` becomes `/etc/network-agent`: the two paths in `snmp-base.yaml`, plus any custom mounts or flags that use it.
- **Agent logs and health data:** `collector.name` changes from `ktranslate` to `network-agent`, and log messages start with `NetworkAgent` instead of `KTranslate`.

**User impact:**
- Every customer running kTranslate for SNMP polling, SNMP traps, network flow, syslog, or Meraki Dashboard API polling must migrate each container. A container stops sending data for a minute or two during the switch.
- Queries, alerts, and dashboards that monitor the agent itself, such as filters on `collector.name = 'ktranslate'`, need updating.
- New deployments should start on the New Relic Network Agent from **[ANNOUNCEMENT DATE]**.

**Not impacted:**
- Device entities, their history, and metric names (`kentik.*`). Dashboards and alerts built on device data keep working.
- License key, account ID, region, environment variables, `docker run` flags, and the device list in `snmp-base.yaml` all carry over.
- Cloud flow log monitoring for AWS, Azure, and GCP doesn't use kTranslate, so no action is needed.

**Action plan to transition** (for each container; the migration guide has the commands):
1. Pull `newrelic/network-agent:latest`.
2. Stop the kTranslate container.
3. Update the two paths in `snmp-base.yaml`. Every container type needs this, or the new container keeps restarting.
4. Start the new container with the original `docker run` command and the new image.
5. Check that it reports every device, then remove the old container.

The guide also covers moving from Podman or a Linux package to Docker, and has a query that shows which containers still run kTranslate.

**What happens if no action is taken:** After **[DEADLINE DATE]**, New Relic supports only deployments that use the `newrelic/network-agent` image. Customers still on kTranslate run an unsupported agent with no fixes or security updates, and can't use the capabilities below.

**What's next:** New Relic is building tools to manage network agents from the New Relic UI: discover devices and choose what to monitor, spread monitoring across containers automatically, and see the health of every agent in one place. These will work only with the New Relic Network Agent. _Roadmap with no committed dates, so please don't share timelines with customers._ [Network Agent Orchestration & Deployment](https://newrelic.atlassian.net/wiki/spaces/NNM/pages/6208651986/Network+Agent+Orchestration+Deployment)

A similar announcement will go out internally via RevReport (GTM) and ShiftNotes (GTS), and externally via direct customer emails and public Docs on **[ANNOUNCEMENT DATE]**. For more information, see the EOL notice: [EOL NOTICE LINK]

**Questions?** Reply to this thread or reach out to @Jay Karnik/PM and @Ravi Kiran Noothi/EM. Support cases go to #gts-npm-case-assistance.

**Additional resources:**
- [Migration guide: kTranslate to the New Relic Network Agent](https://docs.newrelic.com/docs/network-performance-monitoring/get-started/migrate-to-network-agent/)
- [New Relic Network Agent on Docker Hub](https://hub.docker.com/r/newrelic/network-agent)
