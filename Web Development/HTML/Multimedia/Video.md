# HTML Video: Comprehensive Programming Cheat Sheet

---

## Topic Overview

### Definitions

**Core Definition**

HTML video is the native web platform capability for embedding, controlling, and playing video content directly within a document using the `<video>` element, without requiring external plugins.

**Technical Definition**

The `<video>` element is an HTML embedded content element defined by the WHATWG HTML Living Standard. It is used for playing videos or movies, and audio files with captions. It is categorised as flow content, phrasing content, and embedded content, and its DOM interface is `HTMLVideoElement`. The element may contain one or more video sources, represented using the `src` attribute or the `<source>` element, and the browser selects the most suitable one. It supports common media attributes including `controls`, `autoplay`, `loop`, `muted`, `poster`, `preload`, `playsinline`, `width`, and `height`.

**Beginner-Friendly Explanation**

The `<video>` tag lets you put a video player directly on a web page — no Flash, no plugins, no third-party libraries. You point to a video file, add the `controls` attribute, and the browser gives you a play button, volume slider, and seek bar. It works everywhere, from desktop browsers to phones.

---

### Key Characteristics

| Characteristic | Description |
|---|---|
| **Native playback** | Plays video without plugins or external libraries |
| **Multiple sources** | Can provide multiple file formats for browser fallback |
| **Built-in controls** | `controls` attribute provides play/pause, volume, and seeking UI |
| **Poster frame** | `poster` attribute shows a placeholder image before playback |
| **Autoplay restrictions** | Modern browsers block audible autoplay unless muted |
| **Mobile inline playback** | `playsinline` attribute prevents full-screen hijacking on iOS |
| **Dimension attributes** | `width` and `height` prevent Cumulative Layout Shift (CLS) |
| **JavaScript API** | Full playback control via `HTMLMediaElement` |

---

### Prerequisites

- Basic familiarity with HTML document structure
- Understanding of HTML elements, tags, and attributes
- Awareness of video file formats (MP4, WebM, Ogg) and MIME types
- Basic knowledge of the DOM and JavaScript (helpful for advanced control)

---

### Related Programming Areas

- **HTML Audio** – `<audio>` shares the same media element API
- **Web Audio API** – For advanced audio processing and synthesis
- **Media Source Extensions** – For adaptive streaming
- **Accessibility (A11y)** – Captions, transcripts, and keyboard-accessible controls
- **Web Performance** – `preload`, `poster`, and `loading` affect bandwidth and CLS

---

## Core Concepts / Features

---

### 1. The `<video>` Element

#### Definitions

**Core Definition**

The `<video>` element is the native HTML element used to embed a media player that supports video playback natively within the document layout.

**Technical Definition**

The `<video>` HTML element embeds a media player which supports video playback into the document. It is categorised as flow content, phrasing content, and embedded content. Its content model is: if it has a `src` attribute, zero or more `<track>` elements followed by transparent content; otherwise, zero or more `<source>` elements followed by zero or more `<track>` elements followed by transparent content. Both start and end tags are mandatory. Its DOM interface is `HTMLVideoElement`.

**Beginner-Friendly Explanation**

The `<video>` tag is the container for your video. You put the video file inside it, and the browser handles the rest. If the browser doesn't support video, you can put fallback text inside the tags.

#### Purposes

- To embed video content natively in a web page
- To provide video playback without external plugins
- To serve as a container for multiple video sources
- To enable JavaScript control over playback

#### Syntax Rules and Structure

**General Syntax**

```html
<video src="video.mp4" controls></video>
```

**Component Breakdown**

| Component | Description |
|---|---|
| `<video>` | Opening tag; container for video content |
| `src` | Optional; URL of the video file |
| `<source>` | Alternative to `src`; allows multiple formats |
| `<track>` | Optional; for captions, subtitles, descriptions |
| `Content` | Fallback content for unsupported browsers |
| `</video>` | Closing tag; required |

**Syntax Rules**

- Both start and end tags are mandatory
- At least one `src` attribute or `<source>` child must be present
- Fallback content inside `<video>` is shown in browsers that don't support it
- The element accepts global attributes plus media-specific attributes

**Constraints and Limitations**

- Autoplay is restricted by modern browsers (must be muted or user-initiated)
- Not all browsers support all formats (MP4, WebM, Ogg)
- The `<video>` element has no visual output unless `controls` is present or the video is playing

#### Annotated Complete Step-by-Step Code Examples

**Example 1: Basic Video Embed**

