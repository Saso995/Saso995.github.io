---
title: "Three Layers That Don't Do What You Think"
date: 2026-11-10 23:05:00 +0100
categories: [Security, Concepts]
tags: [endpoint-security, powershell, evasion, obfuscation, blue-team, offensive-security]
description: "Why obfuscation and encryption keep failing against a modern endpoint, and what that failure is actually telling you."
---

<iframe id="tl-frame"
        src="{{ '/assets/html/three-layers.html' | relative_url }}"
        style="width:100%;border:0;display:block"
        loading="lazy"
        title="Three Layers That Don't Do What You Think"></iframe>

<script>
(function () {
  var f = document.getElementById('tl-frame');
  function fit() {
    try {
      var d = f.contentWindow.document;
      f.style.height = (d.documentElement.scrollHeight || d.body.scrollHeight) + 'px';
    } catch (e) { f.style.height = '2400px'; }
  }
  f.addEventListener('load', fit);
  window.addEventListener('resize', function () { clearTimeout(window.__tl); window.__tl = setTimeout(fit, 150); });
  // re-fit a couple of times after load, in case webfonts shift the layout
  setTimeout(fit, 400); setTimeout(fit, 1200);
})();
</script>
