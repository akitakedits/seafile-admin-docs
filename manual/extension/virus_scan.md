# Virus Scan

Seafile can scan uploaded files for malicious content in the background. When configured to run periodically, the scan process scans all existing libraries on the server. In each scan, the process only scans newly uploaded/updated files since the last scan. For each file, the process executes a user-specified virus scan command to check whether the file is a virus or not. Most anti-virus programs provide command line utility for Linux.

To enable this feature, add the following options to `seafile.conf`:

```
[virus_scan]
scan_command = (command for checking virus)
virus_code = (command exit codes when file is virus)
nonvirus_code = (command exit codes when file is not virus)
scan_interval = (scanning interval, in unit of minutes, default to 60 minutes)
```

More details about the options:

* On Linux/Unix, most virus scan commands returns specific exit codes for virus and non-virus. You should consult the manual of your anti-virus program for more information.

An example for ClamAV (http://www.clamav.net/) is provided below:

```
[virus_scan]
scan_command = clamscan
virus_code = 1
nonvirus_code = 0
```

To test whether your configuration works, you can trigger a scan manually:

```
cd seafile-server-latest
./pro/pro.py virus_scan
```

If a virus was detected, you can see scan records and delete infected files on the Virus Scan page in the admin area.
![virus-scan](../images/virus-scan.png)

!!! tip "Tip"
    If you directly use clamav command line tool to scan files, scanning files will takes a lot of time. If you want to speed it up, we recommend to run Clamav as a daemon. Please refer to [Run ClamAV as a Daemon](./virus_scan_with_clamav.md)

When run Clamav as a daemon, the `scan_command` should be `clamdscan` in `seafile.conf`. An example for Clamav-daemon is provided below:
```
[virus_scan]
scan_command = clamdscan
virus_code = 1
nonvirus_code = 0
```
 
Since Pro edition 6.0.0, a few more options are added to provide finer grained control for virus scan.

```
[virus_scan]
......
scan_size_limit = (size limit for files to be scanned) # The unit is MB.
scan_skip_ext = (a comma (',') separated list of file extensions to be ignored)
threads = (number of concurrent threads for scan, one thread for one file, default to 4)
```

The file extensions should start with '.'. The extensions are case insensitive. By default, files with following extensions will be ignored:

```
.bmp, .gif, .ico, .png, .jpg, .mp3, .mp4, .wav, .avi, .rmvb, .mkv
```

The list you provide will override default list.

## Scanning Files on Upload

You may also configure Seafile to scan files for virus upon the files are uploaded. This only works for files uploaded via web interface or web APIs. Files uploaded with syncing or SeaDrive clients cannot be scanned on upload due to performance consideration.

You may scan files uploaded from shared upload links by adding the option below to `seahub_settings.py`:

```
ENABLE_UPLOAD_LINK_VIRUS_CHECK = True
```

Since Pro Edition 11.0.7, you may scan all uploaded files via web APIs by adding the option below to `seafile.conf`:

```
[fileserver]
check_virus_on_web_upload = true
```
