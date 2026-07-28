# Smooth Hero Video Skill

`smooth-hero-video` is a Codex skill for improving hero and background video performance on websites.

It is useful when a site has:

- slow or blank first-screen video loading
- stuttering autoplay background video
- an old poster/image appearing before the video
- heavy GSAP, ScrollTrigger, canvas, WebGL, filters, overlays, or parallax competing with video playback
- a requirement that the video frame appears before the opening UI animation

## Install

Copy the skill folder into your Codex skills directory:

```powershell
Copy-Item -Recurse .\smooth-hero-video "$env:USERPROFILE\.codex\skills\smooth-hero-video"
```

Restart Codex or start a new task so the skill list refreshes.

## Use

Ask Codex something like:

```text
Use $smooth-hero-video to make this website hero video load first and play smoothly.
```

## Contents

```text
smooth-hero-video/
  SKILL.md
  agents/
    openai.yaml
```

The skill is intentionally small. It focuses on the workflow and decisions that matter most: diagnose the video and render stack, prioritize the first video frame, keep intro animation gated on video readiness, reduce render pressure, and verify dropped frames when needed.
