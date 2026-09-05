---
name: keyword-timing-v2
description: Map real narration audio to millisecond keyword, number, phrase, question, and transition cues for storyboard and motion recommendations.
---

# Keyword Timing V2

Turn real narration into a compact set of meaningful cue times. These cues let
the main task recommend shot structure and let the video Agent synchronize
motion with speech.

## Inputs

Require:

- the real narration audio path;
- the approved script with stable sentence ids;
- `audio/voice-timing.json` normalized from Doubao service subtitles;
- the raw Doubao metadata path retained by `voice-producer-v2`;
- a project output directory.

Start only after real audio exists. Keep the approved wording and sentence ids
unchanged.

## Cue selection

Select cues that materially affect viewer understanding or visual timing:

- concepts and concrete keywords;
- exact numbers or named entities;
- phrases that should be revealed as a unit;
- questions that set up an answer;
- contrasts, turns, conclusions, and topic transitions.

Prefer a meaningful phrase over isolated filler words. Use the ordered `words`
timestamps from `audio/voice-timing.json`, with raw Doubao subtitle events only
when normalization needs clarification. Do not transcribe or realign audio again.
Never place a cue from reading speed alone. Keep every cue inside its sentence
interval.

Assign one useful motion intent:

- `emphasize` for a key term or number;
- `reveal` for a phrase or answer;
- `compare` for contrast;
- `transition` for a topic or argument turn;
- `hold` when the visual should stay readable through the phrase.

## Timing artifact

Write `timing/keywords.json` with integer milliseconds:

```json
{
  "audioPath": "audio/voiceover.mp3",
  "cues": [
    {
      "id": "cue-001",
      "sentenceId": "line-001",
      "text": "关键短语",
      "kind": "phrase",
      "startMs": 1200,
      "endMs": 2150,
      "motionIntent": "reveal"
    }
  ]
}
```

Allowed `kind` values are `keyword`, `number`, `phrase`, `question`, and
`transition`. Order cues by `startMs` and keep ids stable for downstream use.

## Report

Return the timing artifact as the main success:

```json
{
  "successes": [
    {
      "id": "keyword-timing",
      "path": "timing/keywords.json",
      "note": "18 meaningful cues"
    }
  ],
  "failures": [],
  "paths": ["timing/keywords.json"]
}
```

If required service timestamps are missing, report the affected sentence id and
reason. Preserve all usable cues and return immediately; a new paid synthesis
request requires the user's decision in the main task.

## Targeted refresh

For a narration redo, process only the affected sentence ids and their immediate
timing boundary. Return the refreshed cue path and affected ids for the targeted
video Agent.
