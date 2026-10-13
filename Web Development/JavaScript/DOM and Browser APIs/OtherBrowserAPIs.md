# Other Browser APIs — Comprehensive Programming Cheat Sheet

---

## Topic Overview

### Definitions

**Core Definition:** Browser APIs (also called Web APIs) are interfaces built into the browser environment that extend the capabilities of JavaScript beyond the core language, enabling developers to interact with graphics, multimedia, hardware, storage, sensors, and other web platform features.

**Technical Definition:** Web APIs are specified primarily by the W3C (World Wide Web Consortium) and the WHATWG (Web Hypertext Application Technology Working Group), with some specifications managed by the Khronos Group (WebGL), the W3C WebAssembly Community Group (WebAssembly), and the IETF (WebRTC). They are exposed as global objects, interfaces, and methods available in the browser's JavaScript environment, but they are not part of the ECMAScript language specification itself. Each API defines IDL (Interface Definition Language) interfaces that map to JavaScript objects, often with event-driven and Promise-based patterns.

**Beginner-Friendly Explanation:** JavaScript by itself can only do math, manipulate data, and control program flow. Browser APIs are the extra "superpowers" the browser gives you: drawing pictures on a canvas, playing audio, getting the user's location, reading files, using the webcam, sending notifications, and much more. Each API is like a toolbox for a specific job.

### Key Characteristics

- **Not part of ECMAScript:** Browser APIs are provided by the host environment (the browser), not the language itself.
- **Event-driven and Promise-based:** Most APIs use events, callbacks, or Promises for asynchronous operations.
- **Permission-gated:** Many APIs (Geolocation, Notifications, Clipboard, Bluetooth, USB, Serial, MIDI) require explicit user permission.
- **Secure-context-restricted:** Most powerful APIs require HTTPS (`window.isSecureContext === true`).
- **Feature-detectable:** APIs should be detected before use (`if ('geolocation' in navigator)`).
- **Vendor-prefixed historically:** Older APIs used prefixes like `webkit`, `moz`, `ms`; modern APIs are standardised.
- **Cross-origin restricted:** Many APIs are subject to CORS and same-origin policies.

### Prerequisites

- Solid understanding of JavaScript fundamentals (functions, objects, events, Promises, async/await).
- Familiarity with the DOM (Document Object Model) and HTML.
- Basic knowledge of CSS for styling visual APIs.
- Understanding of HTTP, URLs, and the same-origin policy.
- Awareness of HTTPS and secure contexts for permission-gated APIs.

### Related Programming Areas

- **Front-end frameworks:** React, Vue, Angular wrap many browser APIs.
- **Game development:** Canvas, WebGL, WebAudio, Gamepad API.
- **Real-time communication:** WebRTC, WebSocket, MediaStream.
- **Progressive Web Apps (PWAs):** Service Workers, Notifications, Payment Request, Web Share.
- **Accessibility:** ARIA, Intersection Observer for lazy loading.
- **Performance engineering:** Performance API, Observers.

---

## Canvas API

### Definitions

**Core Definition:** The Canvas API provides a means for drawing 2D graphics dynamically via JavaScript using the `<canvas>` HTML element and its rendering context.

**Technical Definition:** Defined by the WHATWG HTML Living Standard, the Canvas API exposes the `HTMLCanvasElement` interface with a `getContext("2d")` method returning a `CanvasRenderingContext2D` object. This context offers raster drawing operations (paths, shapes, text, images, transformations, compositing) against a bitmap canvas. A separate `getContext("webgl")` returns a WebGL context.

**Beginner-Friendly Explanation:** Canvas is like a blank digital sheet of paper. You use JavaScript as your paintbrush to draw lines, shapes, text, and images onto it. It's great for games, charts, image editors, and animations.

### Purposes

- To draw 2D graphics programmatically without plugins.
- To create dynamic charts, games, and image manipulation tools.
- To render offscreen graphics for processing or compositing.
- To build custom UI elements that HTML/CSS cannot easily produce.

### Syntax Rules and Structure

**General Syntax:**
```js
const canvas = document.getElementById("myCanvas");
const ctx = canvas.getContext("2d");
```

**Component Breakdown:**
- `canvas`: An `HTMLCanvasElement` in the DOM.
- `getContext("2d")`: Returns a `CanvasRenderingContext2D` or `null` if unsupported.
- `ctx`: The drawing context with methods like `fillRect`, `beginPath`, `stroke`, `fill`, `drawImage`, `fillText`.

**Common Drawing Methods:**
- `ctx.fillRect(x, y, width, height)` — Filled rectangle.
- `ctx.strokeRect(x, y, w, h)` — Outlined rectangle.
- `ctx.beginPath()`, `ctx.moveTo(x, y)`, `ctx.lineTo(x, y)`, `ctx.stroke()` — Path drawing.
- `ctx.arc(x, y, radius, startAngle, endAngle)` — Circles and arcs.
- `ctx.fillText(text, x, y)` — Text.
- `ctx.drawImage(img, x, y)` — Images.

**Syntax Rules:**
- Coordinates are in CSS pixels, with origin at top-left (0,0).
- The canvas has an intrinsic size (`width`/`height` attributes) separate from its CSS size.
- Drawing is immediate-mode: once drawn, shapes are pixels, not objects.

**Constraints and Limitations:**
- Raster-based: scaling loses quality.
- Not accessible by default (text drawn is not in the DOM).
- Cross-origin images taint the canvas, blocking `getImageData()`.
- `getContext("2d")` returns the same context object on repeated calls.

### Annotated Code Examples

#### Example 1: Drawing a Filled Rectangle and Circle

```html
<!DOCTYPE html>
<html>
<body>
  <canvas id="myCanvas" width="300" height="200"></canvas>
  <script>
    // Step 1: Get the canvas element and its 2D context.
    const canvas = document.getElementById("myCanvas");
    const ctx = canvas.getContext("2d");

    // Step 2: Draw a filled blue rectangle.
    ctx.fillStyle = "blue";
    ctx.fillRect(20, 20, 100, 60);

    // Step 3: Draw a red circle.
    ctx.beginPath();
    ctx.arc(200, 100, 40, 0, Math.PI * 2);
    ctx.fillStyle = "red";
    ctx.fill();
  </script>
</body>
</html>
```

**Expected Output:** A 300×200 canvas showing a blue rectangle in the top-left and a red filled circle in the middle-right.

**Why This Output:** `fillRect` draws a solid rectangle using the current `fillStyle`. The `arc` method defines a circular path, and `fill()` paints it red.

#### Example 2: Animating a Moving Square

```html
<canvas id="anim" width="400" height="100"></canvas>
<script>
  const canvas = document.getElementById("anim");
  const ctx = canvas.getContext("2d");
  let x = 0;

  // Step 1: Animation loop using requestAnimationFrame.
  function animate() {
    // Step 2: Clear the previous frame.
    ctx.clearRect(0, 0, canvas.width, canvas.height);
    // Step 3: Draw the square at its new position.
    ctx.fillStyle = "green";
    ctx.fillRect(x, 40, 20, 20);
    // Step 4: Advance x and wrap around.
    x = (x + 2) % canvas.width;
    // Step 5: Request the next frame.
    requestAnimationFrame(animate);
  }
  animate();
</script>
```

