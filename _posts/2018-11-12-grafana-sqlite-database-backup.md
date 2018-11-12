---
layout: post
title: grafana sqlite database backup
subtitle: 
tags: [grafana, backup]
---

Backup dashboards and user list if using grafana sqlite database.

```bash
sqlite3 /var/lib/grafana/grafana.db .dump > /backup_destination_directory/grafana.db.dump
```
