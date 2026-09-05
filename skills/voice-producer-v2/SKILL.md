---
name: voice-producer-v2
description: Use when an approved Chinese narration needs Doubao TTS audio, real word timestamps, sentence timing, or a targeted voiceover redo.
---

# Voice Producer V2

Generate the complete approved narration in one fresh Agent and publish real
timing for storyboard and motion decisions.

**REQUIRED SUB-SKILL:** Use `doubao-voiceover` for every synthesis request.
Follow its current command, credential, voice, format, timestamp, delivery, and
error-handling instructions. Do not select another TTS provider.

## Inputs

Require:

- the approved script with stable sentence ids;
- the confirmed Doubao speaker id, delivery style, speech rate, and format;
- the project output directory;
- pronunciation notes when the user supplied them.

Treat the supplied script and voice as fixed production choices. Keep all output
inside the project directory and use project-relative paths in the report.

## Doubao batch production

Create `audio/narration.txt` containing only the exact spoken text; do not include
sentence ids in speech. Keep the stable id-to-text mapping in
`audio/narration-map.json`. Then use `doubao-voiceover` once for the complete text
with subtitles enabled. Default to MP3 at 24000 Hz and standard speed unless the
approved choice says otherwise.

Use the confirmed `speaker` id. When the user supplied only a voice description,
the main task must first recommend and confirm a verified Doubao speaker id; do
not guess an id from a display name. Do not silently fall back to the sub-skill's
default speaker; that exact id is usable only when the main task recommended it
and the user approved it. Use the approved style instruction only with a standard
2.0 voice. A confirmed cloned voice uses `seed-icl-2.0` and does not receive a
style instruction.

One Agent owns the whole narration batch; do not create an Agent per sentence.
The expected primary files are `audio/voiceover.mp3` and the provider metadata
file `audio/voiceover.mp3.json` produced by `doubao-voiceover`.

Do not automatically repeat a Doubao synthesis request after any API attempt.
The service has no idempotency key, so reconnecting can duplicate charge. Return
the error, session or Log ID when available, and the recommended correction to
the main task. Local dry-run, metadata parsing, or duration probing may be rerun
because they do not request new audio.

When credentials are unavailable, run only the sub-skill's dry-run, report that
real synthesis has not happened, and request credential setup. Do not publish a
dry-run as a successful audio artifact.

## Real timing

Use the service subtitle events in `audio/voiceover.mp3.json` as the authoritative
word or character timing source. Measure the generated media duration with
`ffprobe` when available. Do not substitute estimated reading speed for either
timestamp source.

Map the approved sentence texts to the ordered service timestamps and write
`audio/voice-timing.json`:

```json
{
  "provider": "doubao",
  "speaker": "confirmed-speaker-id",
  "audioPath": "audio/voiceover.mp3",
  "sourceMetadataPath": "audio/voiceover.mp3.json",
  "totalDurationMs": 0,
  "sentences": [
    {
      "id": "line-001",
      "text": "approved narration text",
      "startMs": 0,
      "endMs": 0,
      "durationMs": 0
    }
  ],
  "words": [
    {
      "text": "真实时间戳词或字",
      "startMs": 0,
      "endMs": 0
    }
  ]
}
```

Use integer milliseconds. Each `durationMs` equals `endMs - startMs`, sentence
intervals remain ordered, and the last interval fits within `totalDurationMs`.
Preserve the raw provider metadata beside this normalized artifact.

If the service returns audio without subtitles, report `word-timestamps` as a
failure and keep the valid audio path. Do not invent timings. The main task asks
the user whether to make a new paid request after correcting the cause.

## Report

Return only useful production facts:

```json
{
  "successes": [
    {
      "id": "voiceover",
      "path": "audio/voiceover.mp3",
      "note": "totalDurationMs=42800"
    },
    {
      "id": "doubao-metadata",
      "path": "audio/voiceover.mp3.json",
      "note": "provider subtitle and usage events"
    },
    {
      "id": "voice-timing",
      "path": "audio/voice-timing.json",
      "note": "sentence timings recorded"
    }
  ],
  "failures": [],
  "paths": [
    "audio/voiceover.mp3",
    "audio/voiceover.mp3.json",
    "audio/voice-timing.json"
  ]
}
```

Include the total duration in the final message. When some sentences failed,
list their ids and reasons while preserving successful paths.

## Targeted redo

When assigned a local redo, create a separate text input containing only the named
sentences and use a new output name. Use the same approved Doubao speaker and
delivery settings. Report replacement audio, provider metadata, and updated
timing range so a new keyword-timing Agent can refresh affected cues. Never
overwrite the prior audio and never automatically repeat a paid request.