```html
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="utf-8">
    <title>Basic Video Demo</title>
</head>
<body>
    <h1>Basic video embed</h1>
    <video controls width="640" height="360" src="video.mp4">
        <p>
            Your browser doesn't support HTML video.
            <a href="video.mp4">Download the video here</a> instead.
        </p>
    </video>
</body>
</html>
```

**Expected Output**

A native video player appears with play/pause, volume, and seek controls. Clicking play begins playback.

**Why This Output Occurs**

The `controls` attribute tells the browser to display its default video player UI. The `src` attribute points to the video file. The `width` and `height` attributes define the display dimensions.

---

**Example 2: Multiple Source Formats**

```html
<video controls width="640" height="360">
    <source src="video.webm" type="video/webm">
    <source src="video.mp4" type="video/mp4">
    <p>Your browser doesn't support HTML video.</p>
</video>
```

**Expected Output**

The browser picks the first format it supports (WebM if supported, otherwise MP4) and plays it.

**Why This Output Occurs**

The browser evaluates each `<source>` element in order and selects the first one whose `type` it can play. This provides a fallback chain for broader compatibility. The `type` attribute is optional but should always be specified — it allows the browser to download only playable files.

#### Real-World Cases

**Case 1: Product Demonstrations**

E-commerce sites embed product videos using `<video>` with controls and poster images.

**Case 2: Educational Content**

Online courses embed lecture videos with captions via `<track>` elements.

**Case 3: Background Videos**

Hero sections use `<video>` with `autoplay muted loop playsinline` for background video.

---

### 2. Controls and Sizing Dimensions

#### Definitions

**Core Definition**

Controls and sizing dimensions are the attributes that display the browser's native playback UI and set explicit width and height dimensions to prevent Cumulative Layout Shift (CLS) while the video downloads.

**Technical Definition**

The `controls` attribute is a boolean attribute that, when present, instructs the browser to offer controls to allow the user to control video playback, including volume, seeking, and pause/resume playback. The `width` and `height` attributes define the display dimensions of the video's playback area in CSS pixels. When these attributes are set, the browser can reserve the correct aspect ratio before the video loads, preventing layout shifts. The `aspect-ratio` CSS property can also be used to reserve space.

**Beginner-Friendly Explanation**

The `controls` attribute gives you a play button, volume slider, and seek bar — all provided by the browser. The `width` and `height` attributes tell the browser how big the video will be so it can reserve that space before the video loads, preventing the page from jumping around.

#### Purposes

- To provide a visible, accessible playback interface
- To allow users to control playback without custom JavaScript
- To prevent Cumulative Layout Shift (CLS) by reserving space
- To define the intrinsic aspect ratio of the video

#### Syntax Rules and Structure

```html
<video controls width="640" height="360" src="video.mp4"></video>
```

**Component Breakdown**

| Attribute | Description |
|---|---|
| `controls` | Boolean; displays browser playback UI |
| `width` | Display width in CSS pixels |
| `height` | Display height in CSS pixels |

**Syntax Rules**

- `controls` is a boolean attribute; presence alone is sufficient
- `width` and `height` accept absolute values only (no percentages)
- Both `width` and `height` should be specified together to define the aspect ratio
- CSS can override the intrinsic dimensions but should preserve the aspect ratio

**Constraints and Limitations**

- Browser control styling cannot be deeply customised with CSS
- The `controlslist` attribute can remove specific controls (e.g., download, fullscreen)
- Without `width` and `height`, the video causes layout shift

#### Annotated Complete Step-by-Step Code Examples

**Example 1: Video with Controls and Dimensions**

```html
<video controls width="640" height="360" src="video.mp4">
    <p>Your browser doesn't support HTML video.</p>
</video>
```

**Expected Output**

A video player with standard controls, displayed at 640×360 pixels.

**Why This Output Occurs**

The `controls` attribute triggers the browser's native player UI. The `width` and `height` attributes define the display dimensions and reserve space before loading.

---

**Example 2: Aspect Ratio via CSS**

```html
<video controls src="video.mp4" style="aspect-ratio: 16 / 9; width: 100%;">
    <p>Your browser doesn't support HTML video.</p>
</video>
```

**Expected Output**

A responsive video that maintains a 16:9 aspect ratio.

**Why This Output Occurs**

The `aspect-ratio` CSS property reserves the correct space based on the ratio, preventing CLS.

#### Real-World Cases

**Case 1: Responsive Video Players**

Sites use `width` and `height` with CSS `max-width: 100%` for responsive playback.

**Case 2: CLS Optimization**

Performance-focused sites always set `width` and `height` on video elements to improve Core Web Vitals.

**Case 3: Custom Players**

Sites build custom controls and omit the `controls` attribute, using the JavaScript API instead.

