# Image SEO — Comprehensive Programming Cheat Sheet & Research

---

## Topic Overview

### Definitions

**Core Definition**
Image SEO is the practice of optimizing images so that search engines — especially Google Images, Google Discover, and text-result images — can discover, crawl, index, and rank them for relevant queries. It combines technical markup (HTML image elements), textual metadata (file names, alt text, captions), machine-readable structured data (image sitemaps, schema.org, IPTC), and performance optimization (responsive images, modern formats).

**Technical Definition**
Google’s image SEO guidance states that the general technical requirements for Search apply to images, with additional requirements because images are a substantially different format from HTML: finding images is different, and the presentation of images influences whether they are indexed at all, and for the right keywords. Image SEO therefore operates across three layers: **discovery** (HTML `<img>` elements, image sitemaps, crawlable pages), **understanding** (descriptive file names, alt text, captions, surrounding text, structured data), and **presentation** (supported formats, responsive images, high resolution, appropriate aspect ratios). Google extracts information about the subject matter of an image from the content of the page, including captions and image titles, and places significant weight on the `alt` attribute as a metadata source.

**Beginner-Friendly Explanation**
Image SEO means helping search engines understand what your pictures show and where they belong. Google cannot “see” images the way people do, so it relies on clues: the file name, the alt text, the text around the image, the page title, and structured data. If you name your files descriptively, write meaningful alt text, place images near relevant content, and submit an image sitemap, Google can index your images and show them in image search results. Image SEO also improves accessibility, because alt text is what screen readers announce.

---

### Key Characteristics

- **HTML-first discovery**: Google finds images in the `src` attribute of `<img>` elements; CSS background images are not indexed.
- **Alt text is the strongest metadata signal**: Google uses alt text along with computer vision to understand images, and alt text also improves accessibility.
- **File names are light clues**: Descriptive file names give Google “very light clues” about subject matter; short but descriptive is better than `IMG00023.JPG`.
- **Image sitemaps extend discovery**: They tell Google about images it might not otherwise find, including images reached via JavaScript, and can include CDN-hosted URLs from other domains.
- **Supported formats matter**: Google Search supports images in BMP, GIF, JPEG, PNG, WebP, SVG, and AVIF.
- **Presentation affects indexing**: Generic images, images with extreme aspect ratios, and low-resolution images are less likely to perform well in visual search.
- **Structured data unlocks enhancements**: Schema.org image metadata and IPTC photo metadata can make images eligible for the Licensable badge and richer image details.

---

### Prerequisites

1. **Basic HTML image syntax**: familiarity with `<img src alt>`, `srcset`, and `<picture>`.
2. **Google Search Console access**: for submitting sitemaps and monitoring indexing.
3. **XML basics**: understanding sitemap structure and XML namespaces.
4. **Awareness of WCAG image accessibility**: alt text serves both accessibility and SEO.
5. **A CDN or image hosting setup**: for image sitemaps that reference other domains.

---

### Related Programming Areas

| Area | Relationship |
|---|---|
| **Technical SEO** | Crawlability, robots.txt, sitemaps, and page indexing directly determine image discovery. |
| **Image accessibility (WCAG 1.1.1)** | Alt text is the shared foundation of accessibility and image SEO. |
| **Structured data (Schema.org)** | `ImageObject`, `license`, and `acquireLicensePage` enable image enhancements. |
| **Responsive images** | `srcset`, `sizes`, and `<picture>` improve user experience and give Google fallback URLs. |
| **Core Web Vitals** | Image formats, compression, and lazy loading affect LCP and page experience. |

---

### Core Concepts / Features

The cheat sheet treats the following concepts, each with the same uniform structure:

1. **Image SEO** — the discipline
2. **File names** — descriptive naming conventions
3. **Alt text** — the primary textual metadata
4. **Image sitemaps** — extending image discovery
5. **Image SEO best practices** — consolidated guidelines

---

## 1. Image SEO — The Discipline

### Definitions

**Core Definition**
Image SEO is the set of techniques used to make images discoverable, understandable, and rankable in search engines, particularly Google Images, Google Discover, and text-result images.

