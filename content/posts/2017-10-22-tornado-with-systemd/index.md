---
title: "Tornado with systemd"
date: 2017-10-22
categories: 
  - "tech"
---

Unlike Django or Flask, tornado is not WSGI based. It also ships with it's own HTTP server and running the tornado server in production is not very much different from what you do while running it while developing. You may want to run the server in background though.

The systemd config file for tornado server that i am currently using is:

_\[Unit\]_ _Description=Tornado server service file_ _After=rabbitmq-server.target_

_\[Service\]_ _ExecStart=/usr/bin/python /path/to/tornado/server/main/method.py_ _Restart=on-abort_

_\[Install\]_ _WantedBy=multi-user.target_

For my use case, it should start after rabbitmq-server has started. After putting this script in _/etc/systemd/system/_ , you can start the server by:

_sudo systemctl start servicename.service_

It's status can be checked using:

_sudo systemctl status servicename.service_

For people with trust issues, you can use curl or visit the site itself to reconfirm. In my case, the tornado server was a websocket server and not http. So, i had to put some additional parameters with curl which i got from [here](https://gist.github.com/htp/fbce19069187ec1cc486b594104f01d0).

_➜ curl --include \\_ _\--no-buffer \\_ _\--header "Connection: Upgrade" \\_ _\--header "Upgrade: websocket" \\_ _\--header "Host: http://127.0.0.1:8080" \\_ _\--header "Origin: http://127.0.0.1:8080" \\_ _\--header "Sec-WebSocket-Key: SGVsbG8sIHdvcmxkIQ==" \\_ _\--header "Sec-WebSocket-Version: 13" \\_ _http://127.0.0.1:8080/ws_

That's it.
