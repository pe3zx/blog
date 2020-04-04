---
title: "How to Prepare Before the Compromise - A Summary From BHIS Talk"
date: 2019-10-20T21:33:00+07:00
author: "P. Boonyakarn"
description: "If you have enrolled in SEC540 and/or have experience with NIST SP800-61, you'd be familiar with the following details about what we need to prepare to handle and respond to an incident, as an incident handler/responder. By the way, I found this talk by John Strand from Black Hills Information Security, How to Prepare Before the Compromise, has a broader scope, not just an incident handler/responder but also a person who may involve with the situation."
---

If you have enrolled in SEC540 and/or have experience with NIST SP800-61, you'd be familiar with the following details about what we need to prepare to handle and respond to an incident, as an incident handler/responder. By the way, I found this talk by John Strand from Black Hills Information Security, [How to Prepare Before the Compromise](https://www.youtube.com/watch?v=V-3-RGsdqpM), has a broader scope, not just an incident handler/responder but also a person who may involve with the situation.

So, I decided to summarize the interesting points with my opinion and hope that it would be useful for anyone who is able to realize that they do really need to proactively prepare for an incident.

- **Start with a good mindset**. There's no 100% for cybersecurity. So, there's no such thing that will detect and prevent an incident from happening. Uncertainty is an essential property of our everyday life and you need to be ready for a bad thing, be able to adapt to the situation and do something that helps yourself come back stronger than before.
- **Useless things**. Most organizations already have capabilities to detect, prevent and respond to an incident, but these capabilities are less known to be used or be ready for managing situations. We have to focus on our environment, config it and/or practice for incident remediation which can be done by table-top exercise and adversary simulation/emulation.
- **Roles, responsibilities, and authorizations**. Make it clear and actionable.
- **Gain trust**. People tend to believe and follow suggestions and recommendations by a confident and rationale person.
- Prepare **a baseline** when the system is operating normally, or at least know a baseline of the system, its roles, software, and activities.
- **SIEM** gonna be useless things if it doesn't config properly, feeding and processing unnecessary information. Use [JPCERT-CC's Tool Analysis Result Sheet](https://jpcertcc.github.io/ToolAnalysisResultSheet/) to evaluate SIEM's readiness against known attack techniques and tools.
- **Segmentation** and **isolation**. Learn how to use a tool like the host-based firewalls to create and control a segment of network. At the same time, learn how to use this to isolate the segment when an incident is happening. Because there **are many levels and strategies of isolation**, such as network-level via switch and router and strategies like just lock the user out or create a contained environment, learn and practice these as well.
- **Crisis management**. What if you got hacked. Who are you gonna report to and how? What's about public announcements and law enforcement?