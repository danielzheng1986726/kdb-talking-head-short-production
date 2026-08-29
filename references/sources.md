# Sources and provenance

This skill is a synthesis of repeated real productions. These sources explain why the guidance exists; they are not mandatory templates for a new video.

## Skill-writing method

- [Best practices for writing skills](https://github.com/grapeot/context-infrastructure/blob/main/rules/skills/bestpractice_skill_writing.md) by grapeot. The skill follows its result-determinacy approach: define the outcome, acceptance criteria, resources, boundaries, output contract, and observed failures without overfitting a rigid SOP.

## Production cases

- Five consecutive KDB one-pass talking-head iterations established the editorial workflow: conservative semantic editing, first-frame covers, explanatory graphics, context-aware openings, next-recording advice, and final delivery QA.
- The public Short [提高自己天花板最快的方法：先试一次你“不敢”的事](https://youtube.com/shorts/o08HjmNSEBE) supplied the latest end-to-end production evidence. Its real rework included overly small mobile type, an opening card that covered the face, wrapped chapter counters, iPhone HDR handling, caption retiming, and encoded-file QC.

## Existing implementations and adjacent skills

- [lizheng-video-production](https://github.com/sunyuzheng/lizheng-video-production) contains the existing KDB transcription, subtitle, filler-cut, title, and content-asset implementation. Use its `lizheng-video-editing` skill for broader interview, highlight, article, and channel-asset production.
- [HyperFrames](https://github.com/heygen-com/hyperframes) is the optional composition layer used for designed explanatory graphics.
- `talking-head-recut` is an optional runtime skill for graphic packaging after the spoken edit is locked. It is not bundled here and should be used only when it is installed in the current environment.

## External methodological comparison

- [Vincent Wei's `video-talkcraft`](https://github.com/Vincentwei1021/video-talkcraft), reviewed at commit `5d6637f1749bf046236c6b8b81eb2aa83f3499d3` on 2026-08-29, is a script-plus-finished-voiceover workflow for motion-designed explainer videos rather than an edit of a recorded mobile take. Its useful transferable ideas are semantic-beat planning, one primary visual job per beat, explicit visual handoffs, speech-anchored timing, measured face-safe regions, and temporal QA using settled and consecutive frames.
- This skill deliberately does not inherit `video-talkcraft`'s constant-motion requirement, mandatory treatment at every shot boundary, fixed visual language, prescribed sound-effect coverage, or default host-as-corner-chip composition. Those choices solve a different product and can conflict with a video-first, restrained talking-head edit.
- The upstream toolkit is distributed under PolyForm Noncommercial 1.0.0. No code, templates, motion cards, assets, or sound samples from that repository are copied into this skill; only the independently expressed editorial and QA principles above are cited and adapted.

Use this skill for end-to-end production of one recorded vertical talking-head short and the publication approval boundary. Use the adjacent tools only when their narrower job actually applies.

## Media boundary

No source MOV, rendered MP4, photographs, private transcripts, API credentials, browser state, platform cookies, or owner-local absolute paths belong in this public repository.
