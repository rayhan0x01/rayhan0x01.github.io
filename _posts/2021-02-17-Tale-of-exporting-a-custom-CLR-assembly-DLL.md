---
layout: post
title:  "Tale of exporting a custom CLR assembly DLL from an SQL Server"
date:   2021-02-17
categories: CTF
thumbnail: /img/dll.webp
tags: dsp_desc_bind,memory-allocation-failure,sqsh,assembly_files,redteam
---
After spending a whole day trying to solve this error called `dsp_desc_bind: Memory allocation failure for column #1` on `SQSH` I decided to do a writeup on how to export custom CLR assembly DLL from an SQL Server when you have a low priv SQL user and don't have execute permission on `sp_OA*` objects to dump the DLL file to file-system.

Since this is part of an active CTF challenge, I'll redact anything thats specific to the challenge but I realize many may start looking around like me when they encounter the error and this should help them! So what we have is an SQSH shell:

```sh
sqsh -s servername -U username -P password
```

What we are trying to do is extract a custom [CLR assembly DLL](https://docs.microsoft.com/en-us/sql/relational-databases/clr-integration/assemblies-database-engine?view=sql-server-ver15) file in base64 which is stored in `sys.assembly_files` as `VARBINARY` data type.

The issue we are facing on latest SQSH (which depends on latest FreeTDS libraries):

```sql
1> SELECT name,content FROM [SERVER\INSTANCE].DBNAME.sys.assembly_files where assembly_id=65536 FOR XML AUTO, BINARY BASE64
2> go
dsp_desc_bind: Memory allocation failure for column #1
```

After many failed attempts to solving this issue I went back to the only [StackOverflow Question](https://stackoverflow.com/questions/50252358/sqsh-gives-dsp-desc-bind-memory-allocation-failure-for-column-1) about this where the author later explained why it occurs and how to solve it by himself.

The solution given was editing `/etc/freetds/freetds.conf` and adding a section:

```sh
[YourDbHostname]
      host = localhost
      port = 1433
      tds version = 8.0
```

However this did not work for me on the latest `SQSH` which uses latest `FreeTds` libraries.

The author also mentioned downgrading the `FreeTds` package to `0.91.6`, which he had working on `Ubuntu:17.10`.

Now in need of an end of the line outdated version of Ubuntu to solve my problem, I turned to my good friend `Docker` to save me the hassle of installing Ubuntu the traditional way on a VirtualBox. Partly the reason for writing this post was to show how docker can be so much handy in situations like this where we need a full framework or OS just for a specific purpose.

I simply pulled the needed docker image and ran the container:

```sh
docker pull ubuntu:17.10
docker run -it --network host ubuntu:17.10 /bin/bash
```

Since Ubuntu 17.10 reached End Of LIFE, the sources shipped with it no longer works so we have to change it:

```sh
sed -i 's/archive.ubuntu.com/old-releases.ubuntu.com/g' /etc/apt/sources.list
sed -i '/http:\/\/security/d' /etc/apt/sources.list
apt update
```

Now we can search for `freetds` on `apt` and see the version `0.91.6` is presented on this distro:

```sh
$ apt search freetds
Sorting... Done
Full Text Search... Done
freetds-bin/artful 0.91-6.1build2 amd64
  FreeTDS command-line utilities
.. snip ..
```

Now we install `sqsh` which would install the `freetds` package as well, and configure:

```sh
# Oh and we need vim too, there's literally no nano/vi/vim on the docker image
$ apt install sqsh vim 
$ vim /etc/freetds/freetds.conf

# Edited last example entry to my needs
[servername]
	host = SERVER_IP
	port = 1433
	tds version = 7.0
```

On this distro the `freetds.conf` example entry will contain `tds version = 5.0` which doesn't work. Changing it to either 7.0 or 8.0 both works.

Now we can login via `sqsh` and read assembly DLL VARBINARY data as base64:

```sql
sqsh -s servername -U username -P password

1> SELECT name,content FROM [SERVER\INSTANCE].DBNAME.sys.assembly_files where assembly_id=65536 FOR XML AUTO, BINARY BASE64
2> go

[DLL content is presented in base64 chunks]
```



These custom CLR assemblies often contain passwords and can be decompiled locally.

If you get SQL server credentials and don't have the permissions to use `xp_cmdshell` or `reconfigure`, its also worth checking Linked Servers with `sp_linkedservers` and searching for custom CLR assembly DLL files in all of them to find something useful.