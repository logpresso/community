## How to analyze registry

The Windows Registry is a hierarchical database that stores low-level settings for the Microsoft Windows operating system and for applications that opt to use the registry.

For demo, export your registry hive using `regedit.exe`. For example, you can obtain your own `NTUSER.DAT` file through following steps:

1. Open the Registry Editor
2. Right click `HKEY_CURRENT_USER` node in the left pane.
3. Select `Export`
4. Change `Save as type` item to `Registry Hive Files`.
5. Browse to the directory to save the file to and enter a file name `NTUSER.DAT`.
5. Click OK to create the export file.


### Finding RDP usage

Most administrators use MSRDP (remote desktop protocol) to connect to a remote windows client or server using `mstsc.exe`. Many attackers also use RDP for lateral movement. This client program maintains recent input history in the registry.

```
logpresso> hive-file NTUSER.DAT | search key == "*Terminal Server Client*" and name == "*MRU*" | fields name, value, last_written
{"name":"MRU0","last_written":"2020-02-06 17:32:23+0900","value":"172.20.36.13"}
{"name":"MRU1","last_written":"2020-02-06 17:32:23+0900","value":"172.20.xx.xx"}
{"name":"MRU2","last_written":"2020-02-06 17:32:23+0900","value":"172.20.xx.xx"}
{"name":"MRU3","last_written":"2020-02-06 17:32:23+0900","value":"host1"}
{"name":"MRU4","last_written":"2020-02-06 17:32:23+0900","value":"172.20.xx.xx"}
{"name":"MRU5","last_written":"2020-02-06 17:32:23+0900","value":"host2"}
{"name":"MRU6","last_written":"2020-02-06 17:32:23+0900","value":"172.20.xx.xx"}
{"name":"MRU7","last_written":"2020-02-06 17:32:23+0900","value":"172.20.xx.xx"}
{"name":"MRU8","last_written":"2020-02-06 17:32:23+0900","value":"acme.iptime.org"}
{"name":"MRU9","last_written":"2020-02-06 17:32:23+0900","value":"172.20.xx.xx"}
```

### Finding recent PuTTY usage

Many applications also keep connection profiles or recent user-input history in the registry. PuTTY is an example.

```
logpresso> hive-file NTUSER.dat | search key == "*SimonTatham\\PuTTY\\Sessions*" and name == "HostName" | fields key, name, value, last_written
{"name":"HostName","last_written":"2020-08-01 22:02:22+0900","value":"192.168.xx.xx","key":"ROOT\\Software\\SimonTatham\\PuTTY\\Sessions\\master"}
{"name":"HostName","last_written":"2020-08-01 22:02:23+0900","value":"localhost","key":"ROOT\\Software\\SimonTatham\\PuTTY\\Sessions\\node2"}
...
```

### Finding USB devices

Let's see CodeGate 2011 Forensic challenge.

```
We are investigating the military secret's leaking. we found traffic with leaking secrets while monitoring the network. Security team was sent to investigate, immediately. But, there was no one present.

It was found by forensics team that all the leaked secrets were completely deleted by wiping tool. And the team has found a leaked trace using potable device. Before long, the suspect was detained. But he denies allegations.

Now, the investigation is focused on potable device. The given files are acquired registry files from system. The estimated time of the incident is Mon, 21 February 2011 15:24:28(KST).

Find a trace of portable device used for the incident.

The Key : "Vendor name" + "volume name" + "serial number" (please write in capitals)
```

You can extract mounted devices from SYSTEM registry hive file.