**Expected Output:** A small green square sliding horizontally across the canvas, wrapping around when it reaches the right edge.

**Why This Output:** `requestAnimationFrame` schedules the next repaint, `clearRect` erases the previous position, and the modulo operator wraps the coordinate.

### Real-World Cases

- **Charting libraries:** Chart.js, D3 (canvas mode), Chartist.
- **Image editors:** Cropping, filters, annotation tools (e.g., Fabric.js).
- **Games:** 2D games and sprite-based engines (Phaser, PixiJS).
- **Signature pads:** Capturing handwritten signatures with `toDataURL()`.
- **Video thumbnails:** Drawing video frames to canvas for previews.

---

## WebGL

### Definitions

**Core Definition:** WebGL (Web Graphics Library) is a JavaScript API for rendering high-performance interactive 2D and 3D graphics within any compatible browser without plugins, based on OpenGL ES.

**Technical Definition:** WebGL 1.0 is based on OpenGL ES 2.0; WebGL 2.0 is based on OpenGL ES 3.0. The API exposes the `WebGLRenderingContext` (and `WebGL2RenderingContext`) obtained via `canvas.getContext("webgl")` or `"webgl2"`. It operates via shaders (GLSL) and buffer objects, requiring vertex and fragment shader programs.

**Beginner-Friendly Explanation:** WebGL lets you create 3D graphics in the browser — games, data visualisations, VR experiences — using the same technology as professional 3D software, but powered by JavaScript.

### Purposes

- To render 3D graphics in real time within the browser.
- To build high-performance games and visualisations.
- To power VR/AR experiences via WebXR.
- To accelerate 2D rendering beyond the Canvas 2D API.

### Syntax Rules and Structure

**General Syntax:**
```js
const gl = canvas.getContext("webgl2") || canvas.getContext("webgl");
```

**Key Objects:**
- Shaders: Vertex and fragment shaders written in GLSL.
- Programs: Compiled shader pairs.
- Buffers: Vertex data stored in GPU memory.
- Textures: Image data for sampling in shaders.

**Syntax Rules:**
- Requires shader compilation and linking.
- Uses column-major matrices (gl-matrix library is common).
- Coordinate system is clip space (-1 to 1 on each axis).
- `gl.drawArrays()` or `gl.drawElements()` issues draw calls.

**Constraints and Limitations:**
- Requires WebGL support (virtually universal now, but disabled in some contexts).
- Security restrictions: cross-origin textures require CORS.
- Debugging is notoriously difficult (use Spector.js, WebGL Inspector).
- Context loss can occur (e.g., GPU reset); handle `webglcontextlost`.

### Annotated Code Examples

#### Example 1: Drawing a Triangle with WebGL 1

```html
<canvas id="gl" width="300" height="300"></canvas>
<script>
  const gl = document.getElementById("gl").getContext("webgl");

  // Step 1: Define vertex shader (positions).
  const vsSource = `
    attribute vec2 aPosition;
    void main() {
      gl_Position = vec4(aPosition, 0.0, 1.0);
    }
  `;

  // Step 2: Define fragment shader (colour).
  const fsSource = `
    void main() {
      gl_FragColor = vec4(1.0, 0.5, 0.0, 1.0);
    }
  `;

  // Step 3: Compile a shader.
  function compile(type, source) {
    const shader = gl.createShader(type);
    gl.shaderSource(shader, source);
    gl.compileShader(shader);
    return shader;
  }

  // Step 4: Create and link a program.
  const program = gl.createProgram();
  gl.attachShader(program, compile(gl.VERTEX_SHADER, vsSource));
  gl.attachShader(program, compile(gl.FRAGMENT_SHADER, fsSource));
  gl.linkProgram(program);
  gl.useProgram(program);

  // Step 5: Provide vertex data.
  const buffer = gl.createBuffer();
  gl.bindBuffer(gl.ARRAY_BUFFER, buffer);
  gl.bufferData(gl.ARRAY_BUFFER, new Float32Array([-0.5, -0.5, 0.5, -0.5, 0.0, 0.5]), gl.STATIC_DRAW);

  // Step 6: Link the attribute to the buffer.
  const loc = gl.getAttribLocation(program, "aPosition");
  gl.enableVertexAttribArray(loc);
  gl.vertexAttribPointer(loc, 2, gl.FLOAT, false, 0, 0);

  // Step 7: Draw.
  gl.clearColor(0, 0, 0, 1);
  gl.clear(gl.COLOR_BUFFER_BIT);
  gl.drawArrays(gl.TRIANGLES, 0, 3);
</script>
```

**Expected Output:** A black canvas with an orange triangle occupying the centre.

**Why This Output:** The vertex shader places three vertices in clip space; the fragment shader colours every covered pixel orange (RGB 1, 0.5, 0). `drawArrays` rasterises the triangle.

### Real-World Cases

- **Three.js, Babylon.js, PlayCanvas:** Popular 3D engines built on WebGL.
- **Google Maps:** Uses WebGL for smooth 3D map rendering.
- **Figma/Excalidraw:** Use WebGL for performance in complex vector rendering.
- **Data visualisation:** Deck.gl, regl for large datasets.

---

## SVG

### Definitions

**Core Definition:** SVG (Scalable Vector Graphics) is an XML-based markup language for describing two-dimensional vector graphics, which can be embedded in HTML and manipulated via the DOM.

**Technical Definition:** SVG is a W3C recommendation. In HTML5, `<svg>` elements are parsed as part of the DOM, exposing interfaces like `SVGSVGElement`, `SVGPathElement`, etc. SVG shapes are DOM nodes that can be styled with CSS, animated with SMIL or CSS animations, and scripted with JavaScript.

**Beginner-Friendly Explanation:** SVG is a way to draw shapes with HTML-like tags. Unlike canvas (which is pixel-based), SVG shapes are objects you can select, style, and animate with CSS.

### Purposes

- To render resolution-independent vector graphics.
- To create interactive, styleable diagrams and icons.
- To animate shapes declaratively with CSS or SMIL.
- To produce accessible graphics (SVG elements appear in the DOM).

### Syntax Rules and Structure

```html
<svg width="200" height="200" viewBox="0 0 200 200">
  <circle cx="100" cy="100" r="50" fill="blue" />
  <rect x="20" y="20" width="40" height="40" fill="red" />
  <path d="M10 10 L90 90" stroke="black" />
</svg>
```

**Component Breakdown:**
- `<svg>`: Root container; `viewBox` defines internal coordinate system.
- `<circle>`, `<rect>`, `<line>`, `<path>`, `<text>`: Shape elements.
- `fill`, `stroke`, `stroke-width`: Presentation attributes (CSS-compatible).

**Syntax Rules:**
- SVG uses a namespace (`xmlns="http://www.w3.org/2000/svg"` in standalone files).
- Inline SVG in HTML does not require the namespace.
- CSS properties like `fill` and `stroke` can be set via CSS.

