# Media Accessibility: Comprehensive Programming Cheat Sheet

---

## Topic Overview

### Definitions

**Core Definition**

Media accessibility is the practice of making audio and video content usable by everyone, including people who are deaf or hard of hearing, blind or visually impaired, and those with cognitive disabilities, through the provision of synchronized text alternatives, audio descriptions, and transcripts.

**Technical Definition**

Media accessibility is implemented through the HTML `<track>` element, which specifies timed text tracks for `<audio>` and `<video>` media elements. The `<track>` element uses the WebVTT (Web Video Text Tracks) format, a line-based text format for marking up external text track resources. The `kind` attribute of the `<track>` element defines the type of text track: `subtitles`, `captions`, `descriptions`, `chapters`, or `metadata`. WCAG Success Criteria 1.2.1 through 1.2.8 define the requirements for captions, audio descriptions, and media alternatives at Levels A, AA, and AAA.

**Beginner-Friendly Explanation**

Videos and audio need to work for everyone. If someone can't hear the audio, they need captions. If someone can't see the video, they need audio descriptions. If someone prefers reading, they need a transcript. The `<track>` element and WebVTT files are how you add these text alternatives to your media players.

---

### Key Characteristics

| Characteristic | Description |
|---|---|
| **Synchronized text** | `<track>` elements provide timed text that syncs with media playback |
| **Multiple track types** | `captions`, `subtitles`, `descriptions`, `chapters`, `metadata` |
| **WebVTT format** | Standard `.vtt` file format for timed text data |
| **Multiple languages** | Multiple `<track>` elements can provide different languages |
| **WCAG compliance** | Captions (1.2.2), audio descriptions (1.2.3, 1.2.5), transcripts (1.2.1, 1.2.3) |
| **Native browser support** | Browsers render and control tracks without JavaScript |

---

### Prerequisites

- Basic familiarity with HTML `<video>` and `<audio>` elements
- Understanding of HTML attributes and nesting
- Basic knowledge of accessibility principles (WCAG)

---

### Related Programming Areas

- **HTML Video** – `<video>` element hosts `<track>` children
- **HTML Audio** – `<audio>` element hosts `<track>` children
- **WebVTT API** – JavaScript API for text track manipulation
- **WCAG** – Success Criteria 1.2.1–1.2.8 define media accessibility requirements
- **Subtitling Tools** – Software for authoring WebVTT files

---

## Core Concepts / Features

---

### 1. The `<track>` Element

#### Definitions

**Core Definition**

The `<track>` element is used as a child of `<audio>` and `<video>` elements to specify timed text tracks (or time-based data) that are displayed in parallel with the media element.

**Technical Definition**

The `<track>` HTML element is used as a child of the media elements (`<audio>` and `<video>`). It lets you specify timed text tracks (or time-based data) that can be displayed in parallel with the media element, for example to automatically handle subtitles. It is a void element with an implicit ARIA role of `none`. Its DOM interface is `HTMLTrackElement`. It supports the `default`, `kind`, `label`, `src`, and `srclang` attributes. Multiple `<track>` elements can be specified for a single media element, each containing different kinds of timed text data or translations for different locales.

**Beginner-Friendly Explanation**

A `<track>` element adds text to a video or audio player. You put it inside a `<video>` or `<audio>` tag, and it points to a `.vtt` file that contains the text. You can add multiple tracks for different languages or different purposes (captions, subtitles, descriptions).

#### Purposes

- To add synchronized captions, subtitles, or descriptions to media
- To provide chapter navigation for media
- To supply metadata for scripts
- To make media accessible to users with disabilities

#### Syntax Rules and Structure

**General Syntax**

```html
<video controls>
    <source src="video.mp4" type="video/mp4">
    <track src="captions-en.vtt" kind="captions" srclang="en" label="English" default>
</video>
```

**Component Breakdown**

| Attribute | Description |
|---|---|
| `src` | URL of the WebVTT file |
| `kind` | Type of track: `subtitles`, `captions`, `descriptions`, `chapters`, `metadata` |
| `srclang` | Language of the track (BCP 47 language tag) |
| `label` | User-readable title for the track |
| `default` | Boolean; enables the track by default |

**Syntax Rules**

