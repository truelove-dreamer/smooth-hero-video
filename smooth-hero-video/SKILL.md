---
name: smooth-hero-video
description: Optimize website hero/background video loading and playback performance. Use when a site has slow, stuttering, delayed, blank, or non-prioritized autoplay video; when the user wants the first screen video to load before UI animations; or when canvas/WebGL, GSAP, ScrollTrigger, filters, overlays, posters, parallax, or large media assets compete with a hero video.
---

# Smooth Hero Video

Use this skill to make a website's first-screen hero or background video appear quickly and play smoothly, especially on static hosting such as GitHub Pages.

## Workflow

1. Inspect the current video and page structure before changing behavior.
   - Find all video assets and references.
   - Check file size, duration, resolution, codec, bitrate, and whether the MP4 has fast-start metadata when tools are available.
   - Look for expensive first-screen competitors: full-page canvas/WebGL gradients, CSS blur/filter/backdrop-filter, blend modes, large overlays, heavy GSAP timelines, ScrollTrigger scrubbing, image posters, and parallax.

2. Prioritize the hero video as the first meaningful visual.
   - Use a black or very lightweight fallback background when the user wants the video to appear first.
   - Avoid heavy poster images unless the user explicitly wants a poster.
   - Preload the video in the document head when appropriate:

```html
<link rel="preload" as="video" href="./assets/hero.mp4" type="video/mp4">
```

   - Use autoplay-compatible markup:

```html
<video class="hero-video" autoplay muted loop playsinline preload="auto">
  <source src="./assets/hero.mp4" type="video/mp4">
</video>
```

3. Gate the opening animation on the first video frame.
   - Keep hero text and decorative UI hidden at initial load if the video must appear first.
   - Wait for `loadeddata` or `video.readyState >= 2`, then reveal UI and start the opening animation.
   - Avoid waiting for `canplaythrough`; it can delay the page too long on static hosting.

```js
const video = document.querySelector(".hero-video");
const reveal = () => document.body.classList.add("is-hero-ready");

if (video?.readyState >= 2) {
  reveal();
} else {
  video?.addEventListener("loadeddata", reveal, { once: true });
}
setTimeout(reveal, 3500);
```

4. Make the media web-friendly.
   - Prefer MP4/H.264 for reliable autoplay compatibility.
   - For background video, target 720p or lower unless the design truly needs more detail.
   - Keep bitrate moderate and duration short enough for fast first load.
   - Put the MP4 `moov` atom at the beginning (`faststart`) so playback can begin before the full file downloads.
   - Add WebM only when it is generated reliably and tested; use MP4 as fallback.
   - Do not rely on browser `MediaRecorder` transcoding for production assets unless no media tooling is available and the user accepts an experimental result.

5. Reduce first-screen render pressure.
   - Remove CSS `filter`, `backdrop-filter`, heavy blur, and unnecessary blend layers from video-adjacent elements.
   - Pause, hide, or make static any full-screen canvas/WebGL/animated gradient while the hero video is visible.
   - Prefer opacity and transform animations. Avoid layout-changing animation.
   - Use GSAP and ScrollTrigger with slower, smooth easing, but keep first-screen timelines lightweight until the video is ready.
   - Reduce parallax strength and scrub intensity if playback drops frames.
   - Use `transform: translateZ(0)` sparingly on the video layer only when it improves compositing.

6. Prevent startup scroll surprises when video and animations initialize.
   - Disable automatic scroll restoration when there is no URL hash.
   - Clear ScrollTrigger scroll memory after registering the plugin.
   - Disable CSS scroll anchoring if late-loading media shifts the viewport.
   - Preserve normal anchor behavior when the URL intentionally includes a hash.

```js
if (!window.location.hash && "scrollRestoration" in window.history) {
  window.history.scrollRestoration = "manual";
  window.scrollTo(0, 0);
}

ScrollTrigger.clearScrollMemory?.("manual");
```

```css
html {
  overflow-anchor: none;
}
```

7. Preserve visual quality without sacrificing playback.
   - Keep the video full-bleed or dominant if it is the hero.
   - Move dock cards, badges, and intro labels away from the highest-detail part of the video.
   - Use black load state plus delayed UI fade/reveal for a premium opening rather than showing unrelated static imagery first.

## Verification

When the user wants verification, check the page in a browser and inspect:

```js
const v = document.querySelector("video");
({
  readyState: v?.readyState,
  paused: v?.paused,
  currentTime: v?.currentTime,
  size: `${v?.videoWidth}x${v?.videoHeight}`,
  quality: v?.getVideoPlaybackQuality?.()
})
```

Success signs:

- The first visible background is black or the video frame, not an old poster/image.
- The video reaches `readyState >= 2` quickly.
- `currentTime` advances without repeated stalls.
- Dropped frames are low relative to total frames.
- Canvas/WebGL/decorative backgrounds are inactive or visually behind the hero while the hero video is prioritized.
- Opening animations do not pull the page away from the top on first load.

If the user says they will verify manually, skip browser verification and keep changes scoped to assets, markup, CSS, and startup sequencing.
