# HTMT — HyperText Markup Transcript

**A meeting recording is a gigabyte. The parts that matter are a megabyte.**

HTMT is an open file format for video and meeting transcripts that keeps the
conversation *and* the moments people were pointing at — and throws away the rest.

A one-hour screen-share is ~1 GB and effectively unsearchable. A plain transcript is
searchable but blind: it says *"look at this chart"*, and the chart is gone. An `.htmt`
file keeps both, opens natively in any browser, and typically weighs **1–2 MB**.

```
HTML describes documents.
HTMT describes conversations over time — including what was on screen when they happened.
```

> **Status: 1.0 design approved, implementation in progress.**
> The design document is [`docs/superpowers/specs/2026-09-01-htmt-1.0-design.md`](docs/superpowers/specs/2026-09-01-htmt-1.0-design.md).
> Nothing here is stable yet. Feedback on the format is very welcome.

---

## What it looks like

An `.htmt` file is a strict profile of HTML5 — no new parser, no build step, no plugin:

```html
<!doctype html>
<!-- HTMT/1.0 mode=single-file moments=12 duration=00:47:19 -->
<html lang="en" data-htmt="1.0">
<body>
<htmt-transcript title="Engineering Standup" date="2026-08-27">

  <htmt-turn id="t42" speaker="logan" start="00:13:42.220" end="00:13:49.810">
    Look at this chart — revenue dipped right here.

    <htmt-moment id="m3" t="00:13:44.500" kind="pointing" confidence="0.91">
      <figure>
        <img src="frames/m3.webp" width="1280" height="720"
             alt="Line chart of monthly revenue for 2026 with a pronounced dip in Q3">
        <figcaption>The revenue chart — Q3 dip</figcaption>
      </figure>
    </htmt-moment>
  </htmt-turn>

  <htmt-turn id="t43" speaker="nate" start="00:13:51">
    <htmt-answer to="q17">That dip is the reconciliation bug, not real revenue.</htmt-answer>
    <htmt-decision id="d1" status="accepted">Hold the board deck until it's fixed.</htmt-decision>
  </htmt-turn>

</htmt-transcript>
</body>
</html>
```

Double-click it. It renders. No viewer, no server, no JavaScript required.

## Why it's built this way

- **Browsers already are the viewer.** `.htmt` is HTML5, so every device on earth
  renders it. The format's identity lives in a required `data-htmt="1.0"` declaration
  and a sniffable `<!-- HTMT/1.0 … -->` signature comment, not in a custom doctype that
  would cost standards mode.
- **Images are real `<img>` elements, never attributes.** That one rule gives you the
  no-JavaScript fallback, single-file portability, screen-reader support, and `Ctrl-F`
  for free.
- **Optional everything.** A minimal valid document is a signature and some turns.
  Timestamps, semantics, media, and moments are all additive — and a sophisticated
  document stays readable to a simple processor.
- **Two packaging modes, one markup.** A directory (`.htmt` + `frames/`) for repos and
  archives; a single self-contained file with base64 frames for email and chat.

## Moments

The `<htmt-moment>` element is what the format exists for: a timestamped frame of what
was on screen at the instant someone referred to it, with a caption naming the referent.

Moments are found by a two-stage AI pipeline. A text pass reads the transcript for
deixis — *"look at this"*, *"right here"*, *"as you can see"* — and proposes candidate
time windows. A **vision pass then looks at the candidate frames and confirms something
is genuinely being shown** before a frame is kept, picking the sharpest representative
and writing the caption and alt text.

That second stage is the difference between a format you can trust and a folder of
frames grabbed at guessed timestamps. Figurative language — *"**this** quarter was
rough"* over a wall of faces — gets rejected instead of shipped.

## Planned tooling

```bash
npx htmt make meeting.mp4 --transcript meeting.vtt --single-file
npx htmt check meeting.htmt          # validator / conformance
npx htmt from-vtt lecture.vtt        # no AI, no video — the zero-cost on-ramp
npx htmt pack | unpack               # directory ⇄ single file, lossless
```

| Package | Purpose |
|---|---|
| `packages/core` | Parse, serialize, pack/unpack, shared types |
| `packages/viewer` | `htmt.css` + `htmt.js` — the moment deck (~7 KB, no dependencies) |
| `packages/cli` | The `htmt` binary |
| `fixtures/` | Conformance corpus — valid and deliberately broken documents |
| `spec/` | The specification |

The fixture corpus is the point of that table: it is the public conformance suite any
third-party implementation can run against.

## Prior art

`discovery/uploads/htmt-1.0-specification.html` is the original draft, which modelled
HTMT as a standalone XML-like language. Its conversation model — turns as the primary
unit, optional timestamps, the semantic vocabulary, the module system — is adopted
wholesale. What changed is the substrate: HTML5 rather than a new language, plus a
Vision module the draft did not have.

Compared to what exists today:

| Format | Answers | HTMT adds |
|---|---|---|
| SRT / VTT | What caption, and when? | Discourse structure, meaning, and the frames themselves |
| JSON exports | How does one vendor serialize? | A portable format that opens in a browser |
| Video | Everything, opaquely | The 0.1% that mattered, addressable and searchable |

> SRT/VTT: *"What was said, and when?"*
> HTMT: *"Who said what, when, in response to what, what did it mean, what was decided —
> and what were they pointing at?"*

## Contributing

The format is pre-1.0 and the design document is the best place to push back. Issues
arguing with element names, the `kind` taxonomy, the conformance rules, or the security
model are more valuable right now than code.

## License

Code is [MIT](LICENSE). The specification text under `spec/` and `docs/` is
[CC BY 4.0](LICENSE-SPEC).
