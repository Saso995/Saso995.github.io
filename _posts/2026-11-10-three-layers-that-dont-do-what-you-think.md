---
title: "Three Layers That Don't Do What You Think"
date: 2026-11-10 23:05:00 +0100
categories: [Security, Concepts]
tags: [endpoint-security, powershell, evasion, obfuscation, blue-team, offensive-security]
description: "Why obfuscation and encryption keep failing against a modern endpoint, and what that failure is actually telling you."
toc: true
---

> This is a concepts piece: the mental model, not a method. No code, no commands, no evasion recipe. It explains why certain intuitions about hiding a script are wrong, and stops short of how anyone turns that into a working bypass.
{: .prompt-warning }

Here is a question that sounds like it has an obvious answer and doesn't: if you want a script to slip past the antivirus on a modern Windows machine, what are you actually hiding it *from*? Most people, reaching for the problem for the first time, answer "from the antivirus" and start making the file harder to read. That instinct is the mistake, and it is worth dwelling on because the same mistake has a general shape far beyond security.

The useful way in is to notice that a defense is not one adversary. It is several, stacked, and each one asks a **different question** about your file. You can defeat one completely and the others will not even have noticed you tried.

| # | The question it asks |
|---|----------------------|
| 1 | What shape is it? |
| 2 | What's inside it? |
| 3 | What will it do? |
| 4 | What did it touch? |

Those four questions are asked by four different things, at four different moments, with four different powers. Map any evasion idea onto them and you can usually predict, in advance, which layer will shrug it off.

## 01 · Obfuscation answers the question nobody important is asking

The first idea everyone has is to change the **form**: rename variables, respell strings, inject junk, write numbers as arithmetic instead of digits. Done well, every copy of the script looks structurally unrelated to the last. There is no fixed pattern left to fingerprint.

This genuinely defeats *Question 1*, the cheapest layer of a defense: the static signature, which recognizes known-bad by shape. But that is the only question it answers. A layer that does not care what the file looks like, and instead asks what it is about to *do*, is completely untouched. You have spent real effort making yourself unrecognizable to the one observer that was never going to judge you on looks.

```text
your script → unrecognizable shape → static signature     → bypassed
your script → unrecognizable shape → runtime inspection   → unaffected
```

So the first principle, and it is the one that costs beginners the most time: **obfuscation is not evasion.** It is camouflage against a sensor that was already the weakest in the room.

## 02 · Encryption protects against the wrong adversary

The natural next move is to stop *hiding* the content and start making it genuinely *unavailable*: compress it, then encrypt it with a real key. Now someone who opens the file at rest recovers nothing. By every engineering yardstick this is a serious step up from obfuscation.

And against *Question 2*, the analyst reading your file on disk, it works. But it still answers the wrong adversary. Because whatever you wrap a payload in, **something has to unwrap it in order to run it**, and it has to do that on the target machine, in memory, at the moment of execution. Encryption buys you secrecy from the reader. It buys you nothing from the observer watching the process come alive.

```text
the analyst  → reads a file at rest    → encryption defeats this
the endpoint → watches a live process  → encryption is irrelevant here
```

Second principle: **encryption defends against the analyst, not against the endpoint.** The two get confused constantly because both are "the security," but they live at different times in the life of your file and only one of them can stop it running.

## 03 · The component in the middle

Both failures have a single cause, and naming it explains everything at once. Modern script hosts on Windows do not simply execute what they are handed. Before a script runs, the host passes its contents to a standard scanning interface that the operating system exposes to whatever antimalware is installed, and it passes it **resolved**: de-obfuscated, decrypted, as it will actually run. This is documented, deliberate platform behavior, not a trick anyone discovered.

Read that twice, because it inverts the whole problem. The layers you added were never hiding the payload from the endpoint. They were just *lengthening the path along which you hand the endpoint the payload yourself.* Obfuscation, encryption, compression: by the time it matters, all of it has been undone, and the finished article is submitted for inspection by your own code, on your behalf, as a condition of running.

```text
wrapped payload → your loader undoes every layer → resolved content → submitted for scan
```

This is the point where the honest version of the write-up stops. The next question, the one offensive security actually works on, is what you could do about a component that is shown the finished article. That has real answers, they are in the public literature, and I am deliberately not walking through them here, because the line between "understanding why a defense works" and "a reproducible method for defeating it" sits exactly at that question. The concepts above are the half worth publishing. The recipe is the half that isn't.

## 04 · The general lesson, which has nothing to do with Windows

Strip the security specifics away and what remains is a reasoning failure anyone can fall into. Two different, well-built attempts failed for the *same* reason, and that reason was invisible as long as the only thing coming back was the word "blocked." A single bit of feedback, pass or fail, is compatible with a hundred wrong theories, so you keep refining within whichever theory you started with. The instinct, each time, is to add another layer, when the number of layers was never the problem.

What breaks the loop is never a cleverer attempt. It is **higher-resolution feedback**: not "it failed" but *which* stage failed, with *which* error, in *what* order. The moment the failure is described precisely instead of just scored, the wrong assumption usually dies on its own. The decisive asset is rarely a better idea. It is a measurement fine enough to kill the idea you were already attached to.

## 05 · For the other side of the fence

If you defend Windows estates, the takeaway is that the layer that blocks and the layer that sees are not the same layer.

- **A control that runs inside the same process as the code it inspects is tamperable by anyone who already has execution in that process.** It is an excellent filter and a poor guarantee. If a check can in principle be influenced by the very thing it is meant to judge, it belongs inside a defense-in-depth stack, never alone at the top of it.
- **What an in-process payload can never reach is everything that lives elsewhere:** script-block logging at the engine level, process-creation telemetry, outbound-connection records. Those are off-host or out-of-process by design, and that is precisely why they are worth investing in.
- **The script still has to call home.** Beating a local check does nothing about the connection that then has to open to an external host on some port. In a monitored environment that egress is the hardest part to hide, and outbound anomaly detection routinely does more work than the endpoint scanner ever did.
- **Strange obfuscation is itself the signal.** Heavily encoded script execution does not look like normal administration. A policy that treats it as an event worth reviewing, rather than a binary to allow or block, catches the attempts that any single technical control would miss.

## The moral

> Obfuscation and encryption both fail against a modern endpoint for the same reason, and the reason has nothing to do with how well either is done. Each answers a question, "what shape is it," "what's inside it," that the layer actually in charge was never asking. The layer in charge asks what the code is about to do, and it is shown the real thing by design. Understand that and you understand both failures in one sentence, which is worth more than any single technique, on whichever side of the fence you sit.
{: .prompt-tip }

---

*A concepts piece drawn from offensive-security preparation in an isolated lab on hardware I own. It deliberately contains no code, commands, flags or implementation detail, and stops short of any method for defeating a security control: the aim is the reasoning, not a tool.*