---

### 3. Poster Images

#### Definitions

**Core Definition**

Poster images are placeholder graphics shown before the video stream is played, provided via the `poster` attribute.

**Technical Definition**

The `poster` attribute gives the URL of an image file that the user agent can show while no video data is available. The image is intended to be a representative frame of the video that gives the user an idea of what the video is like. If the attribute isn't specified, nothing is displayed until the first frame is available, then the first frame is shown as the poster frame.

**Beginner-Friendly Explanation**

A poster image is like a thumbnail or cover image for your video. It shows while the video is loading and gives viewers a preview of what the video is about. If you don't set one, the browser shows the first frame of the video instead.

#### Purposes

- To show a placeholder graphic while the video downloads
- To provide a visual preview of the video's content
- To improve perceived performance and user experience
- To maintain brand consistency with a custom thumbnail

#### Syntax Rules and Structure

```html
<video controls poster="poster.jpg" src="video.mp4"></video>
```

**Component Breakdown**

| Component | Description |
|---|---|
| `poster` | URL of the image to show before playback |

**Syntax Rules**

- The `poster` attribute must contain a valid non-empty URL
- The image should be a representative frame of the video
- If the poster fails to load, the browser falls back to the first frame
- The poster is shown until playback begins

**Constraints and Limitations**

- The poster is not shown once playback begins
- The poster does not affect the video's aspect ratio
- Some browsers may not show the poster after the video ends

#### Annotated Complete Step-by-Step Code Examples

**Example 1: Video with Poster**

```html
<video controls poster="thumbnail.jpg" width="640" height="360" src="video.mp4">
    <p>Your browser doesn't support HTML video.</p>
</video>
```

**Expected Output**

A thumbnail image appears before the video plays. Clicking play starts the video.

**Why This Output Occurs**

The `poster` attribute tells the browser to display `thumbnail.jpg` while no video data is available.

---

**Example 2: Poster with Multiple Sources**

```html
<video controls poster="poster.jpg" width="640" height="360">
    <source src="video.webm" type="video/webm">
    <source src="video.mp4" type="video/mp4">
    <p>Your browser doesn't support HTML video.</p>
</video>
```

**Expected Output**

The poster image displays until the browser selects a source and playback begins.

**Why This Output Occurs**

The `poster` attribute is independent of the source selection; it displays until the video is ready to play.

#### Real-World Cases

**Case 1: Video Streaming Platforms**

Streaming services use poster images as video thumbnails in catalogues.

**Case 2: Social Media Embeds**

Social media embeds use poster images to preview video content.

**Case 3: Product Videos**

E-commerce sites use product images as posters for demo videos.

---

### 4. Multiple Formats

#### Definitions

**Core Definition**

Multiple formats is the practice of delivering cross-browser compatibility by serving video files in varying modern containers and codecs.

**Technical Definition**

Not all browsers support the same video formats. The `<source>` element allows authors to specify multiple media resources for the `<video>` element, providing a fallback chain. The browser evaluates each `<source>` in document order and selects the first one it can play. The `type` attribute should always be specified, as it allows the browser to download only playable files. Common formats include MP4 with H.264, WebM with VP8/VP9/AV1, and Ogg with Theora.

**Beginner-Friendly Explanation**

Different browsers support different video formats. By providing multiple `<source>` elements, you give the browser a list of options. It picks the first one it understands. This ensures your video plays everywhere.

#### Purposes

- To provide fallback formats for browser compatibility
- To enable the browser to choose the best format
- To avoid relying on a single codec
- To optimize bandwidth by letting the browser download only playable files

#### Syntax Rules and Structure

```html
<video controls width="640" height="360">
    <source src="video.webm" type="video/webm">
    <source src="video.mp4" type="video/mp4">
    <source src="video.ogv" type="video/ogg">
    <p>Your browser doesn't support HTML video.</p>
</video>
```

**Component Breakdown**

| Component | Description |
|---|---|
| `<source>` | Void element; specifies a media resource |
| `src` | URL of the video file |
| `type` | MIME type of the video file |

**Syntax Rules**

- `<source>` is a void element (no closing tag)
- Each `<source>` must have `src`
- The `type` attribute is recommended for format negotiation
- Order matters: place most-supported formats first
- The browser stops at the first source it can play

**Constraints and Limitations**

- The browser evaluates sources in document order
- If no source is playable, the fallback content is shown
- Some formats have limited browser support (e.g., Ogg Theora)

#### Format Comparison