**Constraints and Limitations:**
- Complex SVGs can be slow to render (many DOM nodes).
- Not suitable for pixel-based operations (use Canvas instead).
- Cross-browser differences exist in SMIL animation support.

### Annotated Code Examples

#### Example 1: Interactive SVG Circle

```html
<svg id="s" width="200" height="200">
  <circle id="dot" cx="100" cy="100" r="30" fill="teal" />
</svg>
<script>
  // Step 1: Get the circle element.
  const dot = document.getElementById("dot");
  // Step 2: Change its colour on click.
  dot.addEventListener("click", () => {
    dot.setAttribute("fill", "orange");
  });
</script>
```

**Expected Output:** A teal circle that turns orange when clicked.

**Why This Output:** SVG elements are DOM nodes with attributes; `setAttribute` updates the `fill` attribute and the browser repaints.

### Real-World Cases

- **Icons:** Feather Icons, Heroicons, Font Awesome.
- **Charts:** D3.js (SVG mode), ApexCharts.
- **Maps:** Leaflet uses SVG for vector overlays.
- **Logos:** Most modern logos are SVG for scalability.

---

## Audio API

### Definitions

**Core Definition:** The Web Audio API provides a high-level JavaScript interface for processing and synthesising audio in the browser.

**Technical Definition:** The Web Audio API models audio as a graph of `AudioNode` objects connected via `AudioContext`. Sources (oscillators, buffers, media streams) connect to effects (gain, filters, convolvers) and finally to a destination (speakers). It supports sample-accurate scheduling, spatial audio, and real-time analysis.

**Beginner-Friendly Explanation:** Web Audio lets you play sounds, apply effects, generate tones, and analyse audio — all in JavaScript. It's like a modular synthesiser in your browser.

### Purposes

- To play and mix audio files.
- To synthesise sounds (oscillators, noise).
- To apply real-time audio effects (reverb, filters).
- To analyse audio (spectrum, waveform) for visualisations.
- To build games, music apps, and voice communication tools.

### Syntax Rules and Structure

```js
const ctx = new AudioContext();
const osc = ctx.createOscillator();
const gain = ctx.createGain();
osc.connect(gain).connect(ctx.destination);
osc.start();
```

**Component Breakdown:**
- `AudioContext`: The audio processing environment.
- `AudioNode`: Generic node; sources, effects, destinations are all nodes.
- `connect()`: Links node outputs to node inputs.

**Common Nodes:**
- `OscillatorNode`: Generates periodic waveforms.
- `GainNode`: Controls volume.
- `BiquadFilterNode`: Filters frequencies.
- `AnalyserNode`: Provides real-time frequency/time data.

**Syntax Rules:**
- `AudioContext` must be created or resumed after a user gesture due to autoplay policies.
- Nodes are garbage-collected once disconnected and no longer referenced.

**Constraints and Limitations:**
- Autoplay policies block audio until user interaction.
- Performance can suffer with many nodes.
- Safari has historically had partial implementations (improving).

### Annotated Code Examples

#### Example 1: Playing a Tone

```js
// Step 1: Create an AudioContext (must follow a user gesture).
const ctx = new AudioContext();

// Step 2: Create an oscillator and a gain node.
const osc = ctx.createOscillator();
const gain = ctx.createGain();

// Step 3: Set the frequency and volume.
osc.frequency.value = 440; // A4 note
gain.gain.value = 0.2;

// Step 4: Connect the graph: osc -> gain -> speakers.
osc.connect(gain).connect(ctx.destination);

// Step 5: Start and stop the sound.
osc.start();
setTimeout(() => osc.stop(), 1000);
```

**Expected Output:** A 440 Hz tone plays for one second.

**Why This Output:** The oscillator generates a sine wave at 440 Hz; the gain node reduces the amplitude; the destination routes to speakers.

### Real-World Cases

- **Music players:** SoundCloud, Spotify web player (partially).
- **Games:** Real-time sound effects and music.
- **Voice chat:** Combined with MediaStream for processing.
- **Audio visualisers:** Frequency bars and waveforms (via AnalyserNode).

---

## Video API

### Definitions

**Core Definition:** The HTML5 Video API allows embedding and controlling video content via the `<video>` element and its associated JavaScript interface (`HTMLVideoElement`).

**Technical Definition:** `HTMLVideoElement` inherits from `HTMLMediaElement`, providing `play()`, `pause()`, `currentTime`, `duration`, `volume`, `playbackRate`, and events (`play`, `pause`, `timeupdate`, `ended`, `loadedmetadata`). It supports multiple sources via `<source>` and adaptive streaming via MSE (Media Source Extensions).

**Beginner-Friendly Explanation:** The `<video>` element plays videos on a web page, and JavaScript can control playback, volume, seeking, and respond to events.

### Purposes

- To embed and control video playback.
- To build custom video players.
- To react to playback state changes.
- To feed video into other APIs (Canvas, WebRTC, MediaStream).

### Syntax Rules and Structure

```html
<video id="v" src="movie.mp4" controls></video>
<script>
  const v = document.getElementById("v");
  v.play();
</script>
```

**Syntax Rules:**
- `play()` returns a Promise in modern browsers; catch rejections from autoplay blocks.
- Cross-origin video requires CORS for canvas access.
- Format support varies by browser (use `<source>` for fallbacks).

**Constraints and Limitations:**
- Autoplay is restricted without muted or user gesture.
- DRM requires Encrypted Media Extensions (EME).

### Annotated Code Examples

#### Example 1: Custom Play/Pause Button

```html
<video id="v" src="clip.mp4" width="320"></video>
<button id="toggle">Play</button>
<script>
  const v = document.getElementById("v");
  const btn = document.getElementById("toggle");
  btn.addEventListener("click", async () => {
    if (v.paused) {
      await v.play();
      btn.textContent = "Pause";
    } else {
      v.pause();
      btn.textContent = "Play";
    }
  });
</script>
```

**Expected Output:** A video with a button that toggles playback and updates its label.

**Why This Output:** `v.paused` reflects the current state; `play()` and `pause()` change it; the button label updates accordingly.

### Real-World Cases

- **Video streaming sites:** YouTube, Vimeo (custom players).
- **E-learning platforms:** Course video players with progress tracking.
- **Video editors:** Frame-by-frame analysis via `currentTime`.

---

## MediaStream API

### Definitions

**Core Definition:** The MediaStream API represents streams of media content (audio and/or video) that can be captured from local devices or obtained from remote peers.

**Technical Definition:** A `MediaStream` is a container of `MediaStreamTrack` objects (audio or video). It is obtained from `navigator.mediaDevices.getUserMedia()` (local capture), `getDisplayMedia()` (screen capture), or remotely via WebRTC. Tracks can be added, removed, and cloned.

**Beginner-Friendly Explanation:** MediaStream is how you get access to the user's camera, microphone, or screen — and how you send that media to a video element, canvas, or another peer.

### Purposes

- To capture audio/video from local devices.
- To stream media to remote peers via WebRTC.
- To record media with MediaRecorder.
- To process media in real time (Canvas, Web Audio).

### Syntax Rules and Structure

```js
const stream = await navigator.mediaDevices.getUserMedia({ video: true, audio: true });
videoElement.srcObject = stream;
```

