# AGENTS.md

Guidance for coding agents (and people) working with this repository.

## What this repo is

A small library of reusable prompt blocks for coding models. Each block is a section of prompt text that fixes a defect class models keep producing on a kind of task, plus a full test prompt and real results from several models.

- `fog/`: volumetric ground fog that drifts, catches light and is stirred by moving objects.
- `walk/`: procedural humanoid characters with walk, run and fight-stance animation (two blocks: character model and walk).
- `index.html`: the gallery page for all results. It is served at https://loktar00.github.io/prompt-blocks/

## Using a block in your own task

1. Read the block file (`fog/FOG-BLOCK.md`, `walk/CHARACTER-MODEL-BLOCK.md`, `walk/WALK-BLOCK.md`).
2. Append it to your prompt after your own scene description. Keep its numbers as written: each one came from a defect seen in real runs.
3. Or send a block folder's `PROMPT.md` as is, as one message, to test a model on the same task as the published results.

Blocks are written for three.js in a single self-contained HTML file. If you target another engine, keep the constraints and translate the API calls.

## Repository layout

```
<block>/
  <NAME>-BLOCK.md        the block text
  PROMPT.md              full test prompt containing the block(s)
  showcase.mp4/.jpg      optional showcase video and poster
  results/<model>/
    index.html           the model's page, unmodified apart from removing machine-specific details
    <model>.mp4          a recording of that page
    <model>.jpg          poster frame for the video
```

## Adding a result

1. Run the block's `PROMPT.md` on the model. Record which harness ran it (one-shot, or the agent harness name).
2. Put the page and recording under `<block>/results/<model>/` using the layout above. Keep each file under 90 MB.
3. Add the model to the block's table and to the results section in `README.md`, in the same format as the existing entries (clickable poster linked to the github.io mp4, plus a "Run it live" link). Add a card to `index.html`.
4. Check that every new link returns 200 on GitHub Pages after the push.

## Rules

- Never commit secrets, API keys, model endpoints, internal IP addresses or local file paths. Scrub them from result pages before adding them.
- Don't hand-edit a model's result page to improve it. Results are shown as produced.
- Don't add raw audio files.
- One commit per logical change, with a plain commit message.
