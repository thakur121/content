---
title: XRWebGLLayer.antialias
slug: Web/API/XRWebGLLayer/antialias
tags:
- API
- AR
- Drawing
- Graphics
- Quality
- Reality
- Reference
- VR
- Virtual
- WebXR
- WebXR API
- WebXR Device API
- XR
- XRWebGLLayer
- anti-aliasing
- antialias
- appearance
- augmented
- rendering
browser-compat: api.XRWebGLLayer.antialias
---
<p>{{APIRef("WebXR Device API")}}</p>

<p>The read-only {{domxref("XRWebGLLayer")}} property
    <code><strong>antialias</strong></code> is a Boolean value which is <code>true</code>
    if the rendering layer's frame buffer supports antialiasing. Otherwise, this
  property's value is <code>false</code>. The specific antialiasing technique used is left
  to the {{Glossary("user agent", "user agent's")}} discretion and cannot be specified by
  the web site or web app.</p>

<h2 id="Syntax">Syntax</h2>

<pre
  class="brush: js">let <em>antialiasingSupported</em> = <em>xrWebGLLayer</em>.antialias;</pre>

<h3 id="Value">Value</h3>

<p>A Boolean value which is <code>true</code> if the WebGL rendering layer's frame buffer
  is configured to support antialiasing. Otherwise, this property is <code>false</code>.
</p>

<p>When the <a
    href="/en-US/docs/Web/API/WebXR_Device_API/Fundamentals#The_WebXR_compositor">WebXR
    compositor</a> is enabled, this value corresponds to the value of the
  <code>antialias</code> property on the object returned by the WebGL context's
  {{domxref("WebGLRenderingContext.getContextAttributes", "getContentAttributes()")}}
  method.</p>

<h2 id="Usage_notes">Usage notes</h2>

<p>Since this is a read-only property, you can set the antialiasing mode only when
  initially creating the <code>XRWebGLLayer</code>, by specifying the <code>antialias</code>
  property in the {{domxref("XRWebGLLayer.XRWebGLLayer", "XRWebGLLayer()")}}
  constructor's <code>layerInit</code> configuration object.</p>

<h2 id="Examples">Examples</h2>

<p>This snippet checks the value of <code>antialias</code> to see if it should perform
  additional work to attempt to compensate for the lack of antialiasing on the WebGL
  layer.</p>

<pre class="brush: js">let glLayer = xrSession.renderState.baseLayer;
gl.bindFrameBuffer(gl.FRAMEBUFFER, glLayer.framebuffer);

/* .. */

if (!glLayer.antialias) {
  /* compensate for lack of antialiasing */
}
</pre>

<h2 id="Specifications">Specifications</h2>

{{Specifications}}

<h2 id="Browser_compatibility">Browser compatibility</h2>

<p>{{Compat}}</p>

<h2 id="See_also">See also</h2>

<ul>
  <li><a href="/en-US/docs/Web/API/WebXR_Device_API">WebXR Device API</a></li>
  <li>{{domxref("WebGLLayerInit")}}</li>
</ul>