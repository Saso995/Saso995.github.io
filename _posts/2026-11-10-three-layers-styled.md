---
title: "Three Layers That Don't Do What You Think"
date: 2026-11-10 23:05:00 +0100
categories: [Security, Concepts]
tags: [endpoint-security, powershell, evasion, obfuscation, blue-team, offensive-security]
description: "Why obfuscation and encryption keep failing against a modern endpoint, and what that failure is actually telling you."
---

<link rel="preconnect" href="https://fonts.googleapis.com">
<link rel="preconnect" href="https://fonts.gstatic.com" crossorigin>
<link rel="stylesheet" href="https://fonts.googleapis.com/css2?family=Bricolage+Grotesque:opsz,wght@12..96,500;12..96,700;12..96,800&family=Spectral:ital,wght@0,400;0,500;0,600;1,400&family=JetBrains+Mono:wght@400;500;700&display=swap">

<style>
  .tl{
    --surface:#fbf8f3; --surface-2:#ece3d8; --code-bg:#f0e8dd;
    --ink:#231a16; --muted:#6c5d53; --faint:#8a7a6f; --border:#dcd0c3; --border-strong:#cab9a8;
    --ember:#bb3720; --human:#9a6412; --claude:#246d8f; --good:#2c7a4b; --steel:#45586b;
    --human-soft:#9a641222; --claude-soft:#246d8f22; --ember-soft:#bb372018; --steel-soft:#45586b1a;
    --shadow:0 1px 2px rgba(35,26,22,.06),0 8px 26px rgba(35,26,22,.07);
    color:var(--ink);font-family:"Spectral",Georgia,serif;font-size:18px;line-height:1.65;
  }
  html[data-mode="dark"] .tl, .tl[data-dark]{
    --surface:#1c1613; --surface-2:#271f1b; --code-bg:#120d0c;
    --ink:#ece2da; --muted:#a2948a; --faint:#7d6f66; --border:#342a25; --border-strong:#473a33;
    --ember:#ea5a3c; --human:#e2b259; --claude:#71bbdb; --good:#5ec489; --steel:#8fa7bc;
    --human-soft:#e2b2591e; --claude-soft:#71bbdb1e; --ember-soft:#ea5a3c1c; --steel-soft:#8fa7bc1a;
    --shadow:0 1px 2px rgba(0,0,0,.4),0 10px 30px rgba(0,0,0,.45);
  }
  .tl *{box-sizing:border-box}
  .tl h1,.tl h2,.tl h3{font-family:"Bricolage Grotesque",system-ui,sans-serif;
    text-wrap:balance;letter-spacing:-.01em;line-height:1.12;margin:0;border:0}
  .tl a{color:var(--claude)}
  .tl .eyebrow{font-family:"JetBrains Mono",monospace;font-size:12.5px;letter-spacing:.22em;
    text-transform:uppercase;color:var(--faint);font-weight:500}

  .tl .top{border:1px solid var(--border);border-radius:16px;padding:34px 30px;margin:8px 0 30px;
    background:radial-gradient(120% 100% at 85% -10%,var(--ember-soft),var(--surface) 60%)}
  .tl .series{display:flex;align-items:center;gap:12px;margin-bottom:20px}
  .tl .series .dot{width:7px;height:7px;border-radius:50%;background:var(--ember);box-shadow:0 0 0 4px var(--ember-soft)}
  .tl .top h2{font-size:clamp(28px,5vw,44px);font-weight:800}
  .tl .top h2 .vs{color:var(--ember)}
  .tl .dek{font-size:clamp(18px,2.4vw,21px);color:var(--muted);margin:16px 0 0;max-width:46ch}

  .tl section{padding:34px 0;border-bottom:1px solid var(--border);border-top:0;margin:0}
  .tl section:last-of-type{border-bottom:0}
  .tl h2.sec{font-size:clamp(23px,4vw,31px);font-weight:700;margin-bottom:18px}
  .tl h2.sec .n{font-family:"JetBrains Mono",monospace;font-size:14px;color:var(--ember);font-weight:700;margin-right:10px}
  .tl p{margin:0 0 16px} .tl p:last-child{margin-bottom:0}
  .tl strong{font-weight:600;color:var(--ink)} .tl em{font-style:italic}
  .tl .lead{font-size:20px;line-height:1.55}

  .tl .facts{display:grid;grid-template-columns:repeat(auto-fit,minmax(150px,1fr));gap:1px;
    background:var(--border);border:1px solid var(--border);border-radius:12px;overflow:hidden;margin:22px 0}
  .tl .facts div{background:var(--surface);padding:14px 16px}
  .tl .facts .k{font-family:"JetBrains Mono",monospace;font-size:11px;letter-spacing:.14em;text-transform:uppercase;color:var(--faint)}
  .tl .facts .v{font-family:"Bricolage Grotesque",sans-serif;font-size:16px;margin-top:6px;font-weight:700}

  .tl .diagram{background:var(--code-bg);border:1px solid var(--border);border-radius:11px;padding:16px 18px;margin:16px 0;overflow-x:auto}
  .tl .diagram .row{font-family:"JetBrains Mono",monospace;font-size:13px;line-height:2;white-space:nowrap}
  .tl .diagram .lbl{font-family:"JetBrains Mono",monospace;font-size:11px;letter-spacing:.14em;text-transform:uppercase;color:var(--faint);margin-bottom:8px}
  .tl .chip{display:inline-block;border:1px solid var(--border-strong);border-radius:6px;padding:2px 9px;background:var(--surface)}
  .tl .chip.data{border-color:var(--claude);color:var(--claude);background:var(--claude-soft)}
  .tl .chip.code{border-color:var(--human);color:var(--human);background:var(--human-soft)}
  .tl .chip.guard{border-color:var(--steel);color:var(--steel);background:var(--steel-soft)}
  .tl .arr{color:var(--faint);padding:0 6px}
  .tl .verdict{font-family:"JetBrains Mono",monospace;font-size:12px;font-weight:700;letter-spacing:.1em;text-transform:uppercase}
  .tl .verdict.block{color:var(--ember)} .tl .verdict.pass{color:var(--good)}

  .tl .card{border:1px solid var(--border-strong);border-radius:14px;padding:24px 26px;background:var(--surface);box-shadow:var(--shadow)}
  .tl .card.ember{border-left:3px solid var(--ember)}
  .tl .card h3{font-size:clamp(21px,3.2vw,27px);font-weight:800;margin:10px 0 12px}
  .tl .card .lbl{display:block;margin-bottom:2px}

  .tl .defend{background:var(--steel-soft);border:1px solid var(--steel);border-radius:14px;padding:24px 26px}
  .tl .defend h3{font-size:21px;font-weight:700;margin:10px 0 14px}
  .tl .defend ul{margin:0;padding-left:20px} .tl .defend li{margin-bottom:10px}

  .tl .note{display:flex;gap:14px;align-items:flex-start;border:1px dashed var(--border-strong);
    border-radius:12px;padding:16px 18px;margin:0 0 24px;background:var(--surface)}
  .tl .note .m{font-family:"JetBrains Mono",monospace;font-size:11px;letter-spacing:.14em;text-transform:uppercase;color:var(--ember);font-weight:700;flex:none;padding-top:3px}
  .tl .note p{font-size:15px;color:var(--muted);margin:0}
  .tl .foot{color:var(--faint);font-size:14px;padding-top:24px}