```
logpresso> hive-file codegate2011\system.bak | search key == "*MountedDevices" and name == "\\DosDevices*"
{"_file":"system.bak","name":"\\DosDevices\\C:","last_written":"2011-02-19 14:34:06+0900","type":"BINARY","value":"16726f7d0000100000000000","key":"CMI-CreateHive{F10156BE-0E87-4EFB-969E-5DA29D131144}\\MountedDevices"}
{"_file":"system.bak","name":"\\DosDevices\\D:","last_written":"2011-02-19 14:34:06+0900","type":"BINARY","value":"5c003f003f005c0049004400450023004300640052006f006d004d0041005400530048004900540041005f004400560044002d0052005f005f005f0055004a002d003800360038005f005f005f005f005f005f005f005f005f005f005f005f005f005f005f005f005f004b004200310039005f005f005f005f002300350026003200390030006600640033006100620026003000260031002e0030002e00300023007b00350033006600350036003300300064002d0062003600620066002d0031003100640030002d0039003400660032002d003000300061003000630039003100650066006200380062007d00","key":"CMI-CreateHive{F10156BE-0E87-4EFB-969E-5DA29D131144}\\MountedDevices"}
{"_file":"system.bak","name":"\\DosDevices\\A:","last_written":"2011-02-19 14:34:06+0900","type":"BINARY","value":"5c003f003f005c004600440043002300470045004e0045005200490043005f0046004c004f005000500059005f004400520049005600450023003600260032006200630031003300390034003000260030002600300023007b00350033006600350036003300300064002d0062003600620066002d0031003100640030002d0039003400660032002d003000300061003000630039003100650066006200380062007d00","key":"CMI-CreateHive{F10156BE-0E87-4EFB-969E-5DA29D131144}\\MountedDevices"}
{"_file":"system.bak","name":"\\DosDevices\\E:","last_written":"2011-02-19 14:34:06+0900","type":"BINARY","value":"5f003f003f005f00550053004200530054004f00520023004400690073006b002600560065006e005f0043006f00720073006100690072002600500072006f0064005f0055004600440026005200650076005f0030002e00300030002300640064006600300038006600620037006100380036003000370035002600300023007b00350033006600350036003300300037002d0062003600620066002d0031003100640030002d0039003400660032002d003000300061003000630039003100650066006200380062007d00","key":"CMI-CreateHive{F10156BE-0E87-4EFB-969E-5DA29D131144}\\MountedDevices"}
{"_file":"system.bak","name":"\\DosDevices\\F:","last_written":"2011-02-19 14:34:06+0900","type":"BINARY","value":"5c003f003f005c004400540053004f0046005400420055005300260052006500760031002300440054004300440052004f004d002600520065007600310023003100260037003900660035006400380037002600300026003000300023007b00350033006600350036003300300064002d0062003600620066002d0031003100640030002d0039003400660032002d003000300061003000630039003100650066006200380062007d00","key":"CMI-CreateHive{F10156BE-0E87-4EFB-969E-5DA29D131144}\\MountedDevices"}
{"_file":"system.bak","name":"\\DosDevices\\G:","last_written":"2011-02-19 14:34:06+0900","type":"BINARY","value":"5f003f003f005f00550053004200530054004f00520023004400690073006b002600560065006e005f00530061006e004400690073006b002600500072006f0064005f00550033005f004300720075007a00650072005f004d006900630072006f0026005200650076005f0032002e0031003800230030003000300030003100350036003000350039003600300035004100350043002600300023007b00350033006600350036003300300037002d0062003600620066002d0031003100640030002d0039003400660032002d003000300061003000630039003100650066006200380062007d00","key":"CMI-CreateHive{F10156BE-0E87-4EFB-969E-5DA29D131144}\\MountedDevices"}
{"_file":"system.bak","name":"\\DosDevices\\H:","last_written":"2011-02-19 14:34:06+0900","type":"BINARY","value":"5f003f003f005f00550053004200530054004f00520023004400690073006b002600560065006e005f00530045004c004600490043002600500072006f0064005f005300570049004e0047005f004d0049004e00490026005200650076005f0030002e00310041002300530046004100300037003100310030003000300030003500370046002600300023007b00350033006600350036003300300037002d0062003600620066002d0031003100640030002d0039003400660032002d003000300061003000630039003100650066006200380062007d00","key":"CMI-CreateHive{F10156BE-0E87-4EFB-969E-5DA29D131144}\\MountedDevices"}
{"_file":"system.bak","name":"\\DosDevices\\I:","last_written":"2011-02-19 14:34:06+0900","type":"BINARY","value":"5f003f003f005f00550053004200530054004f00520023004400690073006b002600560065006e005f004a006500740046006c006100730068002600500072006f0064005f005400720061006e007300630065006e0064005f0031004700420026005200650076005f0038002e0030003700230042005a0057004a005a004700470039002600310023007b00350033006600350036003300300037002d0062003600620066002d0031003100640030002d0039003400660032002d003000300061003000630039003100650066006200380062007d00","key":"CMI-CreateHive{F10156BE-0E87-4EFB-969E-5DA29D131144}\\MountedDevices"}
```

