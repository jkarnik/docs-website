# Engineering Inputs

 1: Reorder the migration steps
The doc says to edit snmp-base.yaml first, then pull the image, then stop the old container. The order should be:

Stop the old container.
Pull the new image.
Edit snmp-base.yaml.
Start the new container.
Why the old container must be stopped first:

The old and new containers use the same snmp-base.yaml file.
While the old container runs, its discovery job can rewrite the file and undo the edit.
If the old container restarts after the edit, it can’t find the new paths and keeps restarting.


 2: Add a section for container logs
The doc has no section about the agent’s own logs. The new section should say:

The logs are in docker logs network-agent-$CONTAINER_SERVICE.
The logs also go to New Relic Logs when the container runs with -tee_logs=true.
Log lines now start with network-agent/<msg> instead of ktranslate/<msg>.
The message prefix is now NetworkAgent instead of KTranslate.
collector.name changes to network-agent.
logtype and plugin.type stay ktranslate-health, so existing filters on them keep working.


Corrections

 1: Add one line to the device check query
Every agent sends a heartbeat metric about itself, and its device_name is the container’s hostname. On Docker’s default network, that hostname is the container ID, so it changes when the container is re-created. If a customer removes the tags.container_service line, the query counts these heartbeats too, and the old container’s ID shows up as a “missing device”. Adding AND provider != 'kentik-agent' removes the heartbeats, so only real devices are checked.

2: Search for shorter text in the startup-line query
Change message LIKE '%Running -- version%' to message LIKE '%Running --%'. The KTranslate startup line says “Version” with a capital V, and the Network Agent line says “version” with a small v. Both lines contain “Running --”, so the shorter text finds both, whether or not LIKE is case-sensitive.

3. /etc/ktranslate paths in docker commands: if a customer’s docker run command has a path such as -mapping /etc/ktranslate/config.json, it must change to /etc/network-agent.
Ravi Kiran Noothi  [7:58 AM]
@sbankuru
1: Reorder the migration steps
should we just copy and edit the snmp base yaml, rather edit ?

2: Add a section for container logs, 
why is this required ?

what does this do ?
-mapping /etc/ktranslate/config.jsonSiva Kumar Bankuru  [11:05 AM]

Yes copying is better, still stop the old container before starting the new one.
If the customer enable container logs the message will format will change. 
It tells the agent where to find a lookup table for flow data. Flow data such as NetFlow or IPFIX labels many fields with numbers instead of names.The image already sets it in its built-in startup command, with the new path: -mapping /etc/network-agent/config.json. It only matters if a customer added it to their own command while running the container.
       example- > https://newrelic.atlassian.net/wiki/spaces/NNM/pages/4737957949/KTranslate+-+Troubleshooting+steps#5.-Running-ktranslate-with-debug-logs