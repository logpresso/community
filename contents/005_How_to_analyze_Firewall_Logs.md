## How to analyze Firewall logs

In most environments, firewall controls north-south traffic. Well-designed network also deploys Anti-DDoS, IPS, WAF (Web Application Firewall). Every firewall supports session (or flow) logging via syslog, snmp, or REST API.

Although each firewall product has its own characteristics, but there are common logging elements. All flow records contains standard 5-tuple:
* Source IP
* Source port
* Destination IP
* Destination port
* L4 Protocol

In addition to these fields, there are some especially useful metadata:

* Application: L7 protocol (e.g. HTTP, DNS) or web service name
* Action: PERMIT, DENY or DROP
  * `DENY` means TCP RST response, `DROP` means packet drop (client may suffer long timeout)
* Sent bytes: Bytes transferred from client to server
* Received bytes: Bytes transferred from server to client
* Sent packets: Frames transferred from client to server
* Received packets: Frames transferred from server to client

To analyze firewall logs, you should normalize firewall logs into standard form. Before we start, let's check some installation.

### GeoLite2 setup

Logpresso Mini supports geolocation lookup using MaxMind GeoLite2 database. Download geolite2 files from Logpresso Mini package distribution. Unzip geolite2 database file and use `-g <path>` option to specify geolite2 database path. You can check if geolite2 database is loaded successfully.

```
CMD> logpresso -g geolite2
Logpresso Mini 1.0.0 (2021-04-09)
Copyright (c) 2021 Logpresso Inc.
See https://docs.logpresso.com/manual/query_manual_ko
logpresso> json "{}" | eval ip="8.8.8.8" | lookup geoip ip output country, city, asn, latitude, longitude
{"country":"US","city":null,"ip":"8.8.8.8","latitude":37.750999450683594,"asn":"AS15169 GOOGLE","longitude":-97.8219985961914}
```

### Syslog server setup

You can receive syslog UDP packets using `-p <port>` option. Logpresso Mini will display received syslog packets in real-time.

```
CMD> logpresso -p 514 -q "-"
{"line":"<134>Oct 17 05:54:19 FP60 SFIMS: Protocol: TCP, SrcIP: 172.20.8.30, DstIP: 151.101.72.133, SrcPort: 61360, DstPort: 443, TCPFlags: 0x0, IngressInterface: eth1, IngressZone: Passive, DE: Primary Detection Engine (96f4547e-d715-11e5-b0f4-bdee139db4d8), Policy: FP_IDS_FILE_POLICY, ConnectType: End, AccessControlRuleName: IDS-FILE-RULE, AccessControlRuleAction: Allow, AccessControlRuleReason: Intrusion Block, UserName: No Authentication Required, Client: SSL client, ApplicationProtocol: HTTP, IPSCount: 1, InitiatorPackets: 10, ResponderPackets: 7, InitiatorBytes: 1216, ResponderBytes: 630, NAPPolicy: Balanced Security and Connectivity, DNSResponseType: No Error, Sinkhole: Unknown, URLCategory: Unknown, URLReputation: Risk unknown, URL: https://avatars0.githubusercontent.com"}
{"line":"<134>Oct 17 05:54:19 FP60 SFIMS: Protocol: TCP, SrcIP: 172.20.8.30, DstIP: 151.101.72.133, SrcPort: 61359, DstPort: 443, TCPFlags: 0x0, IngressInterface: eth1, IngressZone: Passive, DE: Primary Detection Engine (96f4547e-d715-11e5-b0f4-bdee139db4d8), Policy: FP_IDS_FILE_POLICY, ConnectType: End, AccessControlRuleName: IDS-FILE-RULE, AccessControlRuleAction: Allow, UserName: No Authentication Required, Client: SSL client, ApplicationProtocol: HTTP, InitiatorPackets: 10, ResponderPackets: 7, InitiatorBytes: 1216, ResponderBytes: 630, NAPPolicy: Balanced Security and Connectivity, DNSResponseType: No Error, Sinkhole: Unknown, URLCategory: Unknown, URLReputation: Risk unknown, URL: https://avatars0.githubusercontent.com"}
```

### How to write syslog packets

In some cases, you need to receive firewall logs and write them to a text log file. `-p 514` option means that Logpresso Mini uses 514/udp port as standard input.

