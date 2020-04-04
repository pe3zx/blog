---
title: "Short Summary of NIST SP 800-63B on Strength of Memorized Secrets"
date: 2018-12-07T16:05:00+07:00
author: "P. Boonyakarn"
---

According to [NIST SP 800-63B](https://pages.nist.gov/800-63-3/sp800-63b.html),

- On the topic about required password complexity based-on composition rules, the publication doesn't exhibit its motivation to deprecate using of composition rules. Although, it presents statistically counterproductive effects when users are forced to follow these composition rules.
- The counterproductive effects can be explained that, for example, they tend to use more predictable password which poorly followed the composition rules or they tend to write down or store electronically in an unsafe manner because they couldn't remember the password.
- "Users should be encouraged to make their passwords as lengthy as they want..."
- Utilize other preventive mechanisms including, blacklists, secure hashed storage, and rate limiting instead of solely relies on policies and users.
- Threat modeling is essential. Without this process, you will end up having non-sense policy.