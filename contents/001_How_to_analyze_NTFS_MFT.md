## How to analyze NTFS MFT

In this article, we will use CodeGate 2012 dataset.

In NTFS, all file metadata is stored in the Master File Table (MFT). `ntfs-mft` query command parses MFT file and provides following fields:

* no: MFT record number
* file_path: File path
* file_name: File name
* file_size: File size in bytes
* alloc_size: Allocated file size
* in_use: Is in use (or deleted)
* is_dir: Is directory
* link_count: Hard link count
* created_at: File creation time from $FILENAME attribute
* modified_at: File modification time from $FILENAME attribute
* access_at: File access time from $FILENAME attribute
* mft_modified_at: MFT record modification time from $FILENAME attribute
* std_created_at: File creation time from $STANDARD_INFORMATION attribute
* std_modified_at: File modification time from $STANDARD_INFORMATION attribute
* std_mft_modified_at: MFT record modification time from $STANDARD_INFORMATION attribute
* std_access_at: File access time from $STANDARD_INFORMATION attribute
* is_readonly: Is read-only file
* is_hidden: Is hidden file
* is_system: Is system file
* is_archive: Is archive file
* is_device: Is device file
* is_normal: Is normal file
* is_temp: Is temporary file
* is_sparse: Is sparse file
* is_reparse: Is reparse file
* is_compressed: Is compressed file
* is_offline: Is offline file
* is_indexed: Is indexed file
* is_encrypted: Is encrypted file
* lsn: $LogFile sequence number
* seq: Sequence of MFT record
* file_ref: File reference number
* parent_file_ref: Directory reference number
* parent_no: MFT record number of parent directory


### Find new .exe file after specific date
Use `search` command to filter records using boolean expressions. `fields` command selects output fields.

```
logpresso> ntfs-mft MFT | search file_name == "*.exe" and created_at >= date("2012-02-23", "yyyy-MM-dd") | fields file_path, created_at
{"file_path":"$Recycle.Bin\\r32.exe","created_at":"2012-02-23 02:39:18+0900"}
{"file_path":"Users\\proneer\\AppData\\Local\\Temp\\RarSFX1\\setup.exe","created_at":"2012-02-23 02:40:27+0900"}
{"file_path":"Users\\proneer\\AppData\\Local\\Temp\\RarSFX1\\WinHex.exe","created_at":"2012-02-23 02:40:27+0900"}
```

### Find deleted files

If file is deleted, `in_use` is changed to `false`. However, if MFT record is not in use, it will be overwritten by other file metadata soon. You can use `not()` function to invert boolean value.

```
logpresso> ntfs-mft MFT | search not(in_use)
{"_file":"MFT","no":16,"lsn":33557136,"is_dir":false,"link_count":0,"in_use":false,"file_ref":0,"seq":1,"file_size":0,"alloc_size":0}
{"_file":"MFT","no":17,"lsn":33557155,"is_dir":false,"link_count":0,"in_use":false,"file_ref":0,"seq":1,"file_size":0,"alloc_size":0}
{"_file":"MFT","no":18,"lsn":33557174,"is_dir":false,"link_count":0,"in_use":false,"file_ref":0,"seq":1,"file_size":0,"alloc_size":0}
{"_file":"MFT","no":19,"lsn":33557193,"is_dir":false,"link_count":0,"in_use":false,"file_ref":0,"seq":1,"file_size":0,"alloc_size":0}
{"_file":"MFT","no":20,"lsn":33557212,"is_dir":false,"link_count":0,"in_use":false,"file_ref":0,"seq":1,"file_size":0,"alloc_size":0}
{"_file":"MFT","no":21,"lsn":33557231,"is_dir":false,"link_count":0,"in_use":false,"file_ref":0,"seq":1,"file_size":0,"alloc_size":0}
{"_file":"MFT","no":22,"lsn":33557250,"is_dir":false,"link_count":0,"in_use":false,"file_ref":0,"seq":1,"file_size":0,"alloc_size":0}
{"_file":"MFT","no":23,"lsn":33557269,"is_dir":false,"link_count":0,"in_use":false,"file_ref":0,"seq":1,"file_size":0,"alloc_size":0}
{"no":1167,"access_at":"2012-02-23 02:45:08+0900","file_path":"BAEDB0~1","lsn":742817085,"link_count":2,"is_hidden":false,"created_at":"2012-02-23 02:45:08+0900","in_use":false,"parent_no":2574,"is_offline":false,"parent_file_ref":281474976713230,"is_temp":false,"is_compressed":false,"is_dir":false,"is_device":false,"modified_at":"2012-02-23 02:45:08+0900","is_indexed":true,"seq":4,"std_created_at":"2012-02-23 02:45:08+0900","is_system":false,"std_mft_modified_at":"2012-02-23 02:45:08+0900","is_encrypted":false,"file_name":"BAEDB0~1","is_sparse":false,"std_modified_at":"2012-02-23 02:45:08+0900","file_size":20,"alloc_size":20,"mft_modified_at":"2012-02-23 02:45:08+0900","is_reparse":false,"is_readonly":false,"_file":"MFT","std_access_at":"2012-02-23 02:45:08+0900","is_normal":false,"is_archive":true,"file_ref":0}
```

### Count files and directories
Use `stats` command for statistics. you can group by one or more fields.
```
logpresso> ntfs-mft MFT | stats count by is_dir
{"is_dir":false,"count":247}
{"is_dir":true,"count":1285}
```

### Find the 5 largest files
Use `sort` commands to sort data. `-` means descending order. (default is ascending order).

```
logpresso> ntfs-mft MFT | sort limit=5 -file_size | fields file_path, file_size
{"file_path":"Windows\\Installer\\1227ea.msp","file_size":425345024}
{"file_path":"7a5cfa309ab9f2ea8bcb81b710721bf5\\OFFICE~1.CAB","file_size":271250712}
{"file_path":"$MFT","file_size":161218560}
{"file_path":"$LogFile","file_size":67108864}
{"file_path":"Windows\\assembly\\NativeImages_v2.0.50727_32\\PresentationFramewo#\\8435718626a24beaeefc98d45ae77127\\PRESEN~1.DLL","file_size":14322688}
```