**Component Breakdown:**
- `getUserMedia(constraints)`: Requests access; returns a Promise resolving to a MediaStream.
- `constraints`: Specifies required media types and settings.
- `srcObject`: Assigns a MediaStream to a media element (not `src`).

**Syntax Rules:**
- Must be called from a secure context (HTTPS or localhost).
- Requires user permission.
- Tracks should be stopped (`track.stop()`) when no longer needed to release the camera/mic.

**Constraints and Limitations:**
- Permissions can be denied.
- Device availability varies.
- Some browsers restrict concurrent camera access.

### Annotated Code Examples

#### Example 1: Webcam Preview

```html
<video id="cam" autoplay playsinline width="320"></video>
<script>
  async function startCamera() {
    try {
      // Step 1: Request camera access.
      const stream = await navigator.mediaDevices.getUserMedia({ video: true });
      // Step 2: Attach the stream to the video element.
      document.getElementById("cam").srcObject = stream;
    } catch (err) {
      // Step 3: Handle permission denial or missing devices.
      console.error("Camera access denied:", err);
    }
  }
  startCamera();
</script>
```

**Expected Output:** The user's webcam feed displayed in the video element.

**Why This Output:** `getUserMedia` returns a MediaStream; assigning it to `srcObject` connects the stream to the element.

### Real-World Cases

- **Video conferencing:** Zoom, Google Meet (via WebRTC).
- **Photo booths:** Capture frames to canvas.
- **Screen sharing:** `getDisplayMedia()` for presentations.
- **Audio recorders:** MediaRecorder for voice memos.

---

## WebRTC

### Definitions

**Core Definition:** WebRTC (Web Real-Time Communication) is a set of APIs enabling peer-to-peer audio, video, and data communication directly between browsers without a server intermediary (after initial signalling).

**Technical Definition:** WebRTC comprises `RTCPeerConnection` (media transport), `RTCDataChannel` (arbitrary data), and the signalling process (SDP offer/answer, ICE candidates). It uses SRTP for media encryption and DTLS for data channels. Signalling is not specified by WebRTC and must be implemented by the developer (typically via WebSocket).

**Beginner-Friendly Explanation:** WebRTC lets two browsers talk directly to each other for video calls, voice chat, or file transfer — without routing data through a server.

### Purposes

- To establish peer-to-peer audio/video calls.
- To transfer files or data directly between peers.
- To build low-latency real-time applications.
- To enable decentralised communication.

### Syntax Rules and Structure

```js
const pc = new RTCPeerConnection(configuration);
pc.addTrack(track, stream);
pc.onicecandidate = (e) => { /* send e.candidate to peer */ };
pc.ontrack = (e) => { /* receive remote track */ };
const offer = await pc.createOffer();
await pc.setLocalDescription(offer);
// Send offer to peer, receive answer, setRemoteDescription...
```

**Component Breakdown:**
- `RTCPeerConnection`: Manages the connection.
- `configuration`: Includes ICE servers (STUN/TURN).
- `SDP`: Session Description Protocol, describing media capabilities.
- `ICE`: Interactive Connectivity Establishment, for NAT traversal.

**Syntax Rules:**
- Signalling channel is required (WebSocket, HTTP, etc.).
- STUN/TURN servers are needed for NAT traversal.
- Secure context required.

**Constraints and Limitations:**
- Complex signalling setup.
- Firewalls and symmetric NATs require TURN relays.
- Codec support varies.

### Annotated Code Examples

#### Example 1: Minimal Data Channel Setup

```js
// Step 1: Create a peer connection with a public STUN server.
const pc = new RTCPeerConnection({ iceServers: [{ urls: "stun:stun.l.google.com:19302" }] });

// Step 2: Create a data channel.
const dc = pc.createDataChannel("chat");

// Step 3: Handle incoming messages.
dc.onmessage = (e) => console.log("Received:", e.data);

// Step 4: Open the channel and send a message.
dc.onopen = () => dc.send("Hello, peer!");

// Step 5: Create an offer and set it as local description.
pc.createOffer().then((offer) => pc.setLocalDescription(offer));
// In a real app: send the offer to the peer via a signalling server.
```

**Expected Output:** When connected to a peer, the console logs "Received: Hello, peer!" on the remote side.

**Why This Output:** The data channel provides a direct channel for arbitrary data once the peer connection is established.

### Real-World Cases

- **Video conferencing:** Google Meet, Jitsi, Discord.
- **File transfer:** ShareDrop, Snapdrop.
- **Gaming:** Low-latency multiplayer.
- **IoT:** Device-to-device communication.

---

## Geolocation API

### Definitions

**Core Definition:** The Geolocation API allows web applications to obtain the user's geographic position, subject to user consent.

**Technical Definition:** Exposed via `navigator.geolocation`, providing `getCurrentPosition()`, `watchPosition()`, and `clearWatch()`. Positions include latitude, longitude, accuracy, altitude, heading, and speed. Backed by GPS, Wi-Fi, cellular, and IP-based location sources.

**Beginner-Friendly Explanation:** Geolocation asks the user "Can I know where you are?" and if they agree, gives your app their latitude and longitude.

### Purposes

- To provide location-aware services (maps, weather, nearby stores).
- To track movement over time.
- To tailor content by region.

### Syntax Rules and Structure

```js
navigator.geolocation.getCurrentPosition(success, error, options);
```

**Component Breakdown:**
- `success`: Callback receiving a `GeolocationPosition`.
- `error`: Callback receiving a `GeolocationPositionError`.
- `options`: `{ enableHighAccuracy, timeout, maximumAge }`.

**Constraints and Limitations:**
- Requires HTTPS and user permission.
- Accuracy varies.
- Not available in all contexts (e.g., some WebViews).

### Annotated Code Examples

#### Example 1: Getting Current Position

```js
navigator.geolocation.getCurrentPosition(
  (pos) => {
    const { latitude, longitude } = pos.coords;
    console.log(`Lat: ${latitude}, Lon: ${longitude}`);
  },
  (err) => console.error("Geolocation error:", err.message),
  { enableHighAccuracy: true, timeout: 10000 }
);
```

**Expected Output:** Logs the user's latitude and longitude (after permission).

**Why This Output:** `getCurrentPosition` invokes the success callback with a position object once the device determines its location.

### Real-World Cases

- **Maps:** Google Maps, OpenStreetMap.
- **Ride-sharing:** Uber, Lyft.
- **Weather apps:** Local forecasts.
- **Fitness trackers:** Route logging.

---

## Notification API

### Definitions

**Core Definition:** The Notification API allows web pages to display system-level notifications to the user outside the browser window.

**Technical Definition:** Exposed via the `Notification` constructor and `Notification.requestPermission()`. Notifications can be simple text or include icons, actions, and data. The Service Worker `showNotification()` method enables notifications even when the page is closed.

**Beginner-Friendly Explanation:** Notifications are the little pop-ups that appear on your desktop or phone from websites, like "You have a new message."

### Purposes

- To alert users to events even when the tab is not focused.
- To re-engage users in PWAs.
- To provide timely information (messages, reminders).

### Syntax Rules and Structure