- `<track>` is a void element (no closing tag)
- `<track>` must be a child of `<audio>` or `<video>`
- `<track>` elements must come after all `<source>` elements
- The `kind` attribute defaults to `subtitles` if omitted
- Only one `<track>` per media element may have `default`
- The `srclang` attribute is required for `subtitles` and `captions` tracks

**Constraints and Limitations**

- Only one `chapters` track with `default` per media element is allowed
- The `src` attribute must point to a valid WebVTT file
- Not all browsers support all track kinds

#### Annotated Complete Step-by-Step Code Examples

**Example 1: Basic Captions Track**

```html
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="utf-8">
    <title>Track Demo</title>
</head>
<body>
    <video controls width="640" height="360">
        <source src="video.mp4" type="video/mp4">
        <track src="captions-en.vtt" kind="captions" srclang="en" label="English" default>
        <p>Your browser doesn't support HTML video.</p>
    </video>
</body>
</html>
```

**Expected Output**

The video player displays a "CC" button that toggles English captions. When enabled, captions appear synchronized with the audio.

**Why This Output Occurs**

The `<track>` element with `kind="captions"` tells the browser to load a caption track. The `default` attribute enables it automatically. The browser renders the captions over the video.

---

**Example 2: Multiple Tracks in Multiple Languages**

```html
<video controls width="640" height="360">
    <source src="video.mp4" type="video/mp4">
    <track src="captions-en.vtt" kind="captions" srclang="en" label="English" default>
    <track src="captions-fr.vtt" kind="captions" srclang="fr" label="French">
    <track src="descriptions-en.vtt" kind="descriptions" srclang="en" label="English Audio Description">
</video>
```

**Expected Output**

The player displays a menu allowing users to choose between English captions, French captions, and English audio descriptions.

**Why This Output Occurs**

Each `<track>` provides a different text resource. The `srclang` and `label` attributes let the browser present a language selection menu.

#### Real-World Cases

**Case 1: Streaming Platforms**

Netflix and YouTube use `<track>` elements to provide captions and subtitles in dozens of languages.

**Case 2: Educational Platforms**

Online courses use `<track>` elements for captions and transcripts.

**Case 3: Government Videos**

Government agencies use `<track>` elements to comply with Section 508 and WCAG.

---

### 2. Captions vs. Subtitles

#### Definitions

**Core Definition**

Captions provide a transcription of dialogue and important background sounds for deaf and hard-of-hearing users (`kind="captions"`), while subtitles provide a translation of spoken dialogue for users who can hear but don't understand the language (`kind="subtitles"`).

**Technical Definition**