| Format | Container | Common Codecs | Browser Support |
|---|---|---|---|
| **MP4** | MP4 | H.264, H.265, AV1 | Universal |
| **WebM** | WebM | VP8, VP9, AV1 | Modern browsers |
| **Ogg** | Ogg | Theora, Vorbis | Limited (Firefox, Chrome) |

#### Annotated Complete Step-by-Step Code Examples

**Example 1: WebM and MP4 Fallback**

```html
<video controls width="640" height="360">
    <source src="video.webm" type="video/webm">
    <source src="video.mp4" type="video/mp4">
    <p>Your browser doesn't support HTML video. <a href="video.mp4">Download</a>.</p>
</video>
```

**Expected Output**

The browser plays WebM if supported, otherwise MP4, otherwise shows the fallback text.

**Why This Output Occurs**

The browser evaluates each `<source>` in document order and selects the first one whose `type` it supports.

---

**Example 2: Three-Format Fallback**

```html
<video controls width="640" height="360">
    <source src="video.av1.webm" type="video/webm">
    <source src="video.vp9.webm" type="video/webm">
    <source src="video.h264.mp4" type="video/mp4">
    <p>Your browser doesn't support HTML video.</p>
</video>
```

**Expected Output**

The browser selects the most advanced format it supports (AV1, then VP9, then H.264).

**Why This Output Occurs**

Each `<source>` provides a different codec within the same container. The browser picks the first playable one.

#### Real-World Cases

**Case 1: Video Streaming Services**

Netflix and YouTube serve multiple formats for different browsers and devices.

**Case 2: Educational Platforms**

Online courses provide MP4 and WebM fallbacks for broad compatibility.

**Case 3: News Websites**

News sites use MP4 as the primary format with WebM for modern browsers.

---

### 5. The `playsinline` Attribute

#### Definitions

**Core Definition**

The `playsinline` attribute forces videos to play inline within the document flow on mobile browsers instead of automatically hijacking the screen into a native full-screen player.

**Technical Definition**

The `playsinline` attribute is a boolean attribute that encourages the user agent to display video content within the element's playback area, rather than in fullscreen mode. On iPhone, `<video playsinline>` elements are allowed to play inline and will not automatically enter fullscreen mode when playback begins. `<video>` elements without `playsinline` attributes will continue to require fullscreen mode for playback on iPhone. It is required for autoplay on Safari iOS.

**Beginner-Friendly Explanation**

On mobile, especially iPhones, videos normally take over the whole screen when you press play. The `playsinline` attribute stops that — the video plays right in the page, like a background video. This is essential for background videos and inline playback.

#### Purposes

- To prevent full-screen hijacking on mobile
- To enable inline video playback in the document flow
- To support background video experiences on mobile
- To satisfy autoplay requirements on iOS Safari

#### Syntax Rules and Structure

```html
<video playsinline autoplay muted loop src="background.mp4"></video>
```

**Component Breakdown**

| Component | Description |
|---|---|
| `playsinline` | Boolean; enables inline playback on mobile |

**Syntax Rules**

- Boolean attribute; presence alone is sufficient
- Required for autoplay on Safari iOS
- Essential for background videos on mobile
- Works alongside `autoplay`, `muted`, and `loop`

**Constraints and Limitations**

- Only affects mobile browsers (primarily iOS Safari)
- Without `playsinline`, mobile browsers may force full-screen mode
- May not be supported in very old browsers

#### Annotated Complete Step-by-Step Code Examples

**Example 1: Inline Mobile Video**

```html
<video playsinline controls width="640" height="360" src="video.mp4">
    <p>Your browser doesn't support HTML video.</p>
</video>
```

**Expected Output**

On mobile, the video plays inline within the page rather than taking over the full screen.

**Why This Output Occurs**

The `playsinline` attribute tells mobile browsers to play the video in the element's playback area.

---

**Example 2: Background Video with Playsinline**

```html
<video playsinline autoplay muted loop poster="poster.jpg">
    <source src="background.webm" type="video/webm">
    <source src="background.mp4" type="video/mp4">
</video>
```

**Expected Output**

A muted, looping background video that plays inline on mobile.

**Why This Output Occurs**

The combination of `playsinline`, `autoplay`, `muted`, and `loop` enables a background video experience on mobile. The `muted` attribute satisfies autoplay policies, and `playsinline` prevents full-screen mode.

#### Real-World Cases

**Case 1: Hero Background Videos**

Landing pages use `playsinline` with `autoplay muted loop` for mobile-friendly background videos.

**Case 2: Social Media Embeds**

Social media platforms require `playsinline` for in-feed video playback.

**Case 3: E-Commerce Product Videos**

Product pages use `playsinline` to keep videos within the product layout on mobile.

---