**Technical Definition**
Google provides several Search features and products that help users visually discover information on the web, including text-result images, Google Discover, and Google Images. While each feature looks different, the general recommendations for getting images to appear in them are the same. Image SEO therefore focuses on **discovery** (HTML image elements, image sitemaps), **indexing** (crawlable pages, supported formats, accessible URLs), and **ranking** (alt text, file names, captions, surrounding content, structured data, resolution and aspect ratio).

**Beginner-Friendly Explanation**
Image SEO is how you get your pictures to show up when people search Google Images. It is not magic — it is a set of clear signals that tell Google what the image is, what page it belongs to, and whether it is worth showing.

---

### Purposes

- **To help Google discover images** that would otherwise be missed.
- **To help Google understand what an image depicts** and which queries it is relevant to.
- **To improve eligibility for visual search features** such as Google Images, Discover, and text-result images.
- **To drive qualified traffic** from image search to the page.
- **To support accessibility** by providing the same text alternatives that screen readers use.

---

### Syntax Rules and Structure

#### Complete General Syntax (Core Image Markup)

```html
<img src="descriptive-file-name.jpg" alt="Descriptive alt text" width="1200" height="800">
```

**Component breakdown**

| Component | Description |
|---|---|
| `<img>` | Standard HTML image element; Google indexes images in the `src` attribute. |
| `src` | The image URL; must be crawlable and in a supported format. |
| `alt` | The primary textual metadata signal and accessibility requirement. |
| `width` / `height` | Intrinsic dimensions; support responsive presentation and prevent layout shift. |

#### Syntax Rules

1. **Use HTML `<img>` elements**, not CSS background images, because Google does not index CSS images.
2. **Use supported formats**: BMP, GIF, JPEG, PNG, WebP, SVG, and AVIF.
3. **Place images near relevant text** and on pages relevant to the image’s subject matter.
4. **Provide a fallback `src`** when using `srcset` or `<picture>`, because some crawlers do not understand those attributes.
5. **Submit an image sitemap** to help Google discover images, including CDN-hosted images.

#### Constraints and Limitations

- **CSS images are not indexed**; if an image is content, it must be an `<img>` element.
- **JavaScript-discovered images may be missed** without an image sitemap.
- **Generic images and extreme aspect ratios perform poorly** in visual search.

---

### Multiple Annotated Complete Step-by-Step Code Examples

#### Example 1: A Basic, Indexable Image

**Setup Guide**: Save as `image-seo-basic.html`. Ensure the image URL is crawlable and the page is not blocked by `robots.txt` or a `noindex` directive.

```html
<!DOCTYPE html>
<html lang="en">
<head>
  <meta charset="UTF-8">
  <title>Image SEO Basics</title>
</head>
<body>
  <h1>Golden Retriever Puppies</h1>

  <!-- Standard <img> element with descriptive file name and alt text -->
  <img
    src="golden-retriever-puppy-playing-fetch.jpg"
    alt="A golden retriever puppy playing fetch in a grassy field"
    width="1200"
    height="800">
</body>
</html>
```

**Expected output**: The image is rendered on the page, and Google can discover it through the `<img src>` attribute. The descriptive file name and alt text give Google topical clues.

**Why this output occurs**: Google finds images in the `src` attribute of `<img>` elements, and it extracts subject-matter information from the page content, captions, titles, and alt text. The file name provides an additional light clue.

---

#### Example 2: Responsive Image with `srcset` and `src` Fallback

**Setup Guide**: Save as `image-seo-responsive.html`. Provide multiple image sizes.

```html
<img
  srcset="maine-coon-nap-320w.jpg 320w,
          maine-coon-nap-480w.jpg 480w,
          maine-coon-nap-800w.jpg 800w"
  sizes="(max-width: 320px) 280px,
         (max-width: 480px) 440px,
         800px"
  src="maine-coon-nap-800w.jpg"
  alt="A watercolor illustration of a Maine Coon napping leisurely in front of a fireplace">
```

**Expected output**: The browser selects an appropriate candidate for the viewport, and crawlers that do not understand `srcset` fall back to the `src` URL.