The `kind` attribute of the `<track>` element indicates the kind of information in the timed text. `captions` text tracks provide a text version of dialogue and other sounds important to understanding the video. `subtitles` contain only the dialogue. Captions are suitable for when the soundtrack is unavailable (e.g., because it is muted or because the user is deaf). Subtitles are a transcription or translation of the dialogue, suitable for when the sound is available but not understood (e.g., because the user does not understand the language of the media resource's audio track).

**Beginner-Friendly Explanation**

Captions are for people who can't hear the audio. They include everything — dialogue, sound effects, music cues. Subtitles are for people who can hear but don't understand the language. They only include the spoken dialogue, translated.

#### Purposes

- **Captions**: To make audio content accessible to deaf and hard-of-hearing users
- **Subtitles**: To make content understandable to users who don't speak the language
- **Captions**: To include non-speech audio information (sound effects, music)
- **Subtitles**: To translate dialogue into other languages

#### Syntax Rules and Structure

**Captions Syntax**

```html
<track src="captions-en.vtt" kind="captions" srclang="en" label="English Captions" default>
```

**Subtitles Syntax**

```html
<track src="subtitles-es.vtt" kind="subtitles" srclang="es" label="Español">
```

**Comparison Table**

| Aspect | Captions | Subtitles |
|---|---|---|
| **`kind` value** | `captions` | `subtitles` |
| **Target audience** | Deaf, hard of hearing | Language learners, foreign audiences |
| **Content** | Dialogue + sound effects + music cues | Dialogue only (or translated) |
| **Soundtrack** | For when sound is unavailable | For when sound is available but not understood |
| **WCAG** | Required for SC 1.2.2 (Level A) | Not a WCAG requirement |

**Syntax Rules**

- Use `kind="captions"` for accessibility (deaf/hard of hearing)
- Use `kind="subtitles"` for translation
- The `srclang` attribute is required for both
- Captions should include non-speech audio information in the WebVTT file

**Constraints and Limitations**

- Some regions use "subtitle" to mean any visible text; WCAG recommends `captions` for accessibility
- Subtitles alone may not meet WCAG 1.2.2 if important non-speech audio is present

#### Annotated Complete Step-by-Step Code Examples

**Example 1: Captions for Accessibility**

```html
<video controls>
    <source src="video.mp4" type="video/mp4">
    <track src="captions-en.vtt" kind="captions" srclang="en" label="English Captions" default>
</video>
```

**WebVTT File (captions-en.vtt)**

```
WEBVTT

00:00:00.000 --> 00:00:05.000
[Music playing]

00:00:05.000 --> 00:00:10.000
Hello, and welcome to the presentation.

00:00:10.000 --> 00:00:15.000
[Audience applause]
```

**Expected Output**

Captions display dialogue, music cues, and sound effects.

**Why This Output Occurs**

The `kind="captions"` attribute tells the browser this track includes non-speech audio information.

---

**Example 2: Subtitles for Translation**

```html
<video controls>
    <source src="video.mp4" type="video/mp4">
    <track src="subtitles-es.vtt" kind="subtitles" srclang="es" label="Español">
</video>
```

**WebVTT File (subtitles-es.vtt)**

```
WEBVTT

00:00:05.000 --> 00:00:10.000
Hola, y bienvenidos a la presentación.
```

**Expected Output**

Spanish subtitles display the translated dialogue only.

**Why This Output Occurs**

The `kind="subtitles"` attribute tells the browser this track contains translated dialogue only.

#### Real-World Cases

**Case 1: News Broadcasts**

News sites use captions for accessibility and subtitles for international audiences.

**Case 2: Movie Streaming**

Streaming services offer both captions (for accessibility) and subtitles (for translation).

**Case 3: Corporate Training**

Training videos use captions for accessibility compliance and subtitles for multilingual workforces.

---

### 3. Audio Descriptions

#### Definitions

**Core Definition**

Audio descriptions are contextual descriptions of critical visual occurrences on screen, provided for blind or visually impaired users via text tracks (`kind="descriptions"`) that can be synthesized into speech or read aloud.

**Technical Definition**

The `descriptions` value for the `kind` attribute indicates a textual description of the video content, intended for audio synthesis when the visual component is obscured, unavailable, or not usable. This is suitable for blind or visually impaired users. Audio descriptions provide a verbal description of key visual elements in a video (actions, characters, scene changes, on-screen text) that are not apparent from the audio track alone. The descriptions are synchronized with the media and can be rendered as text-to-speech or displayed as text.

**Beginner-Friendly Explanation**

Audio descriptions explain what's happening on screen for people who can't see the video. For example, if a character walks into a room and sits down, the audio description would say "Maria enters the room and sits at the desk." These are added as a `descriptions` track.

#### Purposes

- To make visual content accessible to blind and visually impaired users
- To provide context for visual actions, expressions, and scene changes
- To satisfy WCAG Success Criterion 1.2.3 (Audio Description or Media Alternative, Level A)
- To satisfy WCAG Success Criterion 1.2.5 (Audio Description, Level AA)
- To describe on-screen text, charts, or diagrams

#### Syntax Rules and Structure

```html
<video controls>
    <source src="video.mp4" type="video/mp4">
    <track src="descriptions-en.vtt" kind="descriptions" srclang="en" label="English Audio Description">
</video>
```

**WebVTT File (descriptions-en.vtt)**

```
WEBVTT

00:00:00.000 --> 00:00:05.000
A title card appears reading "Introduction to Web Accessibility."

00:00:05.000 --> 00:00:10.000
A presenter stands in front of a slide showing the POUR principles.
```

**Component Breakdown**

| Attribute | Value |
|---|---|
| `kind` | `descriptions` |
| `srclang` | Language of the descriptions |
| `label` | User-readable title |
| `src` | URL of the WebVTT file |

**Syntax Rules**

- Use `kind="descriptions"` for audio descriptions
- The WebVTT file should describe only visual information not already conveyed by audio
- Descriptions should be timed to occur during pauses in dialogue
- The `srclang` attribute should match the language of the descriptions

**Constraints and Limitations**

- Audio descriptions are text-based; browsers may synthesize speech or display text
- Browser support for `kind="descriptions"` varies
- Descriptions should not duplicate information already in the audio track

#### Annotated Complete Step-by-Step Code Examples

**Example 1: Basic Audio Description Track**

```html
<video controls width="640" height="360">
    <source src="documentary.mp4" type="video/mp4">
    <track src="descriptions-en.vtt" kind="descriptions" srclang="en" label="English Audio Description">
</video>
```

**WebVTT File (descriptions-en.vtt)**

```
WEBVTT

00:00:02.000 --> 00:00:06.000
A sweeping aerial shot of a mountain range at sunrise.

00:00:10.000 --> 00:00:14.000
A hiker reaches the summit and raises both arms in triumph.
```

**Expected Output**

The browser may synthesize the descriptions as speech or display them as text, synchronized with the video.

**Why This Output Occurs**

The `kind="descriptions"` attribute tells the browser this track contains visual descriptions.

---

**Example 2: Descriptions with Multiple Languages**

```html
<video controls>
    <source src="video.mp4" type="video/mp4">
    <track src="captions-en.vtt" kind="captions" srclang="en" label="English Captions" default>
    <track src="descriptions-en.vtt" kind="descriptions" srclang="en" label="English Audio Description">
    <track src="descriptions-fr.vtt" kind="descriptions" srclang="fr" label="Description Audio en Français">
</video>
```

**Expected Output**

Users can choose between English and French audio descriptions.

**Why This Output Occurs**

Multiple `<track>` elements with `kind="descriptions"` provide descriptions in different languages.

#### Real-World Cases

**Case 1: Documentary Films**

Documentaries use audio descriptions to describe wildlife, landscapes, and visual storytelling.

**Case 2: Educational Videos**

Educational content uses descriptions to explain charts, diagrams, and demonstrations.

**Case 3: Theatre and Arts**

Streamed theatre performances use audio descriptions to describe stage actions and visual elements.

---

### 4. WebVTT File Format

#### Definitions

**Core Definition**

WebVTT (Web Video Text Tracks) is a line-based text file format with time-stamped cues used to provide captions, subtitles, descriptions, chapters, and metadata for HTML media elements.

**Technical Definition**

The WebVTT format (Web Video Text Tracks) is a format intended for marking up external text track resources. The main use for WebVTT files is captioning video content. WebVTT files provide captions or subtitles for video content, and also text video descriptions, chapters for content navigation, and metadata. A WebVTT file consists of cues, each with a start time, end time, and text content. The MIME type is `text/vtt`. WebVTT files are plain ASCII text files.

**Beginner-Friendly Explanation**

WebVTT is the file format you use to write captions and descriptions. It's a simple text file with time codes: "from this second to that second, show this text." You save it as a `.vtt` file and link to it from your `<track>` element.

#### Purposes

- To provide the text content for `<track>` elements
- To synchronize text with media playback
- To support captions, subtitles, descriptions, chapters, and metadata
- To enable multiple languages and track types

#### Syntax Rules and Structure

**General Syntax**

```
WEBVTT

[Optional cue identifier]
00:00:00.000 --> 00:00:05.000 [Optional cue settings]
Text content for this cue.

00:00:05.000 --> 00:00:10.000
More text content.
```

**WebVTT File Structure**

| Component | Description |
|---|---|
| `WEBVTT` | Mandatory header on the first line |
| Cue identifier | Optional; a name or number for the cue |
| Timing | `HH:MM:SS.mmm --> HH:MM:SS.mmm` |
| Cue settings | Optional; alignment, position, size |
| Cue text | The text to display |
| Blank line | Separates cues |

**Common Cue Settings**

| Setting | Values | Description |
|---|---|---|
| `align` | `start`, `center`, `end`, `left`, `right` | Text alignment |
| `position` | Percentage | Horizontal position |
| `size` | Percentage | Width of the cue box |
| `line` | Number or percentage | Vertical position |

**Syntax Rules**

- The first line must be `WEBVTT`
- Timestamps use the format `HH:MM:SS.mmm` (hours optional)
- The arrow `-->` separates start and end times
- Cues are separated by blank lines
- Text content can include HTML-like tags for formatting (`<b>`, `<i>`, `<u>`, `<c>`, `<v>`)

**Constraints and Limitations**

- WebVTT files must be UTF-8 encoded
- Special characters must be escaped
- The `-->` separator must be exact (no variations)
- Some browsers have limited support for advanced cue settings

#### Annotated Complete Step-by-Step Code Examples

**Example 1: Basic WebVTT File**

```
WEBVTT

1
00:00:00.000 --> 00:00:05.000
Welcome to the video.

2
00:00:05.000 --> 00:00:10.000
In this tutorial, we'll learn about WebVTT.

3
00:00:10.000 --> 00:00:15.000
[Music playing in background]
```

**Expected Output**

Three cues display sequentially as the video plays: the welcome message, the tutorial introduction, and a music note.

**Why This Output Occurs**

Each cue has a start time, end time, and text. The browser displays each cue during its time window.

---

**Example 2: WebVTT with Cue Settings**

```
WEBVTT

00:00:01.000 --> 00:00:04.000 align:start size:50%
- Never drink liquid nitrogen.

00:00:05.000 --> 00:00:09.000 align:end
- It will perforate your stomach.
- You could die.
```

**Expected Output**

The first cue appears at the start (left) and occupies 50% of the width. The second cue appears at the end (right).

**Why This Output Occurs**

Cue settings modify the position and size of the text within the video frame.

#### Real-World Cases

**Case 1: Broadcast Captions**

Broadcasters use WebVTT for streaming captions.

**Case 2: Online Courses**

Educational platforms use WebVTT for lecture captions and chapter markers.

**Case 3: Podcasts**

Podcast transcripts are converted to WebVTT for synchronized playback.

---

### 5. Transcripts

#### Definitions

**Core Definition**

Transcripts are fully readable, plain-text written descriptions of media content placed on the page to optimize broad cognitive accessibility and provide indexable search data.

**Technical Definition**

A transcript is a text version of the audio and visual information in a media file. WCAG Success Criterion 1.2.1 (Audio-only and Video-only, Prerecorded, Level A) requires that for prerecorded audio-only content, a text transcript is provided. For prerecorded video-only content, either a transcript or an audio description is required. WCAG Success Criterion 1.2.3 (Audio Description or Media Alternative, Level A) allows a text transcript as an alternative to audio description for synchronized media. Transcripts benefit users with cognitive disabilities, users who prefer reading, and search engines.

**Beginner-Friendly Explanation**

A transcript is the full text of what's said in a video or audio file, plus descriptions of important sounds and visual elements. Unlike captions (which appear on screen), a transcript is a separate document or section of the page that users can read at their own pace. It's great for accessibility and for SEO.

#### Purposes

- To provide a text alternative for audio-only content (WCAG 1.2.1)
- To provide a media alternative for video content (WCAG 1.2.3)
- To support users with cognitive disabilities who prefer reading
- To enable users to search and skim content
- To improve SEO by providing indexable text
- To support users who cannot play audio in their environment

#### Syntax Rules and Structure

**HTML Structure**

```html
<figure>
    <video controls>
        <source src="video.mp4" type="video/mp4">
        <track src="captions-en.vtt" kind="captions" srclang="en" label="English" default>
    </video>
    <figcaption>Video Title</figcaption>
</figure>

<details>
    <summary>Read the transcript</summary>
    <div class="transcript">
        <p><strong>Presenter:</strong> Welcome to the presentation.</p>
        <p>[Music playing]</p>
        <p><strong>Presenter:</strong> Today we'll discuss accessibility.</p>
    </div>
</details>
```

**Transcript Components**

| Component | Description |
|---|---|
| Speaker labels | Identify who is speaking |
| Dialogue | The spoken words |
| Sound effects | Descriptions of important non-speech audio |
| Visual descriptions | Descriptions of important visual information |
| Timestamps | Optional; link to specific points in the media |

**Syntax Rules**

- The transcript should be on the same page as the media, or linked prominently
- Speaker names should be clearly identified
- Non-speech audio should be described in brackets or italics
- Visual information important to understanding should be described
- The transcript should be searchable and indexable

**Constraints and Limitations**

- Transcripts are not synchronized with playback (unlike captions)
- Transcripts can be lengthy; consider using a `<details>` element to collapse them
- Transcripts should not replace captions for accessibility compliance

#### Annotated Complete Step-by-Step Code Examples

**Example 1: Audio-Only Transcript (Podcast)**

```html
<article>
    <h1>Podcast Episode 1: Introduction</h1>

    <audio controls>
        <source src="podcast.mp3" type="audio/mpeg">
    </audio>

    <h2>Transcript</h2>
    <div class="transcript">
        <p><strong>Host:</strong> Welcome to episode one of our accessibility podcast.</p>
        <p><strong>Guest:</strong> Thanks for having me.</p>
        <p><strong>Host:</strong> Let's start by defining web accessibility.</p>
        <p><em>[Background music plays]</em></p>
    </div>
</article>
```

**Expected Output**

The podcast player appears, followed by a full text transcript that users can read.

**Why This Output Occurs**

WCAG 1.2.1 requires a text transcript for prerecorded audio-only content. The transcript includes dialogue and sound effect descriptions.

---

**Example 2: Collapsible Transcript with `<details>`**

```html
<video controls>
    <source src="tutorial.mp4" type="video/mp4">
    <track src="captions-en.vtt" kind="captions" srclang="en" label="English" default>
</video>

<details>
    <summary>Transcript</summary>
    <div class="transcript">
        <p><strong>Instructor:</strong> In this lesson, we'll learn about the `<track>` element.</p>
        <p><strong>Instructor:</strong> The `<track>` element is used inside `<video>` or `<audio>`.</p>
        <p><em>[A diagram showing the relationship between the media element and track elements appears]</em></p>
    </div>
</details>
```

**Expected Output**

The video displays with captions. Below it, a collapsible "Transcript" section can be expanded to read the full text.

**Why This Output Occurs**

The `<details>` element keeps the transcript available but not overwhelming. The transcript includes dialogue and visual descriptions.

#### Real-World Cases

**Case 1: Podcasts**

Podcasts provide full text transcripts for accessibility and SEO.

**Case 2: Online Courses**

Course platforms provide transcripts alongside captions for students who prefer reading.

**Case 3: Government Meetings**

Public meeting recordings include transcripts for accessibility and public record.

---

## References

- MDN Web Docs – `<track>`: The Embed Text Track element – https://developer.mozilla.org/en-US/docs/Web/HTML/Reference/Elements/track
- MDN Web Docs – WebVTT API – https://developer.mozilla.org/en-US/docs/Web/API/WebVTT_API
- WHATWG HTML Living Standard – The track element – https://html.spec.whatwg.org/multipage/media.html#the-track-element
- W3C – WebVTT: The Web Video Text Tracks Format – https://www.w3.org/TR/webvtt1/
- W3C – H95: Using the track element to provide captions – https://www.w3.org/WAI/WCAG22/Techniques/html/H95
- W3C – Understanding SC 1.2.1 Audio-only and Video-only (Prerecorded) – https://w3c.github.io/wcag/understanding/audio-only-and-video-only-prerecorded
- W3C – Understanding SC 1.2.2 Captions (Prerecorded) – https://w3c.github.io/wcag/understanding/captions-prerecorded
- W3C – Understanding SC 1.2.3 Audio Description or Media Alternative (Prerecorded) – https://w3c.github.io/wcag/understanding/audio-description-or-media-alternative-prerecorded
- W3C – Understanding SC 1.2.5 Audio Description (Prerecorded) – https://w3c.github.io/wcag/understanding/audio-description-prerecorded
- W3C – WebVTT (Web Video Text Tracks) Format Reference – https://subtitleedit.github.io/
- W3C – Authoring WebVTT – https://dvcs.w3.org/hg/html/file/tip/webvtt/Overview.html
- web.dev – Media accessibility – https://web.dev/learn/accessibility/media
- WebAIM – Captions, Transcripts, and Audio Descriptions – https://webaim.org/techniques/captions/
- Deque University – `<video>` elements must have an audio description `<track>` – https://dequeuniversity.com/rules/axe/4.0/video-description