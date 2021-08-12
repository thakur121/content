---
title: 'XRSystem: isSessionSupported()'
slug: Web/API/XRSystem/isSessionSupported
tags:
- API
- AR
- Augmented Reality
- Experimental
- Method
- Reference
- VR
- Virtual Reality
- WebXR
- WebXR Device API
- XR
- isSessionSupported
browser-compat: api.XRSystem.isSessionSupported
---
<div>{{APIRef("WebXR Device API")}}</div>

<p>The {{domxref("XRSystem")}} method
    <code><strong>isSessionSupported()</strong></code> returns a promise which resolves to
    <code>true</code> if the specified WebXR session mode is supported by the user's WebXR
    device. Otherwise, the promise resolves with <code>false</code>.</p>

<p>If no devices are available or the browser doesn't have permission
  to use the XR device, the promise is rejected with an appropriate
  {{domxref("DOMException")}}.</p>

<h2 id="Syntax">Syntax</h2>

<pre class="brush: js">xr.isSessionSupported(mode)</pre>

<h3 id="Parameters">Parameters</h3>

<dl>
  <dt><code>mode</code></dt>
  <dd>A {{jsxref("String")}} specifying the WebXR session mode for which support is to
    be checked. Possible modes to check for:
    <ul>
      <li><code>immersive-ar</code> {{experimental_inline}}</li>
      <li><code>immersive-vr</code></li>
      <li><code>inline</code></li>
     </ul>
  </dd>
</dl>

<h3 id="Return_value">Return value</h3>

<p>A {{jsxref("Promise")}} that resolves to <code>true</code> if the specified session
  mode is supported; otherwise the promise resolves to <code>false</code>.</p>

<h3 id="Exceptions">Exceptions</h3>

<p>Rather than throwing true exceptions, <code>isSessionSupported()</code> rejects the
  returned promise, passing to the rejection handler a {{domxref("DOMException")}} whose
  <code>name</code> is one of the following strings.</p>

<dl>
  <dt><code>SecurityError</code></dt>
  <dd>The document's origin does not have permission to use the
    <code>xr-spatial-tracking</code> <a href="/en-US/docs/Web/HTTP/Feature_Policy">feature
      policy</a>.</dd>
</dl>

<h2 id="Examples">Examples</h2>

<p>In this example, we see <code>isSessionSupported()</code> used to detect whether or not
  the device supports VR mode by checking to see if an <code>immersive-vr</code> session
  is supported. If it is, we set up a button to read "Enter XR", to call a method
  <code>onButtonClicked()</code>, and enable the button.</p>

<p>If no session is already underway, we request the VR session and, if successful, set up
  the session in a method called <code>onSessionStarted()</code>, not shown. If a session
  <em>is</em> already underway when the button is clicked, we call the
  <code>xrSession</code> object's {{domxref("XRSession.end", "end()")}} method to shut
  down the WebXR session.</p>

<pre class="brush: js">if (navigator.xr) {
  navigator.xr.isSessionSupported('immersive-vr')
  .then((isSupported) =&gt; {
    if (isSupported) {
      userButton.addEventListener('click', onButtonClicked);
      userButton.textContent = 'Enter XR';
      userButton.disabled = false;
    }
  });
}

function onButtonClicked() {
  if (!xrSession) {
    navigator.xr.requestSession('immersive-vr')
    .then((session) =&gt; {
      xrSession = session;
      // onSessionStarted() not shown for reasons of brevity and clarity.
      onSessionStarted(xrSession);
    });
  } else {
    // Button is a toggle button.
    xrSession.end();
  }
}</pre>

<h2 id="Specifications">Specifications</h2>

{{Specifications}}

<h2 id="Browser_compatibility">Browser compatibility</h2>

<div>{{Compat}}</div>