**Why this output occurs**: Google recommends always specifying a fallback URL using `src` because some browsers and crawlers do not understand `srcset`. The `alt` text remains the primary semantic description for search and accessibility.

---

### Real-World Cases

1. **E-commerce product pages**: Descriptive product image file names and alt text improve visibility in Google Images and Google Shopping.
2. **Recipe blogs**: Images named `chocolate-chip-cookies-baked.jpg` with alt text describing the finished dish attract image search traffic.
3. **News and media**: Article images placed near relevant text and submitted via image sitemaps improve Discover and Images visibility.

---

### References

- Google Search Central — Image SEO best practices — https://developers.google.com/search/docs/appearance/google-images
- Google Search Central — Image metadata in Google Images — https://developers.google.com/search/docs/appearance/structured-data/image-license-metadata
- Google Search Central — Image sitemaps — https://developers.google.com/search/docs/crawling-indexing/sitemaps/image-sitemaps

---

## 2. File Names — Descriptive Naming Conventions

### Definitions

**Core Definition**
Image file names are the names given to image files before upload. Google uses the file name as a **light clue** about the image’s subject matter, alongside alt text, captions, and surrounding page content.

**Technical Definition**
Google states that “the filename can give Google very light clues about the subject matter of the image. When possible, use filenames that are short, but descriptive”. Descriptive file names are part of the broader recommendation to “use descriptive filenames, titles, and alt text”. Best practice is to rename files **before uploading**, because many platforms lock or mangle file names after upload. Recommended conventions include lowercase, hyphen-separated words, specific descriptors, and matching the file extension to the actual file type.

**Beginner-Friendly Explanation**
File names are the first clue Google sees. `red-honda-cb650r-brooklyn.jpg` tells Google much more than `IMG_4521.jpg`. Use short, descriptive, lowercase names with hyphens between words. Rename the file before you upload it, not after.

---

### Purposes

- **To give Google a light topical clue** about the image’s subject matter.
- **To improve image search relevance** for descriptive, specific queries.
- **To support accessibility** when the file name is exposed in the URL or download.
- **To keep URLs clean and readable** for users and search engines.

---

### Syntax Rules and Structure

#### Complete General Syntax

```
descriptive-keywords-separated-by-hyphens.jpg
```

**Component breakdown**

| Convention | Recommendation |
|---|---|
| Length | Short but descriptive. |
| Separator | Hyphens (`-`), not underscores (`_`). |
| Case | Lowercase. |
| Words | Specific descriptors of what the image shows. |
| Extension | Match the actual file type (`.jpg`, `.png`, `.webp`, `.avif`). |
| Localisation | Translate file names when localising images, with URL encoding for special characters. |

#### Syntax Rules

1. **Use descriptive words** that reflect the image’s subject matter.
2. **Use hyphens, not underscores**; hyphens are the conventional word separator for URLs.
3. **Keep everything lowercase** to avoid case-sensitivity issues.
4. **Match the extension to the file type**; Google notes that it is a good idea for the extension to match the file type.
5. **Rename before uploading**, because many platforms lock or mangle file names after upload.
6. **Do not stuff keywords**; describe the image honestly.

#### Constraints and Limitations

- **File names are a light signal**, not a primary ranking factor; alt text and page content carry more weight.
- **Generic names like `image1.jpg`, `pic.gif`, and `1.jpg`** should be avoided.
- **Overly long file names** can be truncated or difficult to read.

---

### Multiple Annotated Complete Step-by-Step Code Examples

#### Example 1: Good vs Bad File Names

**Setup Guide**: Save as `file-names.html`. Provide images with the corresponding file names.

```html
<!-- GOOD: descriptive, lowercase, hyphenated -->
<img src="red-honda-cb650r-brooklyn.jpg" alt="A red Honda CB650R motorcycle parked in Brooklyn">

<!-- BAD: generic camera file name -->
<!-- <img src="IMG_4521.jpg" alt="A red Honda CB650R motorcycle parked in Brooklyn"> -->

<!-- BAD: underscores instead of hyphens -->
<!-- <img src="red_honda_cb650r_brooklyn.jpg" alt="A red Honda CB650R motorcycle parked in Brooklyn"> -->

<!-- BAD: keyword-stuffed -->
<!-- <img src="red-honda-cb650r-brooklyn-motorcycle-bike-motorbike-honda-cb650r-best-bike.jpg" alt="A red Honda CB650R motorcycle parked in Brooklyn"> -->
```