```js
await Notification.requestPermission();
new Notification("Title", { body: "Message", icon: "icon.png" });
```

**Constraints and Limitations:**
- Requires user permission.
- Typically requires HTTPS.
- Mobile support is limited (use Service Worker notifications).

### Annotated Code Examples

#### Example 1: Simple Notification

```js
// Step 1: Request permission.
Notification.requestPermission().then((perm) => {
  if (perm === "granted") {
    // Step 2: Show a notification.
    new Notification("Hello!", { body: "This is a test notification." });
  }
});
```

**Expected Output:** A system notification appears with the title "Hello!" and body text.

### Real-World Cases

- **Chat apps:** New message alerts.
- **PWAs:** Reminders, updates.
- **Email clients:** New mail notifications.

---

## Clipboard API

### Definitions

**Core Definition:** The Clipboard API provides asynchronous read and write access to the system clipboard.

**Technical Definition:** Exposed via `navigator.clipboard`, offering `readText()`, `writeText()`, `read()`, and `write()` (the latter two for arbitrary MIME types). Requires secure context and, for reads, user permission.

**Beginner-Friendly Explanation:** Clipboard API lets your web app copy text to the clipboard or paste from it, with the user's permission.

### Purposes

- To copy text or images to the clipboard.
- To paste content from the clipboard.
- To enable "Copy to clipboard" buttons.

### Syntax Rules and Structure

```js
await navigator.clipboard.writeText("Hello");
const text = await navigator.clipboard.readText();
```

**Constraints and Limitations:**
- Requires HTTPS.
- Reading requires permission or user gesture.
- Some browsers restrict `read()` to specific MIME types.

### Annotated Code Examples

#### Example 1: Copy Button

```html
<button id="copy">Copy</button>
<script>
  document.getElementById("copy").addEventListener("click", async () => {
    await navigator.clipboard.writeText("Copied text!");
    console.log("Copied.");
  });
</script>
```

**Expected Output:** Clicking the button copies "Copied text!" and logs "Copied."

### Real-World Cases

- **Code editors:** Copy code snippets.
- **Forms:** Paste from clipboard.
- **Sharing:** Copy links.

---

## File API

### Definitions

**Core Definition:** The File API allows web applications to read files selected by the user via `<input type="file">` or drag-and-drop, and to construct file-like objects.

**Technical Definition:** Includes `File`, `FileList`, `Blob`, `FileReader`, and `URL.createObjectURL()`. Files can be read as text, DataURL, ArrayBuffer, or binary string.

**Beginner-Friendly Explanation:** File API lets you read files the user picks — images, text, JSON — without uploading them to a server first.

### Purposes

- To read user-selected files locally.
- To preview images before upload.
- To parse JSON/CSV client-side.

### Syntax Rules and Structure

```js
const file = input.files[0];
const reader = new FileReader();
reader.onload = () => console.log(reader.result);
reader.readAsText(file);
```

### Annotated Code Examples

#### Example 1: Image Preview

```html
<input type="file" id="f" accept="image/*">
<img id="preview" width="200">
<script>
  document.getElementById("f").addEventListener("change", (e) => {
    const file = e.target.files[0];
    if (file) {
      const url = URL.createObjectURL(file);
      document.getElementById("preview").src = url;
    }
  });
</script>
```

**Expected Output:** Selecting an image displays it in the `<img>` element.

### Real-World Cases

- **Image uploaders:** Previews, cropping.
- **CSV importers:** Parsing client-side.
- **PDF viewers:** Rendering locally.

---

## Drag and Drop API

### Definitions

**Core Definition:** The Drag and Drop API enables elements to be dragged and dropped within a page or between pages/applications.

**Technical Definition:** Uses the `draggable` attribute and events (`dragstart`, `drag`, `dragenter`, `dragover`, `drop`, `dragend`) and the `DataTransfer` object to carry data.

**Beginner-Friendly Explanation:** Drag and drop lets users pick up elements with the mouse and drop them elsewhere — like rearranging a to-do list.

### Purposes

- To reorder lists.
- To move files between folders.
- To drop files from the OS onto the page.

### Syntax Rules and Structure

```html
<div draggable="true" id="item">Drag me</div>
<div id="dropzone">Drop here</div>
<script>
  item.addEventListener("dragstart", (e) => e.dataTransfer.setData("text", "item"));
  dropzone.addEventListener("dragover", (e) => e.preventDefault());
  dropzone.addEventListener("drop", (e) => {
    e.preventDefault();
    console.log(e.dataTransfer.getData("text"));
  });
</script>
```

### Annotated Code Examples

#### Example 1: Simple Drag and Drop

```html
<div id="drag" draggable="true" style="width:100px;height:50px;background:lightblue;">Drag</div>
<div id="drop" style="width:200px;height:100px;border:2px dashed #ccc;">Drop</div>
<script>
  const drag = document.getElementById("drag");
  const drop = document.getElementById("drop");
  drag.addEventListener("dragstart", (e) => e.dataTransfer.setData("text/plain", "dragged"));
  drop.addEventListener("dragover", (e) => e.preventDefault());
  drop.addEventListener("drop", (e) => {
    e.preventDefault();
    drop.textContent = "Dropped: " + e.dataTransfer.getData("text/plain");
  });
</script>
```

**Expected Output:** Dragging the blue box onto the dashed area changes the drop zone's text.

### Real-World Cases

- **Kanban boards:** Trello, Jira.
- **File uploaders:** Drop files from OS.
- **Sortable lists:** Reordering.

---

## History API

### Definitions

**Core Definition:** The History API allows manipulation of the browser session history and URL without reloading the page.

**Technical Definition:** Exposed via `window.history`, with methods `pushState()`, `replaceState()`, `back()`, `forward()`, `go()`, and the `popstate` event. Enables SPA routing.

**Beginner-Friendly Explanation:** History API lets you change the URL and add entries to the browser's back button history without reloading — the foundation of single-page app routing.

### Purposes

- To implement SPA routing.
- To update URLs without reloading.
- To handle back/forward navigation.

### Syntax Rules and Structure

```js
history.pushState({ page: 1 }, "Title", "/page1");
window.addEventListener("popstate", (e) => console.log(e.state));
```

### Annotated Code Examples

#### Example 1: SPA Navigation

```js
function navigate(path, data) {
  history.pushState(data, "", path);
  document.getElementById("content").textContent = `Page: ${path}`;
}
window.addEventListener("popstate", (e) => {
  document.getElementById("content").textContent = `Page: ${location.pathname}`;
});
navigate("/about", { page: "about" });
```

### Real-World Cases

- **SPAs:** React Router, Vue Router.
- **Filters:** Updating URL with query params.
- **Pagination:** Adding history entries.

---

## Performance API

### Definitions

**Core Definition:** The Performance API provides access to timing and performance-related information about the current page.

**Technical Definition:** Includes `performance.now()`, `performance.mark()`, `performance.measure()`, `PerformanceObserver`, and the Navigation Timing, Resource Timing, User Timing, and Paint Timing specifications.

**Beginner-Friendly Explanation:** Performance API lets you measure how long things take — page load, function execution, resource fetching — so you can optimise.

