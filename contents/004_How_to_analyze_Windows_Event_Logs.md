## How to analyze Windows Event Logs

Windows operating system and its applications use Windows Event Logs as a centralized logging system. In the time of incidents, Windows Event Logs provide useful information for the incident investigator.

You can prepare .evtx through following steps:

1. Open the Event Viewer.
2. Locate the event channel to be exported.
3. Right click on event channel and select "Save All Events As..."
4. Enter a file name.
5. Save as a `Event Files (*.evtx)` file.

### How to check background network transfer

According to [MITRE ATT&CK T1197](https://attack.mitre.org/techniques/T1197/), adversaries may abuse BITS jobs to persistently execute or clean up after malicious payloads. Windows Background Intelligent Transfer Service (BITS) is a low-bandwidth, asynchronous file transfer mechanism exposed through Component Object Model (COM).

Then how can we enumerate BITS event logs and verify them? If you don't know what is the target event id, you should analyze message by event id. `first()` aggregation function provides first matched `msg` per `event_id` group.


```
logpresso> evtx-file Bits-Client.evtx | stats first(msg) as msg by event_id
{"msg":"The BITS service created a new job: {A6EADE66-0804-0000-1959-000000000000}, with owner {6b33678d-5527-41d1-a5e5-d2424e147491}","event_id":3}
{"msg":"The transfer job is complete.  User: HQ\\xeraph  Transfer job: Push Notification Platform Job: 1  Job ID: {801c2191-26c0-4a15-89e7-95dc87600825}  Owner: HQ\\xeraph  File count: 1","event_id":4}
{"msg":"Job cancelled. User: HQ\\xeraph, job: Edge Component Updater, jobID: {66781f31-c7f2-42a5-85ea-2d2bb5120ef3}, owner: HQ\\xeraph, filecount: 1","event_id":5}
{"msg":"BITS started the Push Notification Platform Job: 1 transfer job that is associated with the https://img-s-msn-com.akamaized.net/tenant/amp/entityid/BB1ev7Gj.img?w=204&h=100&m=6&tilesize=wide&ms-scale=100&ms-contrast=standard URL.","event_id":59}
{"msg":"BITS stopped transferring the Push Notification Platform Job: 1 transfer job that is associated with the https://img-s-msn-com.akamaized.net/tenant/amp/entityid/BB1ev7Gj.img?w=204&h=100&m=6&tilesize=wide&ms-scale=100&ms-contrast=standard URL. The status code is 0.","event_id":60}
{"msg":"BITS stopped transferring the C:\\Users\\xeraph.HQ\\AppData\\Local\\Temp\\{A0076D54-EB5F-478B-BAE0-0F5C95F2DF0D}-89.0.4389.90_89.0.4389.82_chrome_updater.exe transfer job that is associated with the http://edgedl.gvt1.com/edgedl/release2/chrome/AKYwteAwG4Wh7zNuivsoyTg_89.0.4389.90/89.0.4389.90_89.0.4389.82_chrome_updater.exe URL. The status code is 2147954402.","event_id":61}
{"msg":"High performance property for BITS job \"Push Notification Platform Job: 1\" with ID \"{d78ff8fc-2d80-4832-8392-3e56a994d8a8}\" 1.","event_id":209}
{"msg":"The BITS service loaded the job list from disk.","event_id":306}
{"msg":"The initialization of the peer helper modules failed with the following error:  2147942450.","event_id":310}
{"msg":null,"event_id":16403}
```

Event id 59 is for `BITS started` logs. Filter by `event_id == 59` and use regular expression to parse job name and URL.

```
logpresso> evtx-file Bits-Client.evtx \
.. | search event_id == 59 \
.. | rex field=msg "BITS started the (?<job>.*?) [J|j]ob.*? (?<url>https?://.*?) URL" \
.. | fields job, url
{"job":"Push Notification Platform","url":"https://img-s-msn-com.akamaized.net/tenant/amp/entityid/BB1ev7Gj.img?w=204&h=100&m=6&tilesize=wide&ms-scale=100&ms-contrast=standard"}
{"job":"{A6EADE66-0804-0000-1959-000000000000} transfer","url":"https://armmf.adobe.com/arm-manifests/win/ArmManifest3.msi"}
{"job":"Push Notification Platform","url":"https://img-s-msn-com.akamaized.net/tenant/amp/entityid/BB1evAMx.img?w=204&h=100&m=6&tilesize=wide&ms-scale=100&ms-contrast=standard"}
...
```

Now, you can exclude trusted domains and focus rare URLs.

```
logpresso> evtx-file Bits-Client.evtx \
.. | search event_id == 59 \
.. | rex field=msg "BITS started the (?<job>.*?) [J|j]ob.*? (?<url>https?://.*?) URL" \
.. | fields job, url \
.. | search not(in(url, "*.microsoft.com/*", "*ardownload.adobe.com/*", "*armmf.adobe.com/*", "*.googleapis.com/*", "*.gvt1.com/*", "*img-s-msn-com.akamaized.net/*"))
{"job":"C:\\Users\\xeraph.HQ\\AppData\\Local\\Temp\\{01D3B6B2-2973-4793-871A-73C3959515FF}-DropboxClient_119.4.1772.exe transfer","url":"https://edge.dropboxstatic.com/dbx-releng/client/DropboxClient_119.4.1772.exe"}
```

### How to summarize inbound RDP connections

`Microsoft-Windows-TerminalServices-RemoteConnectionManager/Operational` event channel provides remote desktop connection events.

As before, you need to check message format per event id.

```
logpresso> evtx-file TerminalServices-RemoteConnectionManager.evtx \
.. | stats first(msg) as msg by event_id
{"msg":"Listener RDP-Tcp has started listening","event_id":258}
{"msg":"Listener RDP-Tcp received a connection","event_id":261}
{"msg":null,"event_id":263}
{"msg":"RD Session Host Server role is not installed.","event_id":1136}
{"msg":"Remote Desktop Services: User authentication succeeded:    User: 172.20.29.xx  Domain:   Source Network Address: ","event_id":1149}
{"msg":null,"event_id":20523}
```

Event ID 1149 contains IP address. You can parse user, domain, source IP address using regular expression, and summarize it using `stats` command.

```
logpresso> evtx-file TerminalServices-RemoteConnectionManager.evtx \
.. | search event_id == 1149 \
.. | fields _time, msg \
.. | rex field=msg "User: (?<nt_user>\S+).*?Domain: (?<nt_domain>\S+).*?Source Network Address: (?<src_ip>\S+)" \
.. | search isnotnull(nt_user)
.. | stats count, min(_time) as first_seen, max(_time) as last_seen by nt_user, src_ip \
.. | sort -count
{"src_ip":"172.20.29.xx","first_seen":"2020-11-30 15:30:55+0900","last_seen":"2021-02-18 10:27:31+0900","nt_user":"xeraph","count":53}
{"src_ip":"172.20.1.yy","first_seen":"2020-11-27 10:27:36+0900","last_seen":"2021-04-08 13:31:45+0900","nt_user":"xeraph","count":50}
...
{"src_ip":"172.20.29.30","first_seen":"2021-03-25 12:28:02+0900","last_seen":"2021-03-25 12:28:02+0900","nt_user":"8con","count":1}
{"src_ip":"172.20.1.zz","first_seen":"2021-02-26 15:44:54+0900","last_seen":"2021-02-26 15:44:54+0900","nt_user":"xeraph","count":1}
```

There is a connection from another domain account `8con`. Detecting rare RDP connections is easy.
