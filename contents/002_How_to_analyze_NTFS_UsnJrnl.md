## How to analyze NTFS UsnJrnl

USN Journal (Update Sequence Number Journal) is a feature of the NTFS which maintains a record of changes made to the volume.

`ntfs-usnjrnl` command parses USN journal file and provides following output fields:

* _file: Loaded usnjrnl file name
* _time: Event time
* file_name: File name
* reason: File operation
* file_no: File number
* file_ref: MFT reference number
* parent_file_no: Directory number
* parent_file_ref: Directory MFT reference number
* usn: Update sequence number

Let's see how `ntfs-usnjrnl` command works:

```
logpresso> ntfs-usnjrnl UsnJrnl | limit 3
{"_file":"UsnJrnl","reason":["BASIC_INFO_CHANGE","CLOSE"],"parent_file_no":25044,"usn":1501560832,"parent_file_ref":7881299347923412,"file_name":"SetNmpipeStateRequest.class","_time":"2020-01-09 20:44:04+0900","file_ref":4503599627400991,"file_no":30495}
{"_file":"UsnJrnl","reason":["CLOSE","FILE_DELETE"],"parent_file_no":25044,"usn":1501560952,"parent_file_ref":7881299347923412,"file_name":"SetNmpipeStateRequest.class","_time":"2020-01-09 20:44:04+0900","file_ref":4503599627400991,"file_no":30495}
{"_file":"UsnJrnl","reason":["BASIC_INFO_CHANGE"],"parent_file_no":25044,"usn":1501561072,"parent_file_ref":7881299347923412,"file_name":"ReadNmpipeRequest.class","_time":"2020-01-09 20:44:04+0900","file_ref":7881299347923415,"file_no":25047}
```

### Find delete actions

You can split records by `reason` and filter `FILE_DELETE` action like this:

```
logpresso> ntfs-usnjrnl UsnJrnl | explode reason | search reason == "FILE_DELETE" | limit 3
{"_file":"UsnJrnl","parent_file_no":25044,"usn":1501560952,"reason":"FILE_DELETE","parent_file_ref":7881299347923412,"file_name":"SetNmpipeStateRequest.class","_time":"2020-01-09 20:44:04+0900","file_ref":4503599627400991,"file_no":30495}
{"_file":"UsnJrnl","parent_file_no":25044,"usn":1501561296,"reason":"FILE_DELETE","parent_file_ref":7881299347923412,"file_name":"ReadNmpipeRequest.class","_time":"2020-01-09 20:44:04+0900","file_ref":7881299347923415,"file_no":25047}
{"_file":"UsnJrnl","parent_file_no":25044,"usn":1501561648,"reason":"FILE_DELETE","parent_file_ref":7881299347923412,"file_name":"RawWriteNmpipeRequest.class","_time":"2020-01-09 20:44:04+0900","file_ref":4503599627401288,"file_no":30792}
```

$USNJRNL only contains file name, you should join MFT data to obtain full file path. Since deleted file metadata can be already overwritten by other live file metadata, you should join parent directory in this case.

`join` command supports all kinds of join types: inner, left, right, leftonly, rightonly, cross. If you don't specify `type` option, default join type is `inner`. Use `[` and `]` to describe subquery string. `parent_file_no` is a join key field here.

```
logpresso> ntfs-usnjrnl UsnJrnl | explode reason | search reason == "FILE_DELETE" | limit 3 | join parent_file_no [ ntfs-mft MFT | fields no, file_path | rename no as parent_file_no ] | fields _time, file_path, file_name, reason
{"file_path":"github\\ent\\middleware\\pcap\\araqne-pcap-smb\\target\\classes\\org\\araqne\\pcap\\smb\\transreq","reason":"FILE_DELETE","file_name":"SetNmpipeStateRequest.class","_time":"2020-01-09 20:44:04+0900"}
{"file_path":"github\\ent\\middleware\\pcap\\araqne-pcap-smb\\target\\classes\\org\\araqne\\pcap\\smb\\transreq","reason":"FILE_DELETE","file_name":"ReadNmpipeRequest.class","_time":"2020-01-09 20:44:04+0900"}
{"file_path":"github\\ent\\middleware\\pcap\\araqne-pcap-smb\\target\\classes\\org\\araqne\\pcap\\smb\\transreq","reason":"FILE_DELETE","file_name":"RawWriteNmpipeRequest.class","_time":"2020-01-09 20:44:04+0900"}
```

### File operation statistics

If someone tried to destroy evidences, a lot of delete operations may occur. Aggregation can provides 10000 feet view for us.

```
logpresso> ntfs-usnjrnl UsnJrnl | explode reason | search reason == "FILE_DELETE" | timechart span=1d count
{"count":29499,"_time":"2020-01-09 00:00:00+0900"}
{"count":1445,"_time":"2020-01-10 00:00:00+0900"}
{"count":1458,"_time":"2020-01-11 00:00:00+0900"}
{"count":1443,"_time":"2020-01-12 00:00:00+0900"}
{"count":1448,"_time":"2020-01-13 00:00:00+0900"}
{"count":1439,"_time":"2020-01-14 00:00:00+0900"}
{"count":1441,"_time":"2020-01-15 00:00:00+0900"}
{"count":1441,"_time":"2020-01-16 00:00:00+0900"}
{"count":1447,"_time":"2020-01-17 00:00:00+0900"}
{"count":1441,"_time":"2020-01-18 00:00:00+0900"}
{"count":1441,"_time":"2020-01-19 00:00:00+0900"}
{"count":1452,"_time":"2020-01-20 00:00:00+0900"}
{"count":1452,"_time":"2020-01-21 00:00:00+0900"}
{"count":1441,"_time":"2020-01-22 00:00:00+0900"}
{"count":1754,"_time":"2020-01-23 00:00:00+0900"}
{"count":3704,"_time":"2020-01-24 00:00:00+0900"}
{"count":3109,"_time":"2020-01-25 00:00:00+0900"}
```

`1d` means one day. You can use other unit of time:
* s: second
* m: minute
* h: hour
* d: day
* w: week
* mon: month


