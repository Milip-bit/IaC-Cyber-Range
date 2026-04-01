# Splunk SOC Server Configuration (Native XML Parsing)

If you are deploying a fresh Splunk Server and receiving Sysmon logs directly from the Splunk Universal Forwarder running on a Windows endpoint, the logs will arrive in Splunk as raw XML strings.

By default, Splunk does not expand these nested XML nodes (like `<EventID>`, `<CommandLine>`, or `<Image>`), meaning you would have to append `| spath` to every single query.

To fix this natively during the search phase, you can apply search-time extraction rules directly by modifying the `props.conf` file.

## 1. Edit Configuration (`props.conf`)

You need to tell Splunk how to handle the specific Windows Event Log source/sourcetype.

1. SSH into your Splunk server (or connect to the bash terminal of your Docker container).
2. Open or create the `props.conf` file in the main Search application's local directory:

```bash
sudo nano /opt/splunk/etc/apps/search/local/props.conf
```

_(Note: If the `local` directory doesn't exist, create it using `mkdir -p /opt/splunk/etc/apps/search/local`)_

3. Paste the following configuration blocks into the file:

```ini
[XmlWinEventLog:Microsoft-Windows-Sysmon/Operational]
KV_MODE = xml

[source::WinEventLog:Microsoft-Windows-Sysmon/Operational]
KV_MODE = xml
```

### Why Both Stanzas?

- The first stanza explicitly binds the XML mode to the generic Splunk **sourcetype** assigned to forwarded event logs.
- The second stanza explicitly binds the XML mode directly to the raw **host source path**. Splunk parsing order often prioritizes the literal source over the sourcetype depending on how the Universal Forwarder tagged the payload. Including both ensures maximum compatibility without requiring the Splunk Windows Add-on.

## 2. Apply the Changes

For the extraction rules to take effect, the core Splunk daemon (`splunkd`) must be restarted.

Run the following command to reboot the service:

```bash
sudo /opt/splunk/bin/splunk restart
```

## 3. Verify Parsing

Once Splunk restarts, log back into Splunk Web (`http://<server-ip>:8000`).

Run a basic un-filtered search on your index:

```spl
index="main"
```

Look at the **"Interesting Fields"** sidebar on the left. You should now see all nested XML fields automatically extracted, parsed, and populated instantly (e.g., `Event.System.EventID`, `Event.EventData.Data{@Name}`).

You can now natively search and filter against Sysmon events without utilizing the `spath` command!
