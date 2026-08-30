---
name: voice-producer-v2
description: Generate an approved narration as one consistent batch and report real sentence timing, total duration, successful audio paths, and failures.
---

# Voice Producer V2

Generate the complete approved narration in one fresh Agent and publish real
timing for storyboard and motion decisions.

## Inputs

Require:

- the approved script with stable sentence ids;
- the confirmed voice and delivery direction;
- the project output directory;
- pronunciation notes when the user supplied them.

Treat the supplied script and voice as fixed production choices. Keep all output
inside the project directory and use project-relative paths in the report.

## Batch production

Use one available TTS or voice tool appropriate to the confirmed choice. Keep one
voice configuration across the entire script. Produce one continuous master when
the tool supports it; sentence files may also be retained when they make timing
or targeted redo easier.

Preserve sentence ids from input to output. One Agent owns the whole narration
batch; do not create an Agent per sentence.

For a transient tool or network failure, retry the failed operation once. After
the second failure, retain all successful audio and return the failed sentence
ids with actionable reasons.

## Real timing

Measure the generated audio. Do not substitute estimated reading speed for media
duration. Write a project-relative timing artifact such as
`audio/voice-timing.json`:

```json
{
  "voice": "confirmed voice label",
  "audioPath": "audio/voiceover.wav",
  "totalDurationMs": 0,
  "sentences": [
    {
      "id": "line-001",
      "text": "approved narration text",
      "startMs": 0,
      "endMs": 0,
      "durationMs": 0
    }
  ]
}
```

Use integer milliseconds. Each `durationMs` equals `endMs - startMs`, sentence
intervals remain ordered, and the last interval fits within `totalDurationMs`.

## Report

Return only useful production facts:

```json
{
  "successes": [
    {
      "id": "voiceover",
      "path": "audio/voiceover.wav",
      "note": "totalDurationMs=42800"
    },
    {
      "id": "voice-timing",
      "path": "audio/voice-timing.json",
      "note": "sentence timings recorded"
    }
  ],
  "failures": [],
  "paths": [
    "audio/voiceover.wav",
    "audio/voice-timing.json"
  ]
}
```

Include the total duration in the final message. When some sentences failed,
list their ids and reasons while preserving successful paths.

## Targeted redo

When assigned a local redo, generate only the named sentence ids with the same
approved voice direction. Report the replacement audio and updated timing range
so a new keyword-timing Agent can refresh affected cues.
