# Logpresso Mini

Logpresso Mini is minimized single binary command-line tool of Logpresso platform. You can analyze various security logs and forensic artifacts. For example,

 * Parse the NTFS MFT file from an NTFS filesystem, and find deleted file or hidden files.
 * Parse windows .evtx files and find suspicious powershell command executions.
 * Parse registry hive files and find history of every connected USB devices.
 * Parse apache or nginx web logs and detect web application attacks (e.g. SQLi)
 * Parse firewall logs and detect host or port scanning activities.
 * Parse firewall logs and detect known C&C connect attempts.
 * and more!

Logpresso Mini is very powerful because it can sort and aggregate any data, and can join not only local file but also remote data. e.g. web pages.

### Download
* [Logpresso Mini 1.0.0 (Windows x64)](https://github.com/logpresso/community/releases/download/v1.0.0/logpresso.exe)
* [Logpresso Mini 1.0.0 (Linux x64)]()

### Getting Started
You can see basic usage using `-h` option.
```
CMD> logpresso -h
usage: logpresso [option] [arg]
Options and arguments
-d key=value : define query parameter
-p port      : open udp syslog port and use socket as stdin
-q query     : query string passed in as string (terminates option list)
file         : query string read from file
```

On Windows:
```
E:\github\mini>dir | logpresso -q "-"
{"line":" Volume in drive E is DATA"}
{"line":" Volume Serial Number is B830-8C4E"}
{"line":""}
{"line":" Directory of E:\\github\\mini"}
{"line":""}
{"line":"2021-04-09  PM 04:32    <DIR>          ."}
{"line":"2021-04-09  PM 04:32    <DIR>          .."}
...
```

On Linux: (assume that logpresso binary is in /usr/bin)
```
$ ls | logpresso -q '-'
{"line":"fw.csv"}
{"line":"fw.json"}
{"line":"fw.tsv"}
{"line":"fw.txt"}
{"line":"messages"}
{"line":"messages.2020-04-17"}
```

`-` means standard input. You can aggregate standard input using `stats` command like this:

```
$ ls | logpresso -q '- | stats count'
{"count":6}
```
You can find failed ssh login attempts like this:
```
# logpresso -q 'textfile /var/log/secure | search line == "*authentication failure*" | fields line '
{"line":"Apr  6 11:43:26 wood sshd[23029]: pam_unix(sshd:auth): authentication failure; logname= uid=0 euid=0 tty=ssh ruser= rhost=172.20.1.xx  user=8con"}
{"line":"Apr  6 11:43:40 wood sudo: pam_unix(sudo:auth): authentication failure; logname=8con uid=16777216 euid=0 tty=/dev/pts/0 ruser=8con rhost=  user=8con"}
{"line":"Apr  6 14:47:45 wood sshd[23029]: PAM 1 more authentication failure; logname= uid=0 euid=0 tty=ssh ruser= rhost=172.20.1.xx  user=8con"}
{"line":"Apr  6 18:00:09 wood sshd[3109]: pam_unix(sshd:auth): authentication failure; logname= uid=0 euid=0 tty=ssh ruser= rhost=172.20.29.xx  user=8con"}
{"line":"Apr  8 20:41:26 wood sshd[21866]: pam_unix(sshd:auth): authentication failure; logname= uid=0 euid=0 tty=ssh ruser= rhost=xeraph.hq.logpresso.net  user=xeraph"}
{"line":"Apr  8 20:41:31 wood sshd[21897]: pam_unix(sshd:auth): authentication failure; logname= uid=0 euid=0 tty=ssh ruser= rhost=xeraph.hq.logpresso.net  user=xeraph"}
{"line":"Apr  9 17:56:12 wood sshd[13484]: pam_unix(sshd:auth): authentication failure; logname= uid=0 euid=0 tty=ssh ruser= rhost=xeraph.hq.logpresso.net  user=xeraph"}
{"line":"Apr  9 17:56:49 wood sudo: pam_unix(sudo:auth): authentication failure; logname=xeraph uid=16777220 euid=0 tty=/dev/pts/0 ruser=xeraph rhost=  user=xeraph"}
{"line":"Apr  9 18:02:12 wood sudo: pam_unix(sudo:auth): authentication failure; logname=xeraph uid=16777220 euid=0 tty=/dev/pts/0 ruser=xeraph rhost=  user=xeraph"}
```

You can aggregate failed ssh login attempts per username like this:
```
# logpresso -q 'textfile /var/log/secure | search line == "*authentication failure*" | rex field=line "user=(?<user>\S+)" | stats count by user'
{"count":4,"user":"8con"}
{"count":5,"user":"xeraph"}
```

### Contents
 * [How to find deleted files from MFT](https://github.com/logpresso/community/blob/main/contents/001_How_to_analyze_NTFS_MFT.md)
 * [How to build file deletion trend over time from UsnJrnl](https://github.com/logpresso/community/blob/main/contents/002_How_to_analyze_NTFS_UsnJrnl.md)
 * [How to find connected USB devices from registry hive files](https://github.com/logpresso/community/blob/main/contents/003_How_to_analyze_Registry.md)

### Log parser
Logpresso Mini embeds 40+ log parsers for commercial security products. However, parser may not work correctly since log format is ever changing over time. In case of this, you can support Logpresso team by contributing log samples. Please create an issue and describe product name, firmware version, and upload log file.

Following log formats are supported:
* Common Log Formats
  * CSV
  * JSON: logstash uses JSON as primary format.
  * CEF: Micro Focus ArcSight uses CEF as primary format.
  * LEEF: IBM QRadar uses LEEF as primary format.
* Global products
  * paloalto: Palo Alto Networks NGFW
  * fortigate: Fortinet UTM
  * cyberoam: Cyberoam UTM
  * sonicwall: SonicWall NGFW
  * watchguard: WatchGuard Firebox firewall
  * netscreen-isg: Juniper Netscreen firewall
  * darktrace: Darktrace Enterprise Immune System
  * defensepro: Radware DefensePro
  * riorey-sys: RioRey
  * secure-sphere: Imperva SecureSphere
* South Korea products
  * mf2: SECUI MF2 Firewall
  * nxg: SECUI NXG Firewall
  * exshield-csv: SECUI eXshield Firewall
  * axgate: AXGATE UTM
  * trusguard: Ahnlab Trusguard Firewall
  * weguardia: FutureSystems Firewall
  * vforce: NEXG VForce UTM
  * neobox: XNsystems NeoBox Firewall
  * sniper: WINS Sniper IPS
  * mfi: SECUI MFI IPS
  * wapples-syslog: Penta Security Wapples
  * webfront: PIOLINK WebFront
  * ahnlab_mds: Ahnlab MDS
  * sniper-aptx: WINS Sniper APTX
  * cube-defense: TriCubeLab CubeDefense

### Contributing

If you have developed useful query, please send Pull Request. Any bug reports or discussions are also highly appreciated.

### Need SIEM?
If you want full featured SIEM products, please contact sales@logpresso.com

| Feature | Logpresso Mini | Logpresso Enterprise | Logpresso Sonar |
| ------- | -------------- | -------------------- | --------------- |
| Basic Query        | o | o | o |
| Distributed Query  | x | o | o |
| Schemaless Storage | x | o | o |
| Fulltext Search    | x | o | o |
| Streaming Query | x | o | o |
| Complex Event Processing | x | o | o |
| Vectorized Query Execution | x | o | o |
| Dashboard and Drilldown         | x | o | o |
| Rule Management    | x | x | o |
| Case Management    | x | x | o |



