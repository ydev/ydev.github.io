---
layout: post
title: nginx maintenance page using simple cookie
subtitle: 
tags: [nginx, maintenance]
---


```bash
location /jira {
    if (-f /var/maintenance$cookie_opsteam) {
        return 503
    }
    ...
    proxy_pass http://internalserver.com:8080
}

Error_page 503 @maintenance
Location @maintenance {
    Root /var/www/html
    Rewrite ^(.*)$ /maintenance.html break;
}
```

Enable maintenance page
```
touch /var/maintenance
```

Disable maintenance page
```
rm /var/maintenance
```

Enable bypass (browser console)
```
document.cookie = "opsteam=1;path=/;secure=true"
```

Disable bypass
```
document.cookie = "opsteam=;path=/;secure=true"
```