This registry value is of binary type. You can decode this binary value as UTF-16.

```
logpresso> hive-file codegate2011\system.bak | search key == "*MountedDevices" and name == "\\DosDevices*" | eval value = substr(decode(value, "UTF-16LE"), 4)
{"_file":"system.bak","name":"\\DosDevices\\C:","last_written":"2011-02-19 14:34:06+0900","type":"BINARY","value":"\u0000\u0000","key":"CMI-CreateHive{F10156BE-0E87-4EFB-969E-5DA29D131144}\\MountedDevices"}
{"_file":"system.bak","name":"\\DosDevices\\D:","last_written":"2011-02-19 14:34:06+0900","type":"BINARY","value":"IDE#CdRomMATSHITA_DVD-R___UJ-868_________________KB19____#5&290fd3ab&0&1.0.0#{53f5630d-b6bf-11d0-94f2-00a0c91efb8b}","key":"CMI-CreateHive{F10156BE-0E87-4EFB-969E-5DA29D131144}\\MountedDevices"}
{"_file":"system.bak","name":"\\DosDevices\\A:","last_written":"2011-02-19 14:34:06+0900","type":"BINARY","value":"FDC#GENERIC_FLOPPY_DRIVE#6&2bc13940&0&0#{53f5630d-b6bf-11d0-94f2-00a0c91efb8b}","key":"CMI-CreateHive{F10156BE-0E87-4EFB-969E-5DA29D131144}\\MountedDevices"}
{"_file":"system.bak","name":"\\DosDevices\\E:","last_written":"2011-02-19 14:34:06+0900","type":"BINARY","value":"USBSTOR#Disk&Ven_Corsair&Prod_UFD&Rev_0.00#ddf08fb7a86075&0#{53f56307-b6bf-11d0-94f2-00a0c91efb8b}","key":"CMI-CreateHive{F10156BE-0E87-4EFB-969E-5DA29D131144}\\MountedDevices"}
{"_file":"system.bak","name":"\\DosDevices\\F:","last_written":"2011-02-19 14:34:06+0900","type":"BINARY","value":"DTSOFTBUS&Rev1#DTCDROM&Rev1#1&79f5d87&0&00#{53f5630d-b6bf-11d0-94f2-00a0c91efb8b}","key":"CMI-CreateHive{F10156BE-0E87-4EFB-969E-5DA29D131144}\\MountedDevices"}
{"_file":"system.bak","name":"\\DosDevices\\G:","last_written":"2011-02-19 14:34:06+0900","type":"BINARY","value":"USBSTOR#Disk&Ven_SanDisk&Prod_U3_Cruzer_Micro&Rev_2.18#0000156059605A5C&0#{53f56307-b6bf-11d0-94f2-00a0c91efb8b}","key":"CMI-CreateHive{F10156BE-0E87-4EFB-969E-5DA29D131144}\\MountedDevices"}
{"_file":"system.bak","name":"\\DosDevices\\H:","last_written":"2011-02-19 14:34:06+0900","type":"BINARY","value":"USBSTOR#Disk&Ven_SELFIC&Prod_SWING_MINI&Rev_0.1A#SFA0711000057F&0#{53f56307-b6bf-11d0-94f2-00a0c91efb8b}","key":"CMI-CreateHive{F10156BE-0E87-4EFB-969E-5DA29D131144}\\MountedDevices"}
{"_file":"system.bak","name":"\\DosDevices\\I:","last_written":"2011-02-19 14:34:06+0900","type":"BINARY","value":"USBSTOR#Disk&Ven_JetFlash&Prod_Transcend_1GB&Rev_8.07#BZWJZGG9&1#{53f56307-b6bf-11d0-94f2-00a0c91efb8b}","key":"CMI-CreateHive{F10156BE-0E87-4EFB-969E-5DA29D131144}\\MountedDevices"}
```

Filter records by `USB` and extract fields using regular expression.