### Purposes

- To measure code execution time.
- To monitor page load performance.
- To collect Core Web Vitals.

### Syntax Rules and Structure

```js
const t0 = performance.now();
// ... code ...
const t1 = performance.now();
console.log(`Took ${t1 - t0} ms`);
```

### Annotated Code Examples

#### Example 1: Measuring a Function

```js
performance.mark("start");
for (let i = 0; i < 1e6; i++) {}
performance.mark("end");
performance.measure("loop", "start", "end");
const [m] = performance.getEntriesByName("loop");
console.log(`Loop took ${m.duration} ms`);
```

### Real-World Cases

- **Core Web Vitals:** LCP, FID, CLS.
- **Benchmarking:** Comparing algorithms.
- **Monitoring:** Sending metrics to analytics.

---

## Intersection Observer

### Definitions

**Core Definition:** The Intersection Observer API asynchronously observes changes in the intersection of a target element with an ancestor element or the viewport.

**Technical Definition:** `new IntersectionObserver(callback, options)` with `root`, `rootMargin`, `threshold`. Callback receives `IntersectionObserverEntry` objects with `isIntersecting`, `intersectionRatio`, `boundingClientRect`.

**Beginner-Friendly Explanation:** Intersection Observer tells you when an element enters or leaves the screen — perfect for lazy loading images or triggering animations on scroll.

### Purposes

- To lazy-load images and content.
- To trigger animations when elements scroll into view.
- To implement infinite scrolling.
- To track ad visibility.

### Annotated Code Examples

#### Example 1: Lazy Loading Images

```html
<img data-src="photo.jpg" class="lazy" alt="">
<script>
  const observer = new IntersectionObserver((entries) => {
    entries.forEach((entry) => {
      if (entry.isIntersecting) {
        const img = entry.target;
        img.src = img.dataset.src;
        observer.unobserve(img);
      }
    });
  });
  document.querySelectorAll(".lazy").forEach((img) => observer.observe(img));
</script>
```

**Expected Output:** Images load only when they scroll into view.

### Real-World Cases

- **Lazy loading:** Medium, Twitter.
- **Infinite scroll:** Social feeds.
- **Analytics:** Viewport tracking.

---

## Mutation Observer

### Definitions

**Core Definition:** The Mutation Observer API watches for changes to the DOM tree and invokes a callback when mutations occur.

**Technical Definition:** `new MutationObserver(callback)` with `observe(target, config)` where config includes `childList`, `attributes`, `characterData`, `subtree`, `attributeOldValue`, `characterDataOldValue`, `attributeFilter`.

**Beginner-Friendly Explanation:** Mutation Observer tells you when the DOM changes — elements added, removed, attributes changed, text updated.

### Purposes

- To react to DOM changes.
- To implement custom elements that respond to attribute changes.
- To sanitise third-party content.

### Annotated Code Examples

#### Example 1: Watching for New Children

```js
const target = document.getElementById("container");
const observer = new MutationObserver((mutations) => {
  mutations.forEach((m) => {
    m.addedNodes.forEach((node) => console.log("Added:", node));
  });
});
observer.observe(target, { childList: true });
target.appendChild(document.createElement("div"));
```

### Real-World Cases

- **Custom elements:** Reacting to attribute changes.
- **Third-party scripts:** Detecting injected content.
- **Editors:** Tracking content changes.

---

## Resize Observer

### Definitions

**Core Definition:** The Resize Observer API reports changes to the dimensions of an element's content box or border box.

**Technical Definition:** `new ResizeObserver(callback)` with `observe(target)` and `unobserve()`. Callback receives `ResizeObserverEntry` objects with `contentRect` and `borderBoxSize`.

**Beginner-Friendly Explanation:** Resize Observer tells you when an element changes size — useful for responsive components that need to adapt to their container.

### Purposes

- To build responsive components.
- To adjust canvas size on resize.
- To implement container queries (pre-CSS).

### Annotated Code Examples

#### Example 1: Responsive Canvas

```js
const canvas = document.getElementById("c");
const observer = new ResizeObserver((entries) => {
  for (const entry of entries) {
    canvas.width = entry.contentRect.width;
    canvas.height = entry.contentRect.height;
  }
});
observer.observe(canvas);
```

### Real-World Cases

- **Charts:** Resizing on container change.
- **Maps:** Adjusting viewport.
- **Layout components:** Responsive dashboards.

---

## Payment Request API

### Definitions

**Core Definition:** The Payment Request API provides a standardised way to collect payment information from users.

**Technical Definition:** `new PaymentRequest(methods, details, options)` with `show()` returning a Promise. Supports basic card, Google Pay, Apple Pay, and other payment methods.

**Beginner-Friendly Explanation:** Payment Request shows the browser's native payment UI, so users don't have to type their card details on every site.

### Purposes

- To streamline checkout.
- To reduce friction in e-commerce.
- To support multiple payment methods uniformly.

### Annotated Code Examples

#### Example 1: Basic Payment Request

```js
const methods = [{ supportedMethods: "basic-card" }];
const details = { total: { label: "Total", amount: { currency: "USD", value: "10.00" } } };
const request = new PaymentRequest(methods, details);
request.show().then((response) => response.complete("success"));
```

**Note:** `basic-card` is deprecated in some browsers in favour of the Payment Handler API.

### Real-World Cases

- **E-commerce:** One-click checkout.
- **PWAs:** Native payment UX.

---

## Web Share API

### Definitions

**Core Definition:** The Web Share API allows web pages to share text, URLs, and files via the device's native sharing mechanism.

**Technical Definition:** `navigator.share(data)` returns a Promise. `navigator.canShare(data)` checks support. Requires secure context and user gesture. `navigator.share()` for files is limited.

**Beginner-Friendly Explanation:** Web Share lets users share content from your site to any app on their device — messaging, email, social — via the native share sheet.

### Purposes

- To enable native sharing.
- To share files (with support).
- To improve mobile UX.

### Annotated Code Examples

#### Example 1: Sharing a Link

```js
document.getElementById("share").addEventListener("click", async () => {
  if (navigator.share) {
    await navigator.share({ title: "My Page", url: location.href });
  }
});
```

### Real-World Cases

- **News sites:** Share articles.
- **Social apps:** Share content.
- **PWAs:** Native share integration.

---

## Web Components

### Definitions

**Core Definition:** Web Components are a suite of standards for creating reusable, encapsulated custom HTML elements.

**Technical Definition:** Comprising Custom Elements, Shadow DOM, HTML Templates, and Slots. Defined by WHATWG/W3C. Custom elements extend `HTMLElement`; Shadow DOM provides encapsulation; `<template>` and `<slot>` enable declarative content.

**Beginner-Friendly Explanation:** Web Components let you create your own HTML tags — like `<my-button>` — with their own styles and behaviour, reusable across projects.

### Sub-features

#### Custom Elements

**Definitions:**
- **Core Definition:** Custom Elements allow defining new HTML tags with custom behaviour.
- **Technical Definition:** `customElements.define(name, constructor)` registers a class extending `HTMLElement`. Lifecycle callbacks: `connectedCallback`, `disconnectedCallback`, `attributeChangedCallback`, `adoptedCallback`.
- **Beginner-Friendly Explanation:** Custom Elements are your own HTML tags with JavaScript behaviour.

