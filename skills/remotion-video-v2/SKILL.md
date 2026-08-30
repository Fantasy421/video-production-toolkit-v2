---
name: remotion-video-v2
description: Build an editable voice-timed Remotion video with captions and cue-synced motion, then provide a preview and the user-approved export.
---

# Remotion Video V2

Use one fresh video Agent to turn approved narration, timing, storyboard, style,
and images into one editable Remotion project. Produce a reviewable preview
before the final export.

## Inputs

Require:

- the approved storyboard with shot ids and millisecond ranges;
- narration audio and real sentence timings;
- keyword cue times and motion intents;
- the approved style and motion note;
- completed illustrations and other approved resources;
- approved subtitle copy;
- target platform, aspect ratio, frame rate, and output directory.

Treat approved creative choices as production inputs. Use installed Remotion
guidance when detailed project, caption, media, animation, preview, or render
instructions are needed.

## Production order

The same video Agent owns the complete project:

1. Create or update one Remotion project inside the video project directory.
2. Set composition duration from the real narration duration.
3. Place narration and create captions from approved copy and real timing.
4. Build shots in storyboard order and place each approved visual resource.
5. Synchronize reveal, emphasis, compare, hold, and transition motion with the
   keyword cues.
6. For every shot longer than 5 seconds, implement its approved sustained
   information progression, composition change, or rich motion sequence.
7. Render a low-cost preview and return its path for decision point 4.
8. Render final output only after the user chooses export.

Keep the Remotion project editable. Use stable shot, cue, and asset ids in code
so targeted edits can find the intended range without redesigning the project.

## Captions and motion

Keep captions readable in the target safe area. Use sentence timing for caption
blocks and keyword timing for local emphasis. Motion should clarify the spoken
idea and preserve adequate reading time; it does not need to animate every word.

Match the approved style note for typography, palette, spacing, visual treatment,
and motion intensity.

## Preview and report

Return project and preview paths as soon as the preview exists:

```json
{
  "successes": [
    {
      "id": "remotion-project",
      "path": "video/remotion/",
      "note": "editable project"
    },
    {
      "id": "preview",
      "path": "video/preview.mp4",
      "note": "ready for decision point 4"
    }
  ],
  "failures": [],
  "paths": [
    "video/remotion/",
    "video/preview.mp4"
  ]
}
```

After approved export, add the export path with id `final-export`. Retry one
transient build or render failure once. After the second failure, return the
working project path, completed outputs, failed operation, and actionable reason.

## Targeted video redo

For subtitle, timing, motion, or asset-placement feedback, use a new targeted
video Agent. Change only the named shot, cue, caption, or asset range and render
a new preview. Keep earlier preview paths until the user accepts the replacement.

When upstream narration or illustrations changed, consume the replacement paths
and timing supplied by their targeted Agents; keep unaffected project ranges
stable.