```
logpresso> hive-file codegate2011\system.bak | search key == "*MountedDevices" and name == "\\DosDevices*" | eval value = substr(decode(value, "UTF-16LE"), 4) | search value == "*USB*" | rex field=value "Ven_(?<vendor>[^&]+)&Prod_(?<product>[^&]+)&Rev_(?<version>[^#]+)#(?<serial>[^&]+)"  | eval serial = lower(serial) | fields vendor, product, version, serial
{"product":"UFD","serial":"ddf08fb7a86075","vendor":"Corsair","version":"0.00"}
{"product":"U3_Cruzer_Micro","serial":"0000156059605a5c","vendor":"SanDisk","version":"2.18"}
{"product":"SWING_MINI","serial":"sfa0711000057f","vendor":"SELFIC","version":"0.1A"}
{"product":"Transcend_1GB","serial":"bzwjzgg9","vendor":"JetFlash","version":"8.07"}
```

To solve the problem, you also need volume name and connection time. Connection time can be obtained from `HKLM\SYSTEM\ControlSet00X\Enum\USB\VID_####&PID_####` key in the SYSTEM registry hive file. Following query returns 66 records.

```
logpresso> hive-file codegate2011\system.bak | search key == "*USB\\VID_*" | eval serial = lower(valueof(split(key, "\\"), 5)) | stats max(last_written) as last_connect by serial
{"serial":"00000000000001","last_connect":"2011-02-17 15:34:14+0900"}
{"serial":"000000000002f4","last_connect":"2011-02-17 15:23:05+0900"}
{"serial":"00000002b5eb8d","last_connect":"2011-02-17 15:26:31+0900"}
{"serial":"0000156059605a5c","last_connect":"2011-02-19 14:23:07+0900"}
{"serial":"0000187b85700875","last_connect":"2011-02-17 15:32:38+0900"}
{"serial":"000a27001973c954","last_connect":"2011-02-19 14:22:34+0900"}
{"serial":"07751718047d","last_connect":"2011-02-17 15:28:50+0900"}
...
```

Volume name can be obtained from `HKLM\SOFTWARE\Microsoft\Windows Portable Devices\Devices` key in the SYSTEM registry hive file. Following query returns 40 records.

```
logpresso> hive-file codegate2011\software.bak | search key == "*Windows Portable Devices*" and name == "FriendlyName" | rex field=key "&REV_[^#]+#(?<serial>[^&]+)" | eval serial = lower(serial) | stats first(value) as volume_name by serial
{"serial":null,"volume_name":"Apple iPhone"}
{"serial":"00000002b5eb8d","volume_name":"ADATA UFD"}
{"serial":"0000156059605a5c","volume_name":"G:\\"}
{"serial":"0000187b85700875","volume_name":"G:\\"}
{"serial":"000a27001973c954","volume_name":"PRONEERIPOD"}
...
```

Now, you can join 3 different dataset by USB serial number.

```
logpresso> hive-file codegate2011\system.bak | search key == "*MountedDevices*" | eval value = substr(decode(value, "UTF-16LE"), 4) | search value == "*USB*" | rex field=value "Ven_(?<vendor>[^&]+)&Prod_(?<product>[^&]+)&Rev_(?<version>[^#]+)#(?<serial>[^&]+)"  | eval serial = lower(serial) | stats count by vendor, product, version, serial, value | join serial [ hive-file codegate2011\system.bak | search key == "*USB\\VID_*" | eval serial = lower(valueof(split(key, "\\"), 5)) | stats max(last_written) as last_connect by serial ] | join serial [ hive-file codegate2011\software.bak | search key == "*Windows Portable Devices*" and name == "FriendlyName" | rex field=key "&REV_[^#]+#(?<serial>[^&]+)" | eval serial = lower(serial) | stats first(value) as volume_name by serial ] | search last_connect >= date("2011-02-21", "yyyy-MM-dd") and last_connect <= date("2011-02-22", "yyyy-MM-dd") | order volume_name, vendor, product, version, serial, last_connect
{"product":"UFD","serial":"ddf08fb7a86075","vendor":"Corsair","count":2,"volume_name":"PR0N33R","last_connect":"2011-02-21 15:24:16+0900","version":"0.00","value":"USBSTOR#Disk&Ven_Corsair&Prod_UFD&Rev_0.00#ddf08fb7a86075&0#{53f56307-b6bf-11d0-94f2-00a0c91efb8b}"}
```

The answer key is `CORSAIRPR0N33RDDF08FB7A86075`.