**Purposes:**
- To create reusable components.
- To encapsulate behaviour in HTML tags.
- To integrate with frameworks.

**Syntax:**
```js
class MyButton extends HTMLElement {
  connectedCallback() { this.textContent = "Click me"; }
}
customElements.define("my-button", MyButton);
```

**Constraints:**
- Tag names must contain a hyphen.
- Cannot be redefined after registration.

**Example:**
```html
<my-button></my-button>
<script>
  class MyButton extends HTMLElement {
    connectedCallback() {
      this.textContent = "Click me";
      this.addEventListener("click", () => alert("Clicked!"));
    }
  }
  customElements.define("my-button", MyButton);
</script>
```
**Expected Output:** A button-like element labelled "Click me" that alerts on click.

**Real-World Cases:** Design systems, framework-agnostic UI libraries.

---

#### Shadow DOM

**Definitions:**
- **Core Definition:** Shadow DOM provides encapsulated DOM and style scoping for an element.
- **Technical Definition:** `element.attachShadow({ mode: "open" | "closed" })` returns a `ShadowRoot`. Styles inside do not leak out; outer styles do not leak in (except inherited properties).
- **Beginner-Friendly Explanation:** Shadow DOM is like a private room inside an element — its HTML and CSS don't interfere with the outside page.

**Purposes:**
- To encapsulate styles.
- To isolate DOM structure.
- To build self-contained components.

**Syntax:**
```js
const shadow = element.attachShadow({ mode: "open" });
shadow.innerHTML = "<style>p{color:red}</style><p>Hi</p>";
```

**Constraints:**
- Not all elements can host a shadow root (e.g., `<input>` cannot).
- Closed mode hides the shadow root from `element.shadowRoot`.

**Real-World Cases:** Native elements (`<video>`, `<input>`), design systems.

---

#### HTML Templates

**Definitions:**
- **Core Definition:** The `<template>` element holds HTML that is not rendered until cloned via JavaScript.
- **Technical Definition:** Content is stored in `template.content` (a `DocumentFragment`). Cloning with `cloneNode(true)` produces usable DOM.
- **Beginner-Friendly Explanation:** `<template>` is a reusable chunk of HTML you can stamp out whenever you need it.

**Purposes:**
- To define reusable markup.
- To improve performance by avoiding repeated parsing.

**Syntax:**
```html
<template id="card">
  <div class="card"><slot></slot></div>
</template>
```

**Real-World Cases:** Web Components, list rendering.

---

#### Slots

**Definitions:**
- **Core Definition:** `<slot>` elements are placeholders inside a shadow tree that users can fill with their own content.
- **Technical Definition:** Named slots (`<slot name="x">`) and default slots. Assigned nodes accessible via `slot.assignedNodes()`.
- **Beginner-Friendly Explanation:** Slots are holes in your component where users can plug in their own content.

**Purposes:**
- To allow content projection.
- To make components flexible.

**Syntax:**
```html
<!-- Component -->
<template>
  <div><slot name="header"></slot><slot></slot></div>
</template>
<!-- Usage -->
<my-card><h1 slot="header">Title</h1><p>Body</p></my-card>
```

**Real-World Cases:** Card components, layout components.

---

## WebAssembly

### Definitions

**Core Definition:** WebAssembly (Wasm) is a binary instruction format for a stack-based virtual machine, enabling high-performance execution of code compiled from languages like C, C++, Rust, and Go on the web.

**Technical Definition:** Defined by the W3C WebAssembly Community Group. Modules are loaded via `WebAssembly.instantiate()` or `instantiateStreaming()`. Exports/imports bridge Wasm and JavaScript. Memory is a linear `ArrayBuffer` (`WebAssembly.Memory`).

**Beginner-Friendly Explanation:** WebAssembly lets you run near-native-speed code in the browser, written in languages other than JavaScript.

### Purposes

- To run performance-critical code in the browser.
- To reuse existing C/C++/Rust libraries.
- To enable games, video editors, and CAD tools on the web.

### Syntax Rules and Structure

```js
const { instance } = await WebAssembly.instantiateStreaming(fetch("module.wasm"));
instance.exports.myFunction();
```

### Annotated Code Examples

#### Example 1: Loading a Wasm Module

```js
// Step 1: Fetch and instantiate.
const response = await fetch("add.wasm");
const { instance } = await WebAssembly.instantiateStreaming(response);
// Step 2: Call an exported function.
console.log(instance.exports.add(2, 3)); // 5
```

### Real-World Cases

- **Figma:** Uses Wasm for performance.
- **Google Earth:** Wasm for 3D rendering.
- **AutoCAD Web:** Wasm for CAD operations.

---

## Web Crypto API

### Definitions

**Core Definition:** The Web Crypto API provides cryptographic primitives for encrypting, decrypting, hashing, signing, and generating keys.

**Technical Definition:** Exposed via `crypto.subtle` (SubtleCrypto), with methods `encrypt`, `decrypt`, `sign`, `verify`, `digest`, `generateKey`, `deriveKey`, `importKey`, `exportKey`. `crypto.getRandomValues()` provides cryptographically secure random numbers. Requires secure context.

**Beginner-Friendly Explanation:** Web Crypto lets you do encryption and hashing in the browser securely, without third-party libraries.

### Purposes

- To encrypt/decrypt data.
- To hash passwords and data.
- To sign and verify messages.
- To generate keys.

### Syntax Rules and Structure

```js
const hash = await crypto.subtle.digest("SHA-256", data);
```

### Annotated Code Examples

#### Example 1: SHA-256 Hash

```js
const data = new TextEncoder().encode("hello");
const hashBuffer = await crypto.subtle.digest("SHA-256", data);
const hashArray = Array.from(new Uint8Array(hashBuffer));
const hashHex = hashArray.map((b) => b.toString(16).padStart(2, "0")).join("");
console.log(hashHex);
// 2cf24dba5fb0a30e26e83b2ac5b9e29e1b161e5c1fa7425e73043362938b9824
```

### Real-World Cases

- **Password hashing:** Client-side pre-hashing.
- **E2E encryption:** Signal Protocol implementations.
- **JWT signing:** Web Crypto for signatures.

---

## Web Bluetooth

### Definitions

**Core Definition:** The Web Bluetooth API allows web pages to communicate with Bluetooth Low Energy (BLE) devices.

**Technical Definition:** `navigator.bluetooth.requestDevice(options)` returns a `BluetoothDevice`. GATT services and characteristics are accessed via `device.gatt.connect()`. Requires secure context and user gesture.

**Beginner-Friendly Explanation:** Web Bluetooth lets your website talk to Bluetooth devices like heart rate monitors, smart bulbs, and fitness trackers.

### Purposes

- To connect to BLE peripherals.
- To read/write characteristics.
- To build IoT dashboards.

### Constraints

- Chrome/Edge only (not Safari or Firefox).
- Requires user permission per device.
- Limited to BLE (not classic Bluetooth).

### Annotated Code Examples

