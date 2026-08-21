# Delivery and quality control

Use this reference for ingest, transcription, render preparation, final encoding, and handoff.

## Confirm the available local stack

Before a long run, check the actual environment for a media probe and encoder, the available local ASR and aligner, a color-managed macOS conversion path when relevant, fonts, and the selected composition renderer. Record the important versions in the run notes. Reuse a proven project-local implementation when it matches the source rather than retyping a complex media pipeline from memory.

If the preferred local ASR is unavailable, find another local adapter or report the dependency; do not silently send the recording to a cloud transcription service. If the preferred Apple color path fails, a tested FFmpeg tone-map may be used with stricter frame comparison. If there is no compatible audio track, stop instead of delivering silent or corrupt media.

## Inspect before transforming

Record at least:

- source path, size, duration, frame rate, encoded and displayed dimensions, and rotation;
- video codec, bit depth, color primaries, transfer, matrix, range, HDR or Dolby Vision side data;
- all audio tracks, default disposition, sample rate, channels, loudness, and true peak;
- chapters, timed metadata, location, device, creation time, and data tracks;
- decode errors, black or frozen intervals, opening and tail room, and visible sensitive content.

Generate a contact sheet across the full recording and inspect likely openings, transitions, screens, and endings. Technical metadata does not replace looking at the footage.

## Transcribe locally and preserve timing evidence

Default to a local Chinese ASR model with word-level alignment. Keep the raw ASR output alongside a corrected transcript; corrections should repair recognition and punctuation without silently rewriting the speaker.

Review:

- proper nouns, product names, English words inside Chinese speech, numbers, and negation;
- the exact audio at every proposed cut boundary;
- monotonic timestamps and source-duration bounds;
- captions after retiming, not only before editing.

ASR is an adapter, not an architectural dependency. If a different local model is more accurate or already available, use it while preserving the same evidence and outputs.

## Convert color, do not relabel it

iPhone portrait sources may be HEVC Main 10, BT.2020 HLG, and Dolby Vision Profile 8 even when the desired delivery is ordinary SDR. A renderer that merely emits H.264 or changes tags can leave skin washed out, highlights clipped, or the whole image too bright.

- Prefer a color-managed conversion that has been tested on representative frames. On macOS, AVFoundation or another Apple color-managed path may reproduce iPhone material better than a generic tone-map.
- A carefully configured FFmpeg `zscale` plus tone-map path can be a fallback, but it typically works from the HDR base layer rather than fully applying Dolby Vision RPU behavior.
- Compare opening, skin, white clothing, windows, and specular highlights against the source on actual rendered frames.
- Output explicit Rec.709 primaries, transfer, matrix, and range; remove stale HDR side data.

There is no universal tone-map preset. The acceptance test is the encoded picture, not the command string.

## Handle camera audio deliberately

Explicitly map the compatible default AAC track rather than every audio stream; iPhone files can contain an additional spatial audio track that common FFmpeg builds cannot decode.

Measure the edited program before normalization. If true peaks are already near 0 dBFS, compression or limiting must create headroom before raising integrated loudness. A social delivery around -18 to -14 LUFS with controlled true peak is a useful neighborhood, not a mandatory target independent of the material.

Check:

- left/right balance and channel choice;
- clicks or truncated consonants at hard cuts;
- A/V start time, end time, and drift;
- loudness and true peak on the final encode.

## Make captions for viewing, not transcription storage

Segment by short semantic unit and natural breath. Keep enough context to understand the line while avoiding dense two-line paragraphs. Correct mixed Chinese/English spacing and names. Use one visible subtitle system in the frame; platform subtitle tracks may still be uploaded for accessibility and search.

When retiming through cuts, bind a cue's start and end to the same retained source segment. A boundary lookup that maps the start to one segment and the end to another can turn a normal cue into a subtitle that remains on screen for tens of seconds.

Inspect captions in the encoded MP4 at:

- cut boundaries;
- full-screen graphics and PiP scenes;
- the widest and longest lines;
- the bottom platform-safe area;
- the first and last cues.

## Make frame zero the cover

The cover shown in a file browser or uploaded separately should match the first encoded frame of the final MP4. It may use a strong source frame from later in the recording, but the transition into the body must feel intentional.

Do not assume a generated JPEG and the actual first frame are identical. Extract frame zero from the final MP4 and compare them. The cover can last one frame or briefly hold; let the spoken opening determine the rhythm rather than imposing a fixed duration.

## Remove private and incompatible material

For a public delivery:

- explicitly map only the intended video and audio streams;
- drop camera data tracks, chapters, global and stream metadata, device and software labels, GPS, and creation timestamps unless deliberately retained;
- inspect filmed screens, notifications, certificates, QR codes, family material, and readable background text;
- preserve public proof while cropping, masking, blurring, or redrawing only what creates a real risk.

Do not upload the original phone MOV as a shortcut.

## Final QA

Before handoff or upload, verify the final file itself:

1. Decode completes without errors.
2. Container, video, and audio start at zero and end together within a harmless frame tolerance.
3. Display is 9:16, typically 1080×1920 at a stable delivery frame rate, H.264 plus AAC 48 kHz, and SDR Rec.709 when intended.
4. No accidental black opening, black tail, freeze, stale rotation, HDR side data, extra audio, or data streams remain.
5. Loudness, peaks, channel balance, and cut joins are acceptable.
6. Frame zero and cover match; title is readable at phone size.
7. Captions are correct, retimed, readable, and never stuck or duplicated.
8. Graphics explain what they claim to explain, do not hide the face unintentionally, and stay inside platform-safe zones.
9. Sensitive metadata and visible private information are absent.
10. Representative opening, middle, graphic transitions, and ending frames have been visually inspected.

A renderer's success code, a Studio preview, or a single contact sheet is not enough on its own. When one sampled frame suggests a crop, obstruction, or transition failure, inspect adjacent frames before diagnosing the whole scene. Keep a compact QA report with measured facts and the frames that support the visual judgment.

If the intended platform rejects the final duration, aspect, codec, or classification, do not silently cut meaning to satisfy it. Report the live constraint and offer the honest options: revise the edit with the user, publish in another supported format, or postpone that destination.

## Adaptable artifact contract

Names may follow the owning project, but keep the roles recognizable:

```text
project/
  source/                 # immutable original or provenance record
  work/
    inspect/              # technical report and contact sheets
    transcript/           # raw alignment and corrected transcript
    process/              # content map, cuts, captions, timeline map
  hf-project/             # optional editable HyperFrames composition
  output/
    *.ready-to-upload.mp4
    *.cover.jpg
    *.srt
    QA.md
    NEXT_TIME.md          # optional
```

Do not duplicate multi-gigabyte media merely to satisfy this shape. Keep a single owning copy or a clearly documented, verified derivative.
