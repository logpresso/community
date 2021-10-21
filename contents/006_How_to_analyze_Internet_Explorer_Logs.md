## How to analyze Internet Explorer 10 and 11
When you visit any website, a web browser writes internet history. In Internet Explorer 10 or 11, you can find the history logs stored in WebCacheV01.dat file. This file is an ESE database, also known as the Jet Blue engine.
This file is located under C:\Users\USERNAME\AppData\Local\Microsoft\Windows\WebCache. WebCacheV01.dat file is locked by taskhostw process. You can use [lockless.exe](https://github.com/GhostPack/Lockless) to copy locked files like this:
```
cmd> lockless.exe WebCacheV01.dat /process:taskhostw /copy:WebCacheV01.ese
[*] Searching processes for an open handle to "WebCacheV01.dat"
[+] Process "taskhostw" (10540) has a file handle (ID 768) to "C:\Users\USERNAME\AppData\Local\Microsoft\Windows\WebCache\WebCacheV01.dat"
[*] Copying to: WebCacheV01.ese
[*] Copied 120586240 bytes from "C:\Users\USERNAME\AppData\Local\Microsoft\Windows\WebCache\WebCacheV01.dat" to "WebCacheV01.ese"
```
If you can't run lockless.exe, kill taskhostw.exe, copy WebCacheV01.dat, and restart windows:
```
cmd> taskkill /im taskhostw.exe
```
Since it is copied while the process is running, the WebCacheV01.dat file is under dirty state. Yet, Logpresso Mini can read ESE database files in dirty state as well.
Now it's ready for analysis. Run Logpresso Mini 1.1.0 or above.
### IE Visited Websites
Internet Explorer stores all visit logs in the Container_ID table. **ie-visits** command loads all website access logs from the ESE database file.
```
logpresso> ie-visits WebCacheV01.dat | fields - response_headers
{"visit_count":3,"secure_dir":0,"sync_time":"2020-04-09 12:59:23+0900","url":"https://mail.google.com/mail/u/0/","file_size":0,"cache_id":0,"modified_time":"2020-04-09 12:59:20+0900","url_hash":179678190000737534,"expiry_time":"2020-07-14 12:52:10+0900","_time":"2020-04-09 12:59:23+0900","entry_id":3659,"user":"xeraph","container_id":17}
...
```
If you want to summarize the number of visits by domain, use a query as follows:
```
logpresso> ie-visits WebCacheV01.dat | eval domain = valueof(split(url, "/"), 2) | search len(domain) > 0 | stats count by domain | sort -count
{"domain":"login.live.com","count":108}
{"domain":"www.bing.com","count":80}
{"domain":"github.com","count":63}
...
```
### IE Downloaded Files
**ie-downloads** command loads all download logs from the ESE database file.
```
logpresso> ie-downloads e:\logsample\forensic\ie\WebCacheV01.dat | fields _time, file_path, url
{"file_path":"C:\\Users\\xeraph\\Downloads\\PaulLarson.pptx","_time":"2019-07-22 00:55:57+0900","url":"https://dsg.uwaterloo.ca/seminars/notes/PaulLarson.pptx"}
...
```
### IE Cookies
**ie-cookies** command shows URL to cookie file name mappings.
```
logpresso> ie-cookies WebCacheV01.dat | fields _time, url, file_name
{"file_name":"XI81CE9J.cookie","_time":"2018-02-22 11:15:20+0900","url":"microsoft.com/"}
...
```
### IE Cache Files
You can also investigate temporary internet files using **ie-cache-files** command.
```
logpresso> ie-cache-files WebCacheV01.dat | fields _time, file_name, url
{"file_name":"js[1].js","_time":"2021-06-14 15:44:12+0900","url":"https://www.googletagmanager.com/gtag/js?id=UA-90780581-11"}
...
```
### ESE database browsing
*ie-* prefixed commands are just syntactic sugar. You can navigate original ESE database records.
**esedb-tables** command dumps all table names with column names.
```
logpresso> esedb-tables WebCacheV01.dat
{"_file":"WebCacheV01.dat","columns":["ObjidTable","Type","Id","ColtypOrPgnoFDP","SpaceUsage","Flags","PagesOrLocale","RootFlag","RecordOffset","LCMapFlags","KeyMost","Name","Stats","TemplateTable","DefaultValue","KeyFldIDs","VarSegMac","ConditionalColumns","TupleLimits","Version","SortID","CallbackData","CallbackDependencies","SeparateLV","SpaceHints","SpaceDeferredLVHints","LocaleName"],"table_name":"MSysObjects"}
...
```
**esedb-columns** command returns detailed column definitions of the specified table names.
```
logpresso> esedb-columns table="Containers" WebCacheV01.dat
{"column_id":1,"_file":"WebCacheV01.dat","column_name":"ContainerId","column_type":"long long","table_name":"Containers"}
{"column_id":2,"_file":"WebCacheV01.dat","column_name":"SetId","column_type":"unsigned long","table_name":"Containers"}
{"column_id":3,"_file":"WebCacheV01.dat","column_name":"Flags","column_type":"unsigned long","table_name":"Containers"}
{"column_id":4,"_file":"WebCacheV01.dat","column_name":"Size","column_type":"long long","table_name":"Containers"}
...
```
**esedb-records** command loads actual records in the specified table.
```
logpresso> esedb-records table="Containers" WebCacheV01.dat  | fields ContainerId, Name | limit 10
{"ContainerId":1,"Name":"History"}
{"ContainerId":2,"Name":"Content"}
{"ContainerId":3,"Name":"Cookies"}
{"ContainerId":4,"Name":"Content"}
{"ContainerId":5,"Name":"DOMStore"}
...
```