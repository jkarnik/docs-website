# Engineering Inputs

## Reorder the migration steps
The doc says to edit snmp-base.yaml first, then pull the image, then stop the old container. The order should be:

1. Stop the old container.
2. Pull the new image.
3. Copy snmp-base.yaml.
4. Start the new container.

Why the old container must be stopped first:

The old and new containers use the same snmp-base.yaml file. While the old container runs, its discovery job can rewrite the file and undo the edit. If the old container restarts after the edit, it can’t find the new paths and keeps restarting.


## Add a section for container logs
The doc has no section about the agent’s own logs. The new section should say:

1. The logs are in docker logs network-agent-$CONTAINER_SERVICE.
2. The logs also go to New Relic Logs when the container runs with -tee_logs=true.
3. Log lines now start with network-agent/<msg> instead of ktranslate/<msg>.
4. The message prefix is now NetworkAgent instead of KTranslate.
5. collector.name changes to network-agent.
6. logtype and plugin.type stay ktranslate-health, so existing filters on them keep working.


## Other Corrections

### Add one line to the device check query
Every agent sends a heartbeat metric about itself, and its device_name is the container’s hostname. On Docker’s default network, that hostname is the container ID, so it changes when the container is re-created. If a customer removes the tags.container_service line, the query counts these heartbeats too, and the old container’s ID shows up as a “missing device”. Adding AND provider != 'kentik-agent' removes the heartbeats, so only real devices are checked.

### Search for shorter text in the startup-line query
Change message LIKE '%Running -- version%' to message LIKE '%Running --%'. The KTranslate startup line says “Version” with a capital V, and the Network Agent line says “version” with a small v. Both lines contain “Running --”, so the shorter text finds both, whether or not LIKE is case-sensitive.

### /etc/ktranslate paths in docker commands
If a customer’s docker run command has a path such as -mapping /etc/ktranslate/config.json, it must change to /etc/network-agent.