**Expected output**: The good file name gives Google a clear, concise clue about the image’s subject, while the bad examples are generic, use non-standard separators, or overstuff keywords.

**Why this output occurs**: Google recommends short, descriptive file names and warns against generic names. Hyphens are the standard separator, and keyword stuffing is discouraged.

---

#### Example 2: Localised File Names

**Setup Guide**: Save as `file-names-localised.html`.

```html
<!-- English page -->
<img src="red-motorcycle-brooklyn.jpg" alt="A red motorcycle parked in Brooklyn">

<!-- French page, localised file name -->
<img src="moto-rouge-brooklyn.jpg" alt="Une moto rouge garée à Brooklyn">
```

**Expected output**: The localised file name gives French-speaking users and Google’s French-language indexes a relevant clue.

**Why this output occurs**: Google recommends translating file names when localising images, keeping URL encoding guidelines in mind for non-Latin or special characters.

---

### Real-World Cases

1. **E-commerce**: Product images named `wireless-noise-cancelling-headphones-black.jpg` rank better than `product-123.jpg`.
2. **Real estate**: `3-bedroom-house-austin-texas.jpg` is more useful than `DSC_0012.jpg`.
3. **Photography portfolios**: `sunset-over-golden-gate-bridge.jpg` is descriptive and searchable.

---

### References

- Google Search Central — Use descriptive filenames, titles, and alt text — https://developers.google.com/search/docs/appearance/google-images#descriptive-filenames
- Google Search Central — Image SEO best practices — https://developers.google.com/search/docs/appearance/google-images

---

## 3. Alt Text — The Primary Textual Metadata

### Definitions

**Core Definition**
Alt text (alternative text) is the value of the `alt` attribute on an `<img>` element. It describes the image for people who cannot see it and provides Google with its strongest textual metadata signal for understanding the image’s subject matter.

**Technical Definition**
Google states that “the most important attribute when it comes to providing more metadata for an image is the alt text (text that describes an image), which also improves accessibility for people who can’t see images on web pages, including users who use screen readers or have low-bandwidth connections. Google uses alt text along with computer vision algorithms and the page content to understand the image”. Alt text is also useful as anchor text when an image is used as a link. Google warns against keyword stuffing alt text, which can cause a site to be seen as spam.

**Beginner-Friendly Explanation**
Alt text is the words you put in the `alt` attribute to describe the picture. It helps screen readers and it tells Google what the image is about. Write it naturally, describe what the image shows, and do not cram it with keywords. If the image is a link, the alt text should describe where the link goes.

---

### Purposes

- **To provide Google with its primary textual metadata** for understanding the image.
- **To improve accessibility** for screen-reader users and low-bandwidth users.
- **To serve as anchor text** when an image is used as a link.
- **To avoid spam signals** from keyword stuffing.

---

### Syntax Rules and Structure

#### Complete General Syntax

```html
<img src="puppy.jpg" alt="Dalmatian puppy playing fetch">
```

**Component breakdown**

| Component | Description |
|---|---|
| `alt="…"` | A useful, information-rich description of the image, in context. |
| Content | Natural language, not a keyword list. |

#### Syntax Rules

1. **Focus on useful, information-rich content** that uses keywords appropriately and in context.
2. **Do not fill alt attributes with keywords**; keyword stuffing can cause your site to be seen as spam.
3. **Use alt text as anchor text** when the image is a link; describe the link destination.
4. **For inline SVG, use the `<title>` element** instead of `alt`.
5. **Consider accessibility** per W3C guidelines when writing alt text.

#### Constraints and Limitations

- **Missing alt text is a bad practice**; Google explicitly labels `<img src="puppy.jpg"/>` as “Bad (missing alt text)”.
- **Alt text alone is not enough**; it works alongside file names, captions, surrounding text, and computer vision.
- **Decorative images** should use `alt=""` for accessibility, though Google’s guidance focuses on informative images.

