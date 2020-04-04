---
title: "BHUSA19: Flying A False Flag - Advanced C2, Trust Conflicts, and Domain Takeover"
date: 2019-10-14T22:18:00+07:00
author: "P. Boonyakarn"
description: "This a summary for BlackHat USA 2019 talk, Flying A False Flag - Advanced C2, Trust Conflicts, and Domain Takeover, by Nick Landers from Silent Break Security. The presentation of this talk is available here."
---

This a summary for BlackHat USA 2019 talk, **Flying A False Flag - Advanced C2, Trust Conflicts, and Domain Takeover**, by Nick Landers from Silent Break Security. The presentation of this talk is [available here](https://i.blackhat.com/USA-19/Wednesday/us-19-Landers-Flying-A-False-Flag-Advanced-C2-Trust-Conflicts-And-Domain-Takeover.pdf).

- An analogy of command & control technology (C2) comprises the same functions as in the software model: channel selection, redundancy, obfuscation, serialization, encryption, and trust.
- From its basic objective, C2 responsible for the execution of an arbitrary command over a defined channel. There are many techniques for execution and channels which can be mixed up to create an ideal C2.
- Categories of techniques available as a strategy of execution are including:
    - **Orientation** can be a reverse connection from an implant to a listening server or a bind-style where an implant listening for a connection from an operator.
    - **Interval** is an implant sends beacons to a server as a keep-alive mechanism in a specific timeframe. Noisy but responsive.
    - **Distribution**: is a technique that requires more than one C2 servers that an implant will communication with
    - **Failover** is a technique to resume C2 communication when the main C2 server is down or unresponsive. Usually happens with Distribution techniques.
    - **Routing** is a technique focusing on obfuscate paths to the C2 server by routing traffic through 3rd party servers, or bastion when there are distributed C2 servers available. Routing can also combine with the Orientation technique when an implant connects back (reverse) to 3rd party server and an operator connect to (bind) that server for command and control. This is a stealth mode for C2.
- There are many mediums or channels to transmit C2 data
    - **Sockets** is a responsive and simple way. It can be encrypted and separated in a chunk.
    - **HTTP** is a common at the perimeter, already has reliability, encoded binary data isn't rare and has good accessibility. It also has many possibilities and good properties to masquerade data into a payload such as the use of POST requests for limited logging, embeds in special headers to avoid inspection, communicates with look-a-like or expired domain for a good reputation, etc.
    - **HTTP pipelining** is another way to hide actual traffic with a benign traffic. Interesting alternative to domain fronting.
    - **HTTP + WebSocket** Less inspection but gateway support may be limited.
    - **HTTP/2** Less inspection but more limited support that the WebSocket solution. Support binary data so data encoding may not need.
    - **DNS** Limited transfer size and simple to detect in various ways (volume, name length, unique subdomains). Consider using for heartbeats or logic transitions, or implementing with DoT, DoH and DNSSEC.
    - **ICMP** Very popular in the wild and easy to detect in various ways (entropy, mismatched, size). Consider using alternative ICMP codes, reduce payload size, use as heartbeats or with binary lookup tables.
- In modern days, many cloud applications are allowed to use within an organization's environment and are explicitly trusted. This trust can be abused to use as a channel such as C2 communication over social networks or chat applications. These channels have already been used by [actual](https://blog.talosintelligence.com/2017/04/introducing-rokrat.html) [threat](https://www2.fireeye.com/rs/848-DID-242/images/rpt-apt29-hammertoss.pdf) [actors](https://github.com/PaulSec/twittor).
    - Twitter: twittor
    - Multi social network: Social-media-c2
    - Slack: SlackShell, c2s, slack-c2bot
    - Skype: skype-dev-bots
    - Gmail: Gcat, Gdog
    - Office 365: MWR Labs
    - GitHub: canisrufus
    - Active Directory: harmj0y
    - MSSQL: PowerUpSQL / NetSPI
    - File Shares: outflank
    - Wikipedia: wikipedia-c2
- Cloud services also have the capability to be used as C2 due to multi-features available such as CDN endpoints, serverless architectures, file hosting, message queues, and VPNs.
- One important issue for cloud services in a defender's perspective is trust boundaries on dynamic assets. For example, domain fronting, file hosting over simple storage service, C2 with serverless code.
- Another important issue is domain takeover which can be done in various ways and tools.