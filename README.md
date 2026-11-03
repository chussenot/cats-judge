# cats-judge

Typed, calibrated verdicts on photos from a small local vision model: is
there a cat in the picture, and what is its main subject, served by
`transformers serve` and read through a [`jud`](https://github.com/chussenot/judgment)
rubric whose gate says when to trust the answer.

The model is [`Qwen/Qwen2-VL-2B-Instruct`](https://huggingface.co/Qwen/Qwen2-VL-2B-Instruct),
a 2B vision-language model (Apache 2.0) that reads the photo and answers
each question with one token. It runs on a CPU in about seven seconds per
photo and separates the 26 labelled cases cleanly: every cat at 0.82 or
more, every other photo at 0.13 or less.

The companion to [oncall-judge](https://github.com/chussenot/oncall-judge),
which judges text: the same wire, the same loop, with an image in the
request. What changes when a request carries an image is on
[Visual judgments](https://github.com/chussenot/judgment/blob/main/docs/concepts/visual-judgments.md).

## Setup

[mise](https://mise.jdx.dev) installs the tools and runs the tasks:

```sh
mise trust && mise install      # transformers[serving,torch], jud and jaq
```

`jud` reads its backend from `~/.config/jud/config.yaml`; there is no
environment variable for the model name or the image transport, so the
file is required:

```yaml
base_url: http://localhost:8000              # the client appends /v1/systemone
model: Qwen/Qwen2-VL-2B-Instruct
api_key: unused                              # the server ignores it; jud refuses to run without one
timeout_secs: 120                            # a CPU takes a few seconds per photo, more on the first call
shared_images: data-urls                     # images as a top-level `images` array of data: URLs, the shape the server takes
```

`jud config` prints what a run will use.

## Use

```sh
mise run serve                                                   # loads the model, listens on :8000
echo '{"photo": "p"}' | jud --asset p=images/persian-98.jpg cat.jud
mise run judge images/pug-57.jpg                                 # the same, as a task
mise run describe images/pug-57.jpg                              # the chat endpoint's one-sentence caption, for comparison
```

A verdict, for `images/persian-98.jpg`:

```json
{
  "cat_present": {"verdict": "yes", "probability": 0.91},
  "subject": {"verdict": "deferred", "fallback": null, "nearest": "cat", "confidence": 0.71, "bar": 0.8}
}
```

And for `images/pug-57.jpg`:

```json
{
  "cat_present": {"verdict": "no", "probability": 0.09},
  "subject": {"verdict": "deferred", "fallback": null, "nearest": "dog", "confidence": 0.25, "bar": 0.8}
}
```

`cat_present` is the verdict: `yes` from the policy's threshold of 0.5.
`subject` is a diagnostic beside it, deferred below a confidence of 0.8,
which on this model is most of the time (see [Picking the model](#picking-the-model)).

The server speaks two wires. `POST /v1/chat/completions` generates text;
`mise run describe` sends a photo there and prints the caption. `POST /v1/systemone`
is what `jud` calls: no text is generated, each allowed answer is one token
and its probability is read from a single forward pass.

## Files

| Path | What |
|---|---|
| `mise.toml` | tools and the `serve`, `judge` and `describe` tasks |
| `cat.jud` | the rubric: a `noul` for whether a cat is in the photo, a `choice` over what the main subject is, and the tuned gates |
| `cat-cases.jud` | twenty-six labelled photos: twelve cats, seven dogs, one fish, six scenes with no animal |
| `images/` | the photos, re-encoded within the wire's limits; `images/SOURCES.md` names the source and licence of each |
| `recordings/` | the model's recorded answers to those cases, so `jud eval` and `jud tune` run with no server |
| `recordings-smolvlm/` | the same cases answered by `HuggingFaceTB/SmolVLM-500M-Instruct`, the other candidate, for `jud eval --against` |

## How an image gets to the model

The rubric asks about the image by a reference the state carries, `photo`,
in backticks; nothing in the rubric names a path, a size or a format.
`--asset photo=FILE` binds the file to that reference: `jud` reads it,
holds it to the wire's limits (PNG or JPEG, each side 280 to 1568 pixels,
upright), base64-encodes it and sends it. A cases document declares each
case's image beside its state, so `jud record`, `eval` and `tune` take no
flag.

`transformers serve` does not take the hosted API's `assets` map; it takes
a top-level `images` array of `data:` URLs, at most four, and refuses any
other field in the body. `shared_images: data-urls` in the configuration
file sends that shape. Without it, the request is refused with a 422 before
the model sees anything, which is the right failure: Ollama in the same
situation [drops the map without a word](https://github.com/chussenot/judgment/blob/main/docs/project/research/compatible-servers-and-models.md#seen-on-2026-10-17-ollama-0405-images-and-clef-flash)
and judges the text alone.

On the server side each question becomes one chat turn, the image first
and then the rendered question, through the model's own chat template;
the questions of one request go through the model in one batched forward
pass. The server needs a vision-language model: a text-only model answers
a request with images with a 400.

## Why no decision.json

oncall-judge needed a `--decision-config` because its model was fine-tuned
to write JSON after one exact system prompt and answered near chance any
other way. This model is a general instruction-following VLM: the server's
default prompt (`Yes: ...` / `No: ...`, `A. ...` to `D. ...`) is what it is
trained on, and `Yes`, `No` and the letters are single tokens in its
tokenizer, which the server checks on every request. A custom chat template
would also have to reproduce the processor's image placeholder tokens,
which is where a hand-written template for a VLM goes wrong. So the server
runs with its defaults, and the rubric's option order is free to change.

## Test, record, tune

```sh
jud check cat.jud cat-cases.jud recordings/*.jud                 # read every document as the crate does
jud eval --replay recordings cat.jud cat-cases.jud               # grade the recordings, no server
jud eval --replay recordings --against recordings-smolvlm cat.jud cat-cases.jud   # the two models side by side
jud record --out recordings --refresh cat.jud cat-cases.jud      # ask the server again
jud tune --replay recordings cat.jud cat-cases.jud               # propose the gates
```

`jud eval` on the committed recordings:

```text
cat_present
  labelled 26, correct 26, accuracy 1.00 (95% interval 0.87 to 1.00)
  brier 0.021, calibration error 0.097, confidence when right 0.90, when wrong -
  gate: acts on 26 of 26, defers 0, accuracy when acted 1.00

subject
  labelled 26, correct 22, accuracy 0.85 (95% interval 0.66 to 0.94)
  brier 0.287, calibration error 0.211, confidence when right 0.68, when wrong 0.57
  gate: acts on 6 of 26, defers 20, accuracy when acted 1.00
```

The bars in `cat.jud` are what `jud tune` read off 26 cases, which it
rightly calls a guess with a number on it; add cases and re-tune before
trusting them. The `cat_present` threshold of 0.5 is a policy choice: tune
proposed 0.15 by its tie rule, and 0.5 sits in the middle of the gap the
model leaves between 0.13 and 0.82.

## Picking the model

Two small VLMs that run on a CPU were recorded on the same 26 cases, from
the same rubric, through the same server, and graded with `jud eval`:

| | `Qwen/Qwen2-VL-2B-Instruct` | `HuggingFaceTB/SmolVLM-500M-Instruct` |
|---|---|---|
| `cat_present` accuracy | 26/26 | 26/26 |
| `cat_present` Brier, calibration error | 0.021, 0.097 | 0.041, 0.116 |
| `cat_present` on the dogs (yes-probability) | 0.06 to 0.09 | up to 0.41 |
| `subject` accuracy | 22/26 | 21/26 |
| input tokens per photo | about 680 | about 1,900 |
| median time per photo, 4 CPU cores | 6.7 s | 14.4 s |
| licence | Apache 2.0 | Apache 2.0 |

Both get the verdict right on every case. Qwen2-VL is the better judge
because of the margin: a pug at 0.41 is one hard negative away from a
wrong `yes`, a pug at 0.09 is not. It is also twice as fast, because
SmolVLM splits a photo into sub-images and spends three times the tokens.

`subject` is weak on both. Qwen2-VL calls three cats (two sphynxes and a
birman) `other_animal` while `cat_present` still says yes on them at 0.84
to 0.91; SmolVLM calls a church, a gas pump and a parachute `other_animal`.
The Noul with explicit `true` and `false` criteria is the robust question;
a four-way choice over categories a 2B model half-knows is not, which is
why its gate defers most answers and the README calls it a diagnostic.

Other candidates were ruled out before a run: `LiquidAI/LFM2-VL-450M`
(a licence of its own, not Apache), the Clef and decider families (27B and
up, or no vision), and a plain image classifier fine-tuned on cats, which
does not speak the System One wire and would need an adapter server in
front of it. The wire is the point of this repository.

## Caveats

- `shared_images: data-urls` is required and nothing checks it for you:
  without it `transformers serve` answers a 422 (`extra fields not
  permitted`), and `jud` prints it as an invalid request.
- The server takes at most four images per request and only a
  vision-language model; a text model answers 400 on any request with an
  image.
- The recordings name `http://localhost:8002`: the two candidates were
  served side by side on 8001 and 8002 while they were recorded. The
  fingerprint a replay is found by does not include the server, so
  `mise run serve` on 8000 and the committed recordings agree.
- The PyPI `torch` wheel is the CUDA build (several GB) even on a machine
  without an NVIDIA GPU; it runs on the CPU all the same.
- Image tokens are the cost: a 426 by 320 photo is about 680 input tokens
  on this model. Send the smallest image that keeps the subject legible.
- The images are from two published datasets under their own licences
  (`images/SOURCES.md`); they are test data, not part of any licence this
  repository grants.