### 6. Autoplay Policies

#### Definitions

**Core Definition**

Autoplay policies are the browser rules that restrict when video can play automatically, based on user engagement, muting, and other factors.

**Technical Definition**

Modern browsers have strict autoplay policies to improve user experience, reduce data usage, and reduce the incentive to install ad blockers. Chrome's autoplay policy is simple: muted autoplay is always allowed; audible autoplay is allowed when the user has interacted with the domain, when the user's Media Engagement Index (MEI) threshold has been exceeded, or when the user has added the site to the home screen or installed a PWA. The MEI measures the user's propensity to consume media on a site by calculating the ratio of significant media playback events to visits.

**Beginner-Friendly Explanation**

Browsers block videos that play sound automatically because it's annoying. But they always allow muted autoplay. If you want autoplay with sound, the user has to have interacted with your site before. The safest approach is to use `autoplay muted` and let the user unmute.

#### Purposes

- To improve user experience by preventing unwanted sound
- To reduce data usage on mobile networks
- To reduce the incentive to install ad blockers
- To give users control over playback

#### Autoplay Policy Rules

| Condition | Muted Autoplay | Audible Autoplay |
|---|---|---|
| **New page load** | ✅ Allowed | ❌ Blocked |
| **User interacted with domain** | ✅ Allowed | ✅ Allowed |
| **MEI threshold exceeded** | ✅ Allowed | ✅ Allowed |
| **PWA installed / added to home screen** | ✅ Allowed | ✅ Allowed |
| **Top-level frame delegates permission** | ✅ Allowed | ✅ Allowed |

**Syntax Rules**

- Always pair `autoplay` with `muted` for reliable autoplay
- Use `playsinline` for iOS compatibility
- Consider using `preload` to control bandwidth
- Provide an unmute button for user control

**Constraints and Limitations**

- Autoplay with sound is blocked on most browsers by default
- The MEI threshold varies by user and device
- Iframe delegation requires the Permissions Policy

#### Annotated Complete Step-by-Step Code Examples

**Example 1: Muted Autoplay (Always Allowed)**

```html
<video autoplay muted loop playsinline width="640" height="360">
    <source src="video.webm" type="video/webm">
    <source src="video.mp4" type="video/mp4">
</video>
```

**Expected Output**

The video plays automatically, muted, and loops. The user can unmute using the browser controls.

**Why This Output Occurs**

Muted autoplay is always allowed by Chrome and other modern browsers. The `playsinline` attribute ensures inline playback on iOS.

---

**Example 2: Autoplay with Unmute Button**

```html
<video id="myVideo" autoplay muted loop playsinline width="640" height="360">
    <source src="video.webm" type="video/webm">
    <source src="video.mp4" type="video/mp4">
</video>
<button id="unmuteButton">Unmute</button>

<script>
    document.getElementById('unmuteButton').addEventListener('click', function() {
        var video = document.getElementById('myVideo');
        video.muted = false;
    });
</script>
```

**Expected Output**

The video autoplays muted, and the user can click "Unmute" to hear the audio.

**Why This Output Occurs**

The video starts muted (allowed autoplay). The button gives the user control to unmute once they've interacted with the page.

#### Real-World Cases

**Case 1: Background Hero Videos**

Landing pages use `autoplay muted loop playsinline` for hero background videos.

**Case 2: Social Media Feeds**

Social platforms autoplay muted videos in feeds, with unmute buttons.

**Case 3: Product Demos**

Product pages autoplay muted demos with prominent unmute controls.

---

## References

- MDN Web Docs – `<video>`: The Video Embed element – https://developer.mozilla.org/en-US/docs/Web/HTML/Reference/Elements/video
- WHATWG HTML Living Standard – The video element – https://html.spec.whatwg.org/multipage/media.html#the-video-element
- MDN Web Docs – Video and audio content – https://developer.mozilla.org/en-US/docs/Learn_web_development/Core/Structuring_content/HTML_video_and_audio
- web.dev – `<video>` tag and `<source>` tag – https://web.dev/articles/video-and-source-tags
- web.dev – Lazy loading video – https://web.dev/articles/lazy-loading-video
- Chrome for Developers – Autoplay policy in Chrome – https://developer.chrome.com/blog/autoplay
- MDN Web Docs – Autoplay guide for media and Web Audio APIs – https://developer.mozilla.org/en-US/docs/Web/Media/Guides/Autoplay
- MDN Web Docs – Media containers – https://developer.mozilla.org/en-US/docs/Web/Media/Guides/Formats/Containers
- W3C – WCAG 2.1 – https://www.w3.org/TR/WCAG21/