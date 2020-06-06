---
layout: post
title: migrate caldav calender entries
subtitle: 
tags: [caldav, webdav, sogo, nextcloud]
---

How to copy calendars from one service to another? 
In this case from [sogo](https://sogo.nu/) (which is still open source but now paid for releases) to [nextcloud](https://nextcloud.com/).
As source I use the sogo postgres database and copy the calendar entries using [cadaver](http://webdav.org/cadaver/) as webdav client to nextclouds webdav interface.


For caldav login create a ~/.netrc file.
```
machine nextcloud.domain.net login nextcloud_user password nextcloud_password
```

Choose calenders to be migrated.
```sql
select c_path2, c_foldername, c_location, c_folder_type from sogo_folder_info order by c_folder_type, c_path2;
```


Add all calendars which should be migrated to the calendars variable. Deleted entries will be skipped.

```bash
#!/bin/bash

declare -A calendars=(["sogo_source_calendar_db_table"]="nextcloud_destination_calendar")

src_db=sogo
dest_url=http://nextcloud.domain.net/remote.php/dav/calendars
username=nextcloud_user

for cal in "${!calendars[@]}" ; do
  src=$cal
  dest=${calendars[$cal]}

  echo "Export table $src"
  count=`sudo -u postgres psql -d sogo -t -A -c "select count(c_content) from $src where c_deleted is null"`
  echo "No of items to export: $count"
  mkdir -p $dest
  cd $dest
  for((i=0;i<$count;i++)); do
    sudo -u postgres psql -d $src_db -t -A -c "select c_content from $src where c_deleted is null limit 1 offset $i" > $(cat /proc/sys/kernel/random/uuid)-imported.ics
  done
  echo "No of items exported: $(ls | wc -w)"
  for f in *.ics; do sed -i 's/COUNT=0;//g' $f; done # fix reocurring event parameter

  echo "Import $src to calendar $dest"
  echo -e "cd $username/$dest\nmput *" | cadaver $dest_url

  cd ..
done
```

References
* <https://uriesk.wordpress.com/2015/02/13/backup-your-caldav-calendar-with-cadaver/>
* <https://sites.google.com/a/case.edu/hpcc/installed-software/file-transfer/cadaver-using-box-hpc>
* <https://relentlesscoding.com/2017/10/15/import-contacts-vcards-into-nextcloud/>