</style>

<div class="tl" markdown="0">

  <div class="top">
    <div class="series"><span class="dot"></span><span class="eyebrow">Field Notes &middot; Concepts</span></div>
    <h2>Three layers that<span class="vs"> don't </span>do what you think</h2>
    <p class="dek">Why obfuscation and encryption keep failing against a modern endpoint, and what that failure is actually telling you.</p>
  </div>

  <div class="note">
    <span class="m">scope</span>
    <p>This is a concepts piece: the mental model, not a method. No code, no commands, no evasion recipe. It explains why certain intuitions about hiding a script are wrong, and stops short of how anyone turns that into a working bypass.</p>
  </div>

  <section>
    <p class="lead">Here is a question that sounds like it has an obvious answer and doesn't: if you want a script to slip past the antivirus on a modern Windows machine, what are you actually hiding it <em>from</em>? Most people answer "from the antivirus" and start making the file harder to read. That instinct is the mistake, and it has a general shape far beyond security.</p>
    <p>A defense is not one adversary. It is several, stacked, and each one asks a <strong>different question</strong> about your file. You can defeat one completely and the others will not even notice you tried.</p>
    <div class="facts">
      <div><div class="k">Question 1</div><div class="v">What shape is it?</div></div>
      <div><div class="k">Question 2</div><div class="v">What's inside it?</div></div>
      <div><div class="k">Question 3</div><div class="v">What will it do?</div></div>
      <div><div class="k">Question 4</div><div class="v">What did it touch?</div></div>
    </div>
    <p>Four questions, four different things, four different moments. Map any evasion idea onto them and you can usually predict which layer will shrug it off.</p>
  </section>

  <section>
    <h2 class="sec"><span class="n">01</span>Obfuscation answers the question nobody important is asking</h2>
    <p>The first idea everyone has is to change the <strong>form</strong>: rename variables, respell strings, inject junk. Done well, every copy looks structurally unrelated to the last. There is no fixed pattern left to fingerprint.</p>
    <p>This defeats <em>Question 1</em>, the cheapest layer: the static signature, which recognizes known-bad by shape. But that is the only question it answers. A layer that asks what the file is about to <em>do</em> is completely untouched. You made yourself unrecognizable to the one observer that was never going to judge you on looks.</p>
    <div class="diagram">
      <div class="lbl">what obfuscation changes</div>
      <div class="row"><span class="chip">your script</span><span class="arr">&rarr;</span><span class="chip data">unrecognizable shape</span><span class="arr">&rarr;</span><span class="chip guard">static signature</span><span class="arr">&rarr;</span><span class="verdict pass">bypassed</span></div>
      <div class="row"><span class="chip">your script</span><span class="arr">&rarr;</span><span class="chip data">unrecognizable shape</span><span class="arr">&rarr;</span><span class="chip guard">runtime inspection</span><span class="arr">&rarr;</span><span class="verdict block">unaffected</span></div>
    </div>
    <p>First principle, the one that costs beginners the most time: <strong>obfuscation is not evasion.</strong> It is camouflage against a sensor that was already the weakest in the room.</p>
  </section>

  <section>
    <h2 class="sec"><span class="n">02</span>Encryption protects against the wrong adversary</h2>
    <p>The natural next move is to make the content genuinely <em>unavailable</em>: compress it, then encrypt it with a real key. Someone who opens the file at rest recovers nothing. By every engineering yardstick this is a serious step up.</p>
    <p>And against <em>Question 2</em>, the analyst reading your file on disk, it works. But whatever you wrap a payload in, <strong>something has to unwrap it in order to run it</strong>, on the target machine, in memory, at the moment of execution. Encryption buys you secrecy from the reader. It buys you nothing from the observer watching the process come alive.</p>
    <div class="diagram">
      <div class="lbl">two adversaries, different powers</div>
      <div class="row"><span class="chip guard">the analyst</span><span class="arr">&rarr;</span>reads a file at rest<span class="arr">&rarr;</span><span class="chip data">encryption defeats this</span></div>
      <div class="row"><span class="chip guard">the endpoint</span><span class="arr">&rarr;</span>watches a live process<span class="arr">&rarr;</span><span class="chip code">encryption is irrelevant</span></div>
    </div>
    <p>Second principle: <strong>encryption defends against the analyst, not against the endpoint.</strong> Both are "the security," but they live at different times in the life of your file, and only one can stop it running.</p>
  </section>

  <section>
    <h2 class="sec"><span class="n">03</span>The component in the middle</h2>
    <p>Both failures have one cause. Modern script hosts on Windows do not simply execute what they are handed. Before a script runs, the host passes its contents to a standard scanning interface the OS exposes to the installed antimalware, and it passes it <strong>resolved</strong>: de-obfuscated, decrypted, as it will actually run. Documented, deliberate platform behavior.</p>
    <p>Read that twice, because it inverts the problem. The layers you added were never hiding the payload. They were <em>lengthening the path along which you hand the endpoint the payload yourself.</em> By the time it matters, everything has been undone and the finished article is submitted for inspection by your own code, as a condition of running.</p>
    <div class="diagram">
      <div class="lbl">why both ideas fail together</div>
      <div class="row"><span class="chip data">wrapped payload</span><span class="arr">&rarr;</span><span class="chip">loader undoes every layer</span><span class="arr">&rarr;</span><span class="chip code">resolved content</span><span class="arr">&rarr;</span><span class="chip guard">submitted for scan</span></div>
    </div>
    <p>This is where the honest version stops. The next question, what you do about a component shown the finished article, has real answers in the public literature, and I am deliberately not walking through them, because the line between "understanding why a defense works" and "a reproducible method for defeating it" sits exactly there. The concepts above are the half worth publishing. The recipe is the half that isn't.</p>
  </section>

  <section>
    <h2 class="sec"><span class="n">04</span>The general lesson, which has nothing to do with Windows</h2>
    <div class="card ember">
      <span class="eyebrow lbl" style="color:var(--ember)">The crux &middot; pass/fail is not enough signal to steer by</span>
      <h3>You'll optimize the wrong variable, elegantly, as long as the feedback stays coarse</h3>
      <p>Two different, well-built attempts failed for the <em>same</em> reason, and that reason was invisible as long as the only thing coming back was "blocked." A single bit of feedback is compatible with a hundred wrong theories, so you keep refining within whichever one you started from. The instinct is to add another layer, when the number of layers was never the problem.</p>
      <p style="margin-top:12px">What breaks the loop is never a cleverer attempt. It is <strong>higher-resolution feedback</strong>: not "it failed" but <em>which</em> stage, with <em>which</em> error, in <em>what</em> order. The decisive asset is rarely a better idea. It is a measurement fine enough to kill the idea you were already attached to.</p>
    </div>
  </section>

  <section>
    <h2 class="sec"><span class="n">05</span>For the other side of the fence</h2>
    <div class="defend">
      <span class="eyebrow lbl" style="color:var(--steel)">If you defend Windows estates</span>
      <h3>The layer that blocks and the layer that sees are not the same layer</h3>
      <ul>
        <li><strong>A control running inside the same process as the code it inspects is tamperable by anyone who already has execution in that process.</strong> An excellent filter and a poor guarantee: it belongs inside a defense-in-depth stack, never alone at the top of it.</li>
        <li><strong>What an in-process payload can never reach is everything that lives elsewhere:</strong> script-block logging at the engine level, process-creation telemetry, outbound-connection records. Off-host or out-of-process by design, which is why they are worth investing in.</li>
        <li><strong>The script still has to call home.</strong> Beating a local check does nothing about the connection that opens to an external host. In a monitored environment that egress is the hardest part to hide.</li>
        <li><strong>Strange obfuscation is itself the signal.</strong> Heavily encoded script execution does not look like normal administration. A policy that treats it as an event worth reviewing catches what any single technical control would miss.</li>
      </ul>
    </div>
  </section>

  <section>
    <div class="card">
      <span class="eyebrow lbl">The moral &middot; one misread question</span>
      <p style="margin-top:12px">Obfuscation and encryption both fail against a modern endpoint for the same reason, and it has nothing to do with how well either is done. Each answers a question, "what shape is it," "what's inside it," that the layer actually in charge was never asking. The layer in charge asks what the code is about to do, and it is shown the real thing by design. Understand that and you understand both failures in one sentence, on whichever side of the fence you sit.</p>
    </div>
  </section>

  <p class="foot"><em>A concepts piece drawn from offensive-security preparation in an isolated lab on hardware I own. No code, commands, flags or implementation detail, and no method for defeating a security control: the aim is the reasoning, not a tool.</em></p>

</div>