---

### Multiple Annotated Complete Step-by-Step Code Examples

#### Example 1: Good vs Bad Alt Text

**Setup Guide**: Save as `alt-text.html`. Provide a puppy image.

```html
<!-- BAD: missing alt text -->
<!-- <img src="puppy.jpg"> -->

<!-- BAD: keyword stuffing -->
<!-- <img src="puppy.jpg" alt="puppy dog baby dog pup pups puppies doggies pups litter puppies dog retriever labrador wolfhound setter pointer puppy jack russell terrier puppies dog food cheap dogfood puppy food"> -->

<!-- BETTER: minimal but descriptive -->
<img src="puppy.jpg" alt="puppy">

<!-- BEST: natural and specific -->
<img src="puppy.jpg" alt="Dalmatian puppy playing fetch">
```

**Expected output**: The best alt text gives Google a clear, natural description of the image and provides screen-reader users with useful information. The keyword-stuffed version may trigger spam signals.

**Why this output occurs**: Google explicitly contrasts the bad, better, and best examples, recommending useful, information-rich alt text and warning against keyword stuffing.

---

#### Example 2: Alt Text for an Image Link

**Setup Guide**: Save as `alt-text-link.html`.

```html
<a href="/dalmatian-puppies">
  <img src="dalmatian-puppy.jpg" alt="Browse Dalmatian puppies for adoption">
</a>
```

**Expected output**: Google uses the alt text as anchor text for the link, and screen-reader users hear the link purpose.

**Why this output occurs**: Google states that alt text in images is useful as anchor text when an image is used as a link.

---

### Real-World Cases

1. **Product images**: Alt text like “Wireless noise-cancelling headphones in black” improves image search relevance.
2. **Recipe images**: Alt text like “Chocolate chip cookies cooling on a wire rack” describes the finished dish.
3. **News images**: Alt text like “Protesters holding signs outside city hall” gives the image journalistic context.

---

### References

- Google Search Central — Alt text — https://developers.google.com/search/docs/appearance/google-images#alt-text
- W3C — Images Tutorial — https://www.w3.org/WAI/tutorials/images/
- Google Search Central — Image SEO best practices — https://developers.google.com/search/docs/appearance/google-images

---

## 4. Image Sitemaps — Extending Image Discovery

### Definitions

**Core Definition**
An image sitemap is an XML sitemap that uses the Google Image Sitemaps namespace to tell Google about images on your site, especially images that Google might not otherwise find.

**Technical Definition**
Image sitemaps are a way of telling Google about other images on your site, especially those that Google might not otherwise find, such as images your site reaches with JavaScript code. You can create a separate image sitemap or add image sitemap tags to an existing sitemap; either approach is equally fine for Google. Unlike regular sitemaps, you can include URLs from other domains in the `<image:loc>` elements of image sitemaps, which lets you use CDNs to host images. The required tags are `<image:image>` (which encloses all information about a single image, up to 1,000 per `<url>`) and `<image:loc>` (the URL of the image).

**Beginner-Friendly Explanation**
An image sitemap is a list you give Google that says “here are all the images on my site and where they live.” It is especially useful for images that Google might miss — for example, images loaded by JavaScript or images hosted on a CDN. You can put image sitemap tags in your existing sitemap, so you do not need a separate file.

---

### Purposes

- **To tell Google about images it might not otherwise find**, including JavaScript-loaded images.
- **To include CDN-hosted images** from other domains in the sitemap.
- **To keep Google informed of changes** to your images and image pages.
- **To support crawling and indexing** of image content alongside regular page content.

---

### Syntax Rules and Structure

#### Complete General Syntax

```xml
<?xml version="1.0" encoding="UTF-8"?>
<urlset xmlns="http://www.sitemaps.org/schemas/sitemap/0.9"
        xmlns:image="http://www.google.com/schemas/sitemap-image/1.1">
  <url>
    <loc>https://example.com/sample1.html</loc>
    <image:image>
      <image:loc>https://example.com/image.jpg</image:loc>
    </image:image>
    <image:image>
      <image:loc>https://example.com/photo.jpg</image:loc>
    </image:image>
  </url>
  <url>
    <loc>https://example.com/sample2.html</loc>
    <image:image>
      <image:loc>https://example.com/picture.jpg</image:loc>
    </image:image>
  </url>
</urlset>
```