#### Example 1: Connecting to a BLE Device

```js
const device = await navigator.bluetooth.requestDevice({
  filters: [{ services: ["battery_service"] }],
});
const server = await device.gatt.connect();
const service = await server.getPrimaryService("battery_service");
const characteristic = await service.getCharacteristic("battery_level");
const value = await characteristic.readValue();
console.log("Battery:", value.getUint8(0), "%");
```

### Real-World Cases

- **Fitness:** Heart rate monitors.
- **Smart home:** Lights, locks.
- **Health:** Glucose meters.

---

## Web USB

### Definitions

**Core Definition:** The Web USB API allows web pages to communicate with USB devices.

**Technical Definition:** `navigator.usb.requestDevice({ filters })` returns a `USBDevice`. Control transfers, bulk transfers, and interrupt transfers are supported. Requires secure context and user gesture.

**Beginner-Friendly Explanation:** Web USB lets websites talk to USB devices like Arduino, keyboards, and custom hardware.

### Constraints

- Chrome/Edge only.
- Requires user permission.
- Not all USB devices supported (blocked devices list).

### Annotated Code Examples

#### Example 1: Listing USB Devices

```js
const device = await navigator.usb.requestDevice({ filters: [] });
console.log(device.productName, device.manufacturerName);
await device.open();
await device.selectConfiguration(1);
await device.claimInterface(0);
```

### Real-World Cases

- **Hardware dashboards:** Arduino, Raspberry Pi.
- **Firmware updates:** Flashing devices.
- **Custom peripherals:** POS systems.

---

## Web Serial

### Definitions

**Core Definition:** The Web Serial API allows web pages to communicate with serial devices (e.g., microcontrollers) over a serial connection.

**Technical Definition:** `navigator.serial.requestPort()` returns a `SerialPort`. `port.open({ baudRate })` opens the port. `port.readable` and `port.writable` are streams. Requires secure context and user gesture.

**Beginner-Friendly Explanation:** Web Serial lets websites talk to devices connected via USB serial — like Arduino boards.

### Constraints

- Chrome/Edge only.
- Requires user permission.

### Annotated Code Examples

#### Example 1: Reading from a Serial Port

```js
const port = await navigator.serial.requestPort();
await port.open({ baudRate: 9600 });
const reader = port.readable.getReader();
while (true) {
  const { value, done } = await reader.read();
  if (done) break;
  console.log(new TextDecoder().decode(value));
}
```

### Real-World Cases

- **Arduino IDE:** Web-based editor.
- **Robotics:** Controlling robots.
- **IoT:** Sensor data.

---

## Web MIDI

### Definitions

**Core Definition:** The Web MIDI API allows web pages to send and receive MIDI messages to and from connected MIDI devices.

**Technical Definition:** `navigator.requestMIDIAccess()` returns a `MIDIAccess` object with `inputs` and `outputs` maps of `MIDIInput` and `MIDIOutput`. Messages are `Uint8Array` MIDI packets.

**Beginner-Friendly Explanation:** Web MIDI lets websites talk to musical instruments, keyboards, and controllers.

### Constraints

- Chrome/Edge/Firefox (with permission).
- Requires user permission.

### Annotated Code Examples

#### Example 1: Listening for MIDI Notes

```js
const access = await navigator.requestMIDIAccess();
access.inputs.forEach((input) => {
  input.onmidimessage = (msg) => {
    const [status, note, velocity] = msg.data;
    console.log(`Note: ${note}, Velocity: ${velocity}`);
  };
});
```

### Real-World Cases

- **Music apps:** Synthesizers, sequencers.
- **DAWs:** Web-based digital audio workstations.
- **Education:** Learning music theory.

---

## WebXR

### Definitions

**Core Definition:** WebXR Device API provides access to virtual reality (VR) and augmented reality (AR) hardware from the web.

**Technical Definition:** `navigator.xr.requestSession("immersive-vr" | "immersive-ar" | "inline")` returns an `XRSession`. Rendering occurs via `XRSession.requestAnimationFrame()` with an `XRFrame` providing viewer pose and views. Uses WebGL for rendering.

**Beginner-Friendly Explanation:** WebXR lets websites create VR and AR experiences that work with headsets, phones, and AR glasses.

### Purposes

- To build VR experiences.
- To build AR experiences.
- To create immersive 3D applications.

### Constraints

- Requires compatible hardware and browser (Chrome, Edge, Meta Quest Browser).
- Requires secure context and user gesture.
- Performance-sensitive.

### Annotated Code Examples

#### Example 1: Checking XR Support

```js
if (navigator.xr) {
  const supported = await navigator.xr.isSessionSupported("immersive-vr");
  console.log("VR supported:", supported);
}
```

### Real-World Cases

- **VR shopping:** Virtual try-ons.
- **AR navigation:** Overlaying directions.
- **Training simulations:** Medical, industrial.

---

## References

- MDN Web Docs – Web APIs — https://developer.mozilla.org/en-US/docs/Web/API
- WHATWG HTML Living Standard – Canvas — https://html.spec.whatwg.org/multipage/canvas.html
- Khronos Group – WebGL Specification — https://www.khronos.org/webgl/
- W3C – SVG 2 Specification — https://www.w3.org/TR/SVG2/
- W3C – Web Audio API — https://www.w3.org/TR/webaudio/
- WHATWG – HTML Video — https://html.spec.whatwg.org/multipage/media.html
- W3C – Media Capture and Streams — https://www.w3.org/TR/mediacapture-streams/
- W3C – WebRTC 1.0 — https://www.w3.org/TR/webrtc/
- W3C – Geolocation API — https://www.w3.org/TR/geolocation/
- WHATWG – Notifications API — https://notifications.spec.whatwg.org/
- W3C – Clipboard API — https://www.w3.org/TR/clipboard-apis/
- W3C – File API — https://www.w3.org/TR/FileAPI/
- WHATWG – Drag and Drop — https://html.spec.whatwg.org/multipage/dnd.html
- WHATWG – History API — https://html.spec.whatwg.org/multipage/history.html
- W3C – Performance Timeline — https://www.w3.org/TR/performance-timeline/
- W3C – Intersection Observer — https://www.w3.org/TR/intersection-observer/
- WHATWG – Mutation Observer — https://dom.spec.whatwg.org/#mutation-observers
- W3C – Resize Observer — https://www.w3.org/TR/resize-observer/
- W3C – Payment Request API — https://www.w3.org/TR/payment-request/
- W3C – Web Share API — https://www.w3.org/TR/web-share/
- WHATWG – Web Components — https://html.spec.whatwg.org/multipage/custom-elements.html
- W3C – WebAssembly — https://www.w3.org/wasm/
- W3C – Web Cryptography API — https://www.w3.org/TR/WebCryptoAPI/
- W3C – Web Bluetooth — https://www.w3.org/TR/web-bluetooth/
- W3C – Web USB — https://www.w3.org/TR/webusb/
- W3C – Web Serial — https://www.w3.org/TR/serial/
- W3C – Web MIDI — https://www.w3.org/TR/webmidi/
- W3C – WebXR Device API — https://www.w3.org/TR/webxr/