```
CMD> logpresso -p 514 -q "- | outputtxt fw.log line"
{"line":"<134>Oct 17 05:54:19 FP60 SFIMS: Protocol: TCP, SrcIP: 172.20.8.30, DstIP: 151.101.72.133, SrcPort: 61360, DstPort: 443, TCPFlags: 0x0, IngressInterface: eth1, IngressZone: Passive, DE: Primary Detection Engine (96f4547e-d715-11e5-b0f4-bdee139db4d8), Policy: FP_IDS_FILE_POLICY, ConnectType: End, AccessControlRuleName: IDS-FILE-RULE, AccessControlRuleAction: Allow, AccessControlRuleReason: Intrusion Block, UserName: No Authentication Required, Client: SSL client, ApplicationProtocol: HTTP, IPSCount: 1, InitiatorPackets: 10, ResponderPackets: 7, InitiatorBytes: 1216, ResponderBytes: 630, NAPPolicy: Balanced Security and Connectivity, DNSResponseType: No Error, Sinkhole: Unknown, URLCategory: Unknown, URLReputation: Risk unknown, URL: https://avatars0.githubusercontent.com"}
{"line":"<134>Oct 17 05:54:19 FP60 SFIMS: Protocol: TCP, SrcIP: 172.20.8.30, DstIP: 151.101.72.133, SrcPort: 61359, DstPort: 443, TCPFlags: 0x0, IngressInterface: eth1, IngressZone: Passive, DE: Primary Detection Engine (96f4547e-d715-11e5-b0f4-bdee139db4d8), Policy: FP_IDS_FILE_POLICY, ConnectType: End, AccessControlRuleName: IDS-FILE-RULE, AccessControlRuleAction: Allow, UserName: No Authentication Required, Client: SSL client, ApplicationProtocol: HTTP, InitiatorPackets: 10, ResponderPackets: 7, InitiatorBytes: 1216, ResponderBytes: 630, NAPPolicy: Balanced Security and Connectivity, DNSResponseType: No Error, Sinkhole: Unknown, URLCategory: Unknown, URLReputation: Risk unknown, URL: https://avatars0.githubusercontent.com"}
...
```

Press Ctrl-C to terminate. Now check how many logs are received.

### Understanding destinations

```
CMD> logpresso -q "textfile fw.log | stats count"
{"count":100}
```

Altough you can parse this log format using regular expression or combination of string functions, Logpresso Mini embeds log parsers for well-known security products. Since these logs are sent by Cisco Firepower, you can use `firepower` parser.

```
CMD> logpresso -q "textfile fw.log | parse firepower"
{"de":"Primary Detection Engine (96f4547e-d715-11e5-b0f4-bdee139db4d8)","accesscontrolrulereason":"Intrusion Block","nap_policy":"Balanced Security and Connectivity","src_iface":"eth1","type":"FP6","sinkhole":"Unknown","dst_ip":"151.101.72.133","sent_bytes":1216,"src_ip":"172.20.8.30","device_name":"4:19","rcvd_bytes":630,"tcp_flags":"0x0","sfims":"Protocol: TCP","client":"SSL client","action":"PERMIT","ipscount":"1","rcvd_pkts":7,"policy":"FP_IDS_FILE_POLICY","app":"HTTP","sent_pkts":10,"access_control_rule_name":"IDS-FILE-RULE","url":"https://avatars0.githubusercontent.com","src_port":61360,"src_zone":"Passive","dst_port":443,"dns_response_type":"No Error","connect_type":"End","category":"Unknown","user":"No Authentication Required","url_reputation":"Risk unknown"}
...
```

How can we know or classify destination? MaxMind's GeoLite2 converts IP address to geolocation. (assume `-g` option)

```
logpresso> textfile fw.log \
.. | parse firepower \
.. | lookup geoip dst_ip output country, asn \
.. | search isnotnull(country) \
.. | stats count by dst_ip, country, asn
{"country":"KR","count":14,"asn":"AS4766 Korea Telecom","dst_ip":"14.63.169.218"}
{"country":"KR","count":1,"asn":"AS9286 KINX","dst_ip":"103.6.100.24"}
{"country":"US","count":1,"asn":"AS14618 AMAZON-AES","dst_ip":"107.23.165.206"}
{"country":"KR","count":1,"asn":"AS9318 SK Broadband Co Ltd","dst_ip":"114.207.245.166"}
{"country":"KR","count":1,"asn":"AS3786 LG DACOM Corporation","dst_ip":"117.52.2.242"}
```

With ASN (Autonomous System Number) information, you can understand outbound traffic and identify anomalous sessions.

### Finding lateral movements

If an attacker breaks into corporate network successfully, he or she may want to know internal network structure and find valuable resources. Therefore, network scanning attempt from DMZ server or OA zone is a significant signal.

In many cases, only one or two packets are sent for port scan. You can detect scanning activities using this query. However, you have to be careful if source and destination is swapped. Some legacy firewalls cannot detect flow direction correctly.

```
logpresso> textfile fw.log \
.. | parse firepower \
.. | fields _time, src_ip, src_port, dst_ip, dst_port, protocol, sent_pkts, sent_bytes \
.. | search src_port > dst_port \
.. | search protocol == "TCP" and sent_pkts <= 2 \
.. | search network(src_ip, 16) == "172.20.0.0" \
.. | search network(dst_ip, 16) == "172.20.0.0" \
.. | eval _time = datetrunc(_time, "10m") \
.. | eval dst = concat(dst_ip, ":", dst_port) \
.. | stats dc(dst) as count, values(dst) as dst by src_ip, _time \
.. | search count >= 10 \
.. | eval dst = strjoin("\n", dst)

```
(Use sent_bytes if sent_pkts is not available.)