**Component breakdown**

| Component | Description |
|---|---|
| `<urlset>` | Root element; includes the image sitemap namespace. |
| `xmlns:image` | Declares the Image Sitemaps namespace `http://www.google.com/schemas/sitemap-image/1.1`. |
| `<url>` | A page URL containing images. |
| `<image:image>` | Encloses all information about a single image; up to 1,000 per `<url>`. |
| `<image:loc>` | The URL of the image; may be on another domain (e.g. a CDN) if both domains are verified in Search Console. |

#### Syntax Rules

1. **Declare the image namespace** `http://www.google.com/schemas/sitemap-image/1.1`.
2. **Use `<image:image>` for each image**; each `<url>` can contain up to 1,000 `<image:image>` tags.
3. **Use `<image:loc>` for the image URL**; the URL may be on another domain if that domain is verified in Search Console.
4. **General sitemap best practices apply** to image sitemaps.
5. **Deprecated tags** such as `<image:caption>`, `<image:geo_location>`, `<image:title>`, and `<image:license>` should not be used.

#### Constraints and Limitations

- **The image URL may be on another domain**, but you must verify both domains in Search Console.
- **`robots.txt` must not disallow crawling** of any content you want indexed.
- **Deprecated image sitemap tags** are no longer supported.

---

### Multiple Annotated Complete Step-by-Step Code Examples

#### Example 1: A Basic Image Sitemap

**Setup Guide**: Save as `image-sitemap.xml` and submit it in Google Search Console.

```xml
<?xml version="1.0" encoding="UTF-8"?>
<urlset xmlns="http://www.sitemaps.org/schemas/sitemap/0.9"
        xmlns:image="http://www.google.com/schemas/sitemap-image/1.1">
  <url>
    <loc>https://example.com/recipes/chocolate-chip-cookies</loc>
    <image:image>
      <image:loc>https://example.com/images/chocolate-chip-cookies-baked.jpg</image:loc>
    </image:image>
  </url>
</urlset>
```

**Expected output**: Google learns about the cookie image even if it was not discovered through normal crawling.

**Why this output occurs**: The `<image:loc>` provides the image URL, and the `<loc>` provides the page it belongs to, satisfying the required image sitemap structure.

---

#### Example 2: Image Sitemap with CDN-Hosted Images

**Setup Guide**: Save as `image-sitemap-cdn.xml`. Both `example.com` and `cdn.example.com` must be verified in Search Console.

```xml
<?xml version="1.0" encoding="UTF-8"?>
<urlset xmlns="http://www.sitemaps.org/schemas/sitemap/0.9"
        xmlns:image="http://www.google.com/schemas/sitemap-image/1.1">
  <url>
    <loc>https://example.com/products/wireless-headphones</loc>
    <image:image>
      <image:loc>https://cdn.example.com/products/wireless-headphones-black.webp</image:loc>
    </image:image>
  </url>
</urlset>
```

**Expected output**: Google discovers the CDN-hosted image and associates it with the product page.

**Why this output occurs**: Image sitemaps allow `<image:loc>` URLs from other domains, including CDNs, as long as both domains are verified in Search Console.

---

### Real-World Cases

1. **E-commerce**: Product images hosted on a CDN are included in the image sitemap alongside the product page URLs.
2. **JavaScript-heavy sites**: Images loaded dynamically are listed in the image sitemap so Google can discover them.
3. **Large catalogs**: Up to 1,000 images per `<url>` can be listed, making image sitemaps scalable.

---

### References

- Google Search Central — Image sitemaps — https://developers.google.com/search/docs/crawling-indexing/sitemaps/image-sitemaps
- Google Search Central — Image SEO best practices: Use an image sitemap — https://developers.google.com/search/docs/appearance/google-images#image-sitemap

---

## 5. Image SEO Best Practices

### Definitions

