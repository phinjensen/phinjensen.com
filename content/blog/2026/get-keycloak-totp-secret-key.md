---
title: How to get an OTP secret key out of a Keycloak database
author: Phineas Jensen
date: 2026-01-03
description: "Retrieving the key—The motivation"
tags:
  - keycloak
  - journal
---

TL;DR:

```bash
psql keycloak -c "COPY (SELECT secret_data::jsonb->>'value' FROM credential WHERE user_id = (SELECT id FROM user_entity WHERE email = '<email of account>') AND type = 'otp') TO stdout" | tr -d '\n' | base32
```

I lost my phone and wanted to add my 2FA for a Keycloak instance back to [Aegis](https://getaegis.app/) (yes, my backup habits/system could be improved). I had an admin account in the Keycloak instance, but couldn't access it because the 2FA was on my phone, but I did still have access to the database server. Using the above snippet, I was able to find the secret key and plug it into Aegist manually. Worked like a charm!
