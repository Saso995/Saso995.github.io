---
title: "From low privilege to RCE in IUCLID 6: chaining CVE-2023-26089 and CVE-2023-26546"
date: 2026-10-16 10:00:00 +0200
categories: [Security Research, CVE]
tags: [iuclid, jwt, ssti, rce, freemarker, web]
image:
  path: /assets/img/posts/iuclid-rce/ssti-output.png
  alt: Command output returned inside a generated IUCLID report, confirming RCE.
---

This is a write-up of two vulnerabilities I reported in **IUCLID 6** (the chemical-data
application run by ECHA) that were subsequently assigned **CVE-2023-26089** and
**CVE-2023-26546**. Taken on their own, each is serious. Chained together, they turn an
ordinary low-privilege foothold into **remote code execution** on the server.

Tested on **IUCLID 6 v6.27.2** (build of 03/11/2022). Both issues are fixed from
[v6.27.6](https://iuclid6.echa.europa.eu/documents/1387205/1809530/note_v6.27.6.pdf/76545a65-e6be-6486-280a-7d7c3d2ad455?t=1677577170669)
onwards.

## TL;DR

- The application signs its JWTs with a **weak, guessable HMAC secret**. Recover it, and you
  can forge a token for any user, including `SuperUser`. That's **CVE-2023-26089**.
- The report-generation feature renders **user-supplied FreeMarker templates without a
  sandbox**, so a template can execute OS commands. That's **CVE-2023-26546**.
- The SSTI requires an authenticated session. The JWT bug *removes that precondition*, so the
  combined chain is effectively **unauthenticated RCE**.

## CVE-2023-26089: Account takeover via a weakly-signed JWT

A JWT's signature is the only thing binding its claims to the server's authority. It holds up
only while the signing secret stays secret and is hard to guess. IUCLID 6 used a weak HMAC
secret, weak enough that Burp Suite flagged and cracked it immediately after authentication.

![Burp Suite flags the weak HMAC signing secret and recovers the key.](/assets/img/posts/iuclid-rce/jwt-weak-secret.png)
_Burp Suite flags the weak HMAC secret; the recovered key is shown._

Once you know the secret, the signature guarantee is gone. You can mint a token with arbitrary
claims and sign it so it verifies. Take a valid token as a template, change the `sub` claim to
your target (here, `SuperUser`), adjust `exp`, and re-sign:

![The forged JWT with a modified sub claim, re-signed with the recovered secret.](/assets/img/posts/iuclid-rce/jwt-forged.png)
_The forged token: `sub` changed and re-signed with the cracked secret._

Submit an authentication request carrying the crafted token and the server hands you a session
as that user:

![Logged in as SuperUser after submitting the forged token.](/assets/img/posts/iuclid-rce/jwt-ato.png)
_Full account takeover: authenticated as `SuperUser`._

Because one shared weak secret underlies every token, this affects **all** accounts on the
instance.

> **Proposed CVSS 3.1:** `AV:N/AC:L/PR:N/UI:N/S:U/C:H/I:H/A:H`, 9.8 (Critical).
{: .prompt-danger }

## CVE-2023-26546: RCE via Server-Side Template Injection

IUCLID lets users upload custom `.ftl` report templates and renders them server-side with
**FreeMarker**. FreeMarker exposes a utility class, `freemarker.template.utility.Execute`,
whose `?new()` built-in instantiates it, and invoking it runs an arbitrary shell command.
With no restricted `TemplateClassResolver` in place, anyone who controls a template controls
the host.

The payload is a one-liner. Any of these run `whoami`:

```freemarker
<#assign ex = "freemarker.template.utility.Execute"?new()>${ ex("whoami")}
[#assign ex = 'freemarker.template.utility.Execute'?new()]${ ex('whoami')}
${"freemarker.template.utility.Execute"?new()("whoami")}
```

Upload it as a report template under **Dashboard → Report**:

![The malicious .ftl template uploaded to a report.](/assets/img/posts/iuclid-rce/ssti-template.png)
_The malicious template, uploaded like any ordinary report template._

Then generate a report from an article using that template (**Dashboard → Articles → Generate
Report**), download it, and the command output is sitting inside the generated document:

![Command output embedded in the generated report, confirming code execution.](/assets/img/posts/iuclid-rce/ssti-output.png)
_The generated report contains the output of `whoami`: code execution on the server._

> **Proposed CVSS 3.1 (standalone, authenticated):** `AV:N/AC:L/PR:L/UI:N/S:U/C:H/I:H/A:H`, 8.8 (High).
{: .prompt-danger }

## Chaining them

The SSTI needs a session with access to the report feature. On its own that's a privilege
requirement. But the JWT flaw lets you forge a `SuperUser` token from nothing, so you satisfy
the SSTI's precondition without any legitimate access at all.

```
weak JWT secret  ──▶  forge SuperUser token  ──▶  upload malicious .ftl  ──▶  generate report  ──▶  RCE
```

Chained, the effective privilege requirement drops to **none**:

> **Proposed CVSS 3.1 (chained):** `AV:N/AC:L/PR:N/UI:N/S:U/C:H/I:H/A:H`, 9.8 (Critical).
{: .prompt-danger }

## Fixes

- **JWT:** use a strong, high-entropy secret, unique per installation; never ship a default.
  Validate claims server-side and bind tokens to server-side session state.
- **SSTI:** run FreeMarker with a restrictive `TemplateClassResolver` (e.g.
  `ALLOWS_NOTHING_RESOLVER`) and block `?new()` so classes like `Execute` are unreachable.
  Don't render user-supplied templates in a privileged, unsandboxed context.

Both were remediated by ECHA in IUCLID 6 v6.27.6.

---

*Disclosure note: this research was reported to the vendor and the issues are patched. CVSS
vectors above are my own retrospective estimates and were not part of the original reports.*
