---
title: "Three Layers That Don't Do What You Think"
date: 2026-11-10 23:05:00 +0100
categories: [Security, Concepts]
tags: [endpoint-security, powershell, evasion, obfuscation, blue-team, offensive-security]
description: "Why obfuscation and encryption keep failing against a modern endpoint, and what that failure is actually telling you."
---

<iframe id="tl-frame"
        src="{{ '/assets/html/three-layers.html' | relative_url }}"
        scrolling="no"
        style="width:100%;height:600px;border:0;display:block;overflow:hidden"
        title="Three Layers That Don't Do What You Think"></iframe>

<script>
(function () {
  var f = document.getElementById('tl-frame');

  // Preferred path: the embedded page reports its own height via postMessage.
  window.addEventListener('message', function (e) {
    if (e && e.data && e.data.__tl && typeof e.data.height === 'number') {
      f.style.height = e.data.height + 'px';
    }
  });

  // Fallback for same-origin hosting, in case the message never arrives.
  function fit() {
    try {
      var d = f.contentWindow.document;
      var h = Math.max(d.documentElement.scrollHeight, d.body.scrollHeight);
      if (h > 0) f.style.height = h + 'px';
    } catch (e) { /* cross-origin or not ready: leave the postMessage path to it */ }
  }
  f.addEventListener('load', fit);
  setTimeout(fit, 600);
  setTimeout(fit, 1500);
})();
</script>
