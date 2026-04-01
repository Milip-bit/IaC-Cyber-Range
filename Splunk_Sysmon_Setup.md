# Manual Setup Guide: Splunk Universal Forwarder & Sysmon

Due to Windows WinRM connectivity dropping when the firewall is manipulated during automated Vagrant provisioning, the Splunk Universal Forwarder and Sysmon must be installed and configured manually on the `victim-pc`.

Follow this guide after running `vagrant up` to successfully route Sysmon telemetry to your Splunk SOC server.

## 1. Prerequisites

- Ensure your `soc-server` (Ubuntu) and `victim-pc` (Windows 10) are both running via VirtualBox.
- Log into the Windows 10 VM via the VirtualBox Graphical User Interface.
- Open the Start menu, search for `cmd`, right-click **Command Prompt**, and select **Run as Administrator**.

## 2. Install Packages

The `Vagrantfile` pre-installs the Chocolatey package manager. Use it to install Sysmon and the Splunk Universal Forwarder.

```cmd
choco install sysmon splunk-universalforwarder -y
```

## 3. Apply Sysmon Configuration

We use a comprehensive Sysmon configuration file (`sysmonconfig.xml`) that captures Process Creation (ID 1), Network Connections (ID 3), DNS Queries (ID 22), etc. This file is automatically mapped to `C:\vagrant` by Vagrant.

Apply the configuration and accept the EULA:

```cmd
C:\ProgramData\chocolatey\lib\sysmon\tools\Sysmon64.exe -accepteula -i C:\vagrant\sysmonconfig.xml
```

## 4. Configure Splunk Universal Forwarder

By default, Splunk doesn't know what to listen to or where to send its logs.

### A. Define Inputs

Tell Splunk to read the deeply nested Sysmon Operational Event Log channel and forward it as XML.
_Note: Do not use the `[XmlWinEventLog://...]` stanza name alias; use standard `[WinEventLog://...]` combined with `renderXml = true`._

Run the following commands strictly in the Administrator Command Prompt:

```cmd
mkdir "C:\Program Files\SplunkUniversalForwarder\etc\apps\SplunkUniversalForwarder\local"

echo [WinEventLog://Microsoft-Windows-Sysmon/Operational] > "C:\Program Files\SplunkUniversalForwarder\etc\apps\SplunkUniversalForwarder\local\inputs.conf"
echo disabled = false >> "C:\Program Files\SplunkUniversalForwarder\etc\apps\SplunkUniversalForwarder\local\inputs.conf"
echo renderXml = true >> "C:\Program Files\SplunkUniversalForwarder\etc\apps\SplunkUniversalForwarder\local\inputs.conf"
echo index = main >> "C:\Program Files\SplunkUniversalForwarder\etc\apps\SplunkUniversalForwarder\local\inputs.conf"
```

### B. Define Outputs

Tell Splunk to forward the logs to the `soc-server` IP (`10.0.1.10`) on the default listening port (`9997`).

```cmd
echo [tcpout] > "C:\Program Files\SplunkUniversalForwarder\etc\apps\SplunkUniversalForwarder\local\outputs.conf"
echo defaultGroup = default-autolb-group >> "C:\Program Files\SplunkUniversalForwarder\etc\apps\SplunkUniversalForwarder\local\outputs.conf"
echo [tcpout:default-autolb-group] >> "C:\Program Files\SplunkUniversalForwarder\etc\apps\SplunkUniversalForwarder\local\outputs.conf"
echo server = 10.0.1.10:9997 >> "C:\Program Files\SplunkUniversalForwarder\etc\apps\SplunkUniversalForwarder\local\outputs.conf"
```

## 5. Fix Service Permissions (Critical)

By default, the Chocolatey Splunk deployment installs the service under the virtual `NT SERVICE\SplunkForwarder` account. This account lacks the native permissions required to read the `Microsoft-Windows-Sysmon/Operational` event log channel, silently failing with an `Access Denied` (Error Code 5).

Change the service to run as `LocalSystem` using the Service Control (`sc.exe`) utility, and restart it to apply the configuration.

```cmd
sc.exe config SplunkForwarder obj= LocalSystem
net stop SplunkForwarder
net start SplunkForwarder
```

## 6. Verify Log Ingestion

Generate some network noise on the `victim-pc` (e.g., browse the web or run ping commands).

Then, verify the logs are arriving at the SOC server:

1. Open your host machine browser and go to `http://localhost:8000`.
2. Login with `admin` : `CyberLab123!`
3. In Search & Reporting, run the following SPL query (which uses `spath` to automatically unpack the raw XML structure on the fly):
   ```spl
   index="main" | spath | search "Event.System.EventID"=1 OR "Event.System.EventID"=3 OR "Event.System.EventID"=22
   ```
4. Verify you see Process Creation (1), Network Connection (3), and DNS Resolution (22) log events natively parsing.
