---
title: "Enhancing Host File with PortProxy"
date: 2020-02-29T10:58:00+07:00
author: "P. Boonyakarn"
description: "The host file on Windows can be used to provide a simple name resolution mechanism but with a limited option. For example, it's impossible to specify a destination port if you want to access the destination service with a custom port. Only an IP address and a name can be specified in the host file."
---

The host file on Windows can be used to provide a simple name resolution mechanism but with a limited option. For example, it's impossible to specify a destination port if you want to access the destination service with a custom port. Only an IP address and a name can be specified in the host file.

However, Windows also has less known built-in feature which can be used to help tunnel traffic and make it more configurable. This feature is called `PortProxy` and it exists behind the CLI program called netsh. Its capability on tunneling is comparable with `netcat`/`nc` program family.

In this scenario, we have a Linux box is configured to provide an HTTP server on a not well-known port and a Windows client that needs to access the server via a custom domain name. We can use PortProxy to tunnel the connection with the host file help by the following steps:

- Add a line in the host file to map an IP address to a domain name. The IP address can be anything in a loopback interface (`127.xxx.xxx.xxx`).
- Add a PortProxy configuration with `netsh`:
    - `listenport` is a port that PortProxy will listen in the client-side
    - `listenaddress` must be matched with the IP address on the host file
    - `connectport` is the Linux box HTTP server's port
    - `connectaddress` is the Linux box IP address

```
netsh interface portproxy add v4tov4 listenport=80 listenaddress=127.xxx.xxx.xxx connectport=8888 connectaddress=the.linux.box.ip
```

There are a plenty of commands to help manage and custom `PortProxy` rule. You can find and try in with `netsh interface portproxy`.