**Core Definition**
Image SEO best practices are the consolidated guidelines for helping Google discover, understand, and rank images — combining HTML image elements, descriptive file names, alt text, image sitemaps, supported formats, responsive markup, structured data, and appropriate resolution and aspect ratios.

**Technical Definition**
Best practices are drawn from Google Search Central’s image SEO documentation, image metadata guidance, and image sitemap reference. They include using HTML `<img>` elements (not CSS images), submitting image sitemaps, using supported formats, providing responsive images with a `src` fallback, using descriptive file names and alt text, avoiding generic images and extreme aspect ratios, using high-resolution images when possible, and adding structured data or IPTC photo metadata for enhancements.

**Beginner-Friendly Explanation**
The best practices are simple habits: use real `<img>` tags, give every image a descriptive file name and alt text, put images near relevant text, submit an image sitemap, use modern formats, provide responsive versions with a fallback, avoid generic images and weird shapes, use high-resolution images when you can, and add structured data if you want richer image results.

---

### Purposes

- **To maximise image discovery** through HTML elements and image sitemaps.
- **To maximise image understanding** through file names, alt text, captions, and surrounding text.
- **To improve eligibility for visual search features** such as Google Images and Discover.
- **To avoid common indexing and ranking pitfalls** such as CSS images, generic images, and extreme aspect ratios.
- **To support accessibility** through meaningful alt text.

---

### Syntax Rules and Structure

| Practice | Rule |
|---|---|
| **Use HTML `<img>` elements** | Google indexes images in the `src` attribute of `<img>`, not CSS backgrounds. |
| **Submit an image sitemap** | Helps Google find images it might otherwise miss, including CDN-hosted images. |
| **Use supported formats** | BMP, GIF, JPEG, PNG, WebP, SVG, and AVIF. |
| **Provide a `src` fallback** | Some browsers and crawlers do not understand `srcset` or `<picture>`. |
| **Use descriptive file names** | Short but descriptive; hyphens, lowercase; rename before upload. |
| **Write useful alt text** | Information-rich, in-context, not keyword-stuffed. |
| **Place images near relevant text** | Google extracts subject matter from captions, titles, and surrounding content. |
| **Avoid generic images and extreme aspect ratios** | Generic images and very narrow or very wide images perform poorly. |
| **Use high-resolution images when possible** | High resolution is recommended when possible. |
| **Add structured data or IPTC metadata** | Enables the Licensable badge and richer image details. |

#### Syntax Rules

1. **Discovery first**: Use `<img>` elements, crawlable URLs, and image sitemaps.
2. **Semantics second**: Use descriptive file names, alt text, captions, and surrounding text.
3. **Presentation third**: Use supported formats, responsive markup, high resolution, and appropriate aspect ratios.
4. **Enhancement fourth**: Add structured data or IPTC metadata for richer results.

#### Constraints and Limitations

- **Image SEO is not a substitute for page quality**; images are indexed in the context of the page.
- **Alt text is the strongest signal**, but it works alongside computer vision and page content.
- **Structured data requires maintenance**; every image instance may need its own markup.

---

### Multiple Annotated Complete Step-by-Step Code Examples

#### Example 1: A Fully Optimised Image Page

**Setup Guide**: Save as `image-seo-best-practice.html`. Provide a responsive image and structured data.

```html
<!DOCTYPE html>
<html lang="en">
<head>
  <meta charset="UTF-8">
  <title>Golden Retriever Puppies for Adoption</title>
</head>
<body>
  <h1>Golden Retriever Puppies for Adoption</h1>

  <!-- Descriptive file name, natural alt text, responsive srcset, src fallback -->
  <img
    src="golden-retriever-puppy-playing-fetch.jpg"
    srcset="golden-retriever-puppy-playing-fetch-400.jpg 400w,
            golden-retriever-puppy-playing-fetch-800.jpg 800w,
            golden-retriever-puppy-playing-fetch-1200.jpg 1200w"
    sizes="(max-width: 600px) 100vw, 50vw"
    alt="A golden retriever puppy playing fetch in a grassy field"
    width="1200"
    height="800">

  <p>
    This playful golden retriever puppy is available for adoption.
    See our <a href="/adoption-process">adoption process</a> for details.
  </p>
</body>
</html>
```

**Expected output**: The image is discoverable, understandable, and responsive; the descriptive file name and alt text give Google clear signals, and the `src` fallback ensures crawlers can access the image.

**Why this output occurs**: The markup combines HTML `<img>` discovery, descriptive file naming, natural alt text, responsive delivery with a fallback, and placement near relevant text — the core of Google’s image SEO guidance.

---

#### Example 2: Structured Data for Image Metadata

**Setup Guide**: Save as `image-metadata.html`. Add schema.org `ImageObject` markup.

```html
<div itemscope itemtype="https://schema.org/ImageObject">
  <img
    itemprop="contentUrl"
    src="golden-retriever-puppy-playing-fetch.jpg"
    alt="A golden retriever puppy playing fetch in a grassy field"
    width="1200"
    height="800">
  <meta itemprop="name" content="Golden Retriever Puppy Playing Fetch">
  <meta itemprop="description" content="A golden retriever puppy playing fetch in a grassy field.">
  <link itemprop="license" href="https://example.com/license">
</div>
```

**Expected output**: Google can read the image metadata, including the name, description, and license URL, and the image may become eligible for the Licensable badge.

**Why this output occurs**: Google supports schema.org image metadata, including the `license` and `acquireLicensePage` properties, to display creator and licensing information.

---

### Real-World Cases

1. **E-commerce product catalogs**: Descriptive file names, alt text, responsive images, and image sitemaps drive Google Images and Shopping visibility.
2. **News and media**: High-resolution images near relevant text, with structured data for licensing, improve Discover and Images performance.
3. **Recipe and lifestyle blogs**: Descriptive file names and natural alt text capture long-tail image search queries.

---

### References

- Google Search Central — Image SEO best practices — https://developers.google.com/search/docs/appearance/google-images
- Google Search Central — Image metadata in Google Images — https://developers.google.com/search/docs/appearance/structured-data/image-license-metadata
- Google Search Central — Image sitemaps — https://developers.google.com/search/docs/crawling-indexing/sitemaps/image-sitemaps
- Google Search Central — Use descriptive filenames, titles, and alt text — https://developers.google.com/search/docs/appearance/google-images#descriptive-filenames
- Google Search Central — Alt text — https://developers.google.com/search/docs/appearance/google-images#alt-text

---

## Summary Table of Image SEO Concepts

| Concept | Primary Signal | Key Rule | Common Failure |
|---|---|---|---|
| **Image SEO** | HTML `<img>`, page context, structured data | Use HTML images and submit image sitemaps | CSS background images are not indexed |
| **File names** | Light topical clue | Short, descriptive, lowercase, hyphens | Generic names like `IMG_4521.jpg` |
| **Alt text** | Primary textual metadata | Useful, information-rich, in-context | Missing alt text, keyword stuffing |
| **Image sitemaps** | Extended discovery | `<image:image>` and `<image:loc>` tags | Deprecated tags, unverified CDN domains |
| **Best practices** | Combined | HTML elements, sitemaps, formats, responsive, structured data | Generic images, extreme aspect ratios |

---

## Final Notes

- **Alt text is the strongest image SEO signal**, and it also serves accessibility; there is no trade-off between the two.
- **File names are a light clue**, not a primary ranking factor; use them to support, not replace, alt text and page content.
- **Image sitemaps extend discovery**, especially for JavaScript-loaded images and CDN-hosted images.
- **Supported formats and responsive markup** improve both indexing and user experience.
- **Structured data unlocks enhancements** such as the Licensable badge and richer image details.
- **Validate with Search Console**: Submit image sitemaps, monitor indexing, and use the URL Inspection tool to confirm that Google can access and render your images.

---

This cheat sheet has covered Image SEO — the discipline, file names, alt text, image sitemaps, and best practices — with uniform definitions, purposes, syntax rules, annotated examples, real-world cases, and authoritative references. Use it as a reference and a teaching aid, and validate image SEO against Google Search Central’s image SEO documentation, image sitemap reference, and image metadata guidance.