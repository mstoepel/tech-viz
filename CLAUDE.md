# CLAUDE.md — working notes for tech-viz

Interactive, self-contained HTML sandboxes that teach technical concepts by letting the
reader type commands and watch a model move. Served by GitHub Pages from `main` at the
repo root: https://mstoepel.github.io/tech-viz/. Nineteen pages so far across git, Docker,
Kubernetes, databases, observability, AI systems and Spark.

## Repo shape

- One folder per topic, one HTML file per concept. No build step; D3 and fonts come from
  CDN, everything else is inline. Open any file directly and it works.
- `index.html` renders from two arrays near the bottom: `TOPICS` (section order and
  titles) and `VIZ` (one entry per page: topic, path, title, blurb; `status: "planned"`
  or no `path` renders a dimmed placeholder). Sandboxes are numbered within their topic
  in array order, so a new page's manifest entry must come after its predecessors, and
  the page's own eyebrow ("Sandbox 03 · Git internals") must agree with that position.
- `README.md` has the same table plus the conventions and the candidate list.
- Commit messages end with `Co-Authored-By: Claude Fable 5.1 <noreply@anthropic.com>`.
  The user pushes; do not push. Commit when asked, or when the user has said to.

## The engine pattern (every page)

Every sandbox is the same five parts, in this order in the file:

1. **State**: plain data only. `structuredClone` is used everywhere, so a function or a
   RegExp inside state breaks cloning silently (the transactions page hit this by storing
   a predicate; store the text and re-parse).
2. **Reducers**: `REGISTRY[name] = (state, args, raw) => { state?, out: [{text, cls}], error? }`.
   Pure. Never touch the DOM. Errors set `error: true`; auto-advance ignores red-coloured
   informational lines but respects the flag.
3. **Parser**: longest-prefix lookup over the registry (two-word keys like `git add` win
   over one-word keys).
4. **Renderer**: a pure function of state, D3 joins keyed by identity, flashes for what
   changed since the last render. Clickable controls issue the same commands as the
   terminal (`run("layer 2")`) so lessons and replay work unchanged.
5. **Lesson**: an array of `{ title, body (html), try, expect, readonly?, needs? }`.
   - `expect` is a regex the typed command must match to auto-advance.
   - Auto-advance also requires the command to have **changed state**, unless
     `readonly: true`. A Try command that leaves state unchanged stalls the lesson; this
     has bitten four times (a token already selected, a head already selected, a β already
     at the value, a strategy already set). Start state so the first Try changes something.
   - `needs(state)` says whether the step can be demonstrated. If false, `renderLesson`
     restarts and replays every earlier step's `try` quietly. The lesson is therefore its
     own fixture, and every Try must be deterministic (fixed messages, seeded hashes).
   Numeric pages (transformer, diffusion, post-training) keep the same shell with sliders
   and buttons that call `run`.

Design conventions: dark palette, two or three semantic colours that mean something in
the model (amber = unstaged/new/gap, green = staged/cached/pass, purple = refs/model,
red = errors/walls), a "signature" visual that shows the mechanism, and a footer
**Engine notes** paragraph that says honestly what is simulated versus real (hand-written
attention scores, a scripted agent policy, a keyword-matching "model", illustrative
timings). Keep that honesty; it is part of the product.

## Verification workflow that works

Screenshots from the preview pane are flaky and D3 transitions freeze while the pane is
hidden, so verify through the DOM, not by eye:

1. `navigate` to the file, wait 1s, then run JS that loops the lesson: for each step call
   `run(LESSON[step].try)`, wait ~650ms for the auto-advance timer, record a snapshot of
   the state (counts, verdicts, tokens), and stop if `step` didn't move. Read
   `read_console_messages(onlyErrors)` afterwards.
2. Exercise free-play paths the lesson doesn't cover, and the error messages.
3. Jump-ahead replay: set `step = N; renderLesson()` and check the replayed state.
4. Click a control or two to confirm they issue commands.
5. `resize_window` mobile, read `gridTemplateColumns` and `scrollWidth > innerWidth`.
6. One screenshot if it comes; retry once, then move on.
7. Load `index.html` and list the cards for the topic.

If the page throws before `let step` exists, the JS harness fails with "step is not
defined". Read the console; a syntax error has no line number in the pane, so bisect it
in the browser: pull the script text from `document.scripts`, split on the section
comment markers, `new Function(section)` each, then split the failing section by blank
lines or single lines and parse each. Found a missing `)`, a ternary without its `:`
branch, and a parenthesis inside a template literal that way.

Editing a file reloads the preview pane and invalidates tab ids, so don't reuse a tab id
across an edit; navigate fresh. `navigate` to the same URL is a no-op; close the tab or
use `location.reload()`.

## Recurring bugs, so they aren't rediscovered

- Signed bit shifts on 32-bit hashes: `h >> k` goes negative and indexes past an array or
  prints negative numbers. Always `>>>`. Hit three times.
- Lesson step whose Try doesn't change state (see above).
- D3 `.attr("class", …)` that drops a marker class the next render selects by. Select by
  position (`td:nth-child(4)`) or keep the marker class in the new value.
- `d3.select(...).html(empty)` followed by an enter/update join that never removes the
  empty placeholder; remove `.empty` explicitly in the non-empty branch.
- Tree printers: a cheap `+-`/`:-` printer garbles branches; use a proper prefix-carrying
  recursion.
- Toy dynamics that don't show the claimed effect at the lesson's settings. Measure the
  numbers the lesson quotes and fix either the model or the text: the diffusion smoothing
  had to become an additive floor, the post-training update had to use the exponentiated
  (mirror-descent) policy gradient so low-probability answers can move, the Spark straggler
  step needed a sane partition count first, and the disclosure page needed two real
  mechanisms (docs quoting other skills, lost-in-the-middle attention) before eager
  loading degraded.
- A Try command that is a no-op in the state the previous step left behind — `step` after
  a simulation already finished, a `q N` when that question is already selected. Where it
  makes sense, have the command restart rather than refuse (inference-serving's `step`
  reruns a finished sim).
- **Verify the direction of a textbook effect in your toy before writing it down.** Three
  claims failed this session: retrieval answer spans straddled chunk-2 boundaries so
  "bigger chunks fix splits" was false and reranking looked broken; a paged KV allocator
  without an admission watermark thrashed and lost to naive reservation, the opposite of
  reality; and end-to-end latency *falls* with batch size for a burst workload, so the
  throughput/latency tradeoff had to be shown as time-per-output-token instead. Toy
  parameters (document length, workload size, budget) usually decide whether the real
  effect is even visible — a corpus of five-sentence documents cannot demonstrate that
  chunking matters.

## What exists

| Topic | Pages |
|---|---|
| git | three-trees, branching, remotes |
| docker | layers, networking |
| k8s | reconciliation |
| db | transactions |
| otel | traces |
| ai | agent-loop, transformer, diffusion, post-training, coding-agent, progressive-disclosure, tokens-sampling, retrieval, inference-serving |
| spark | lazy-plans, shuffle |

Candidates discussed and not built: DNS resolution, TLS handshake, B-tree indexes,
consistent hashing, git object store, mixture of experts, multi-agent handoffs, evals with
graders.

## Building a new page: checklist

1. Pick the one idea the page exists to teach and the signature visual that shows the
   mechanism moving. Write that sentence into the h1/standfirst first.
2. Copy the shell from the closest existing page (same topic if possible) and keep the
   section order: state, reducers, parser, renderer, lesson, wiring.
3. Make every Try command change state; make the first step's initial state such that it
   does. Give the lesson `needs` predicates so replay reconstructs any step.
4. Write the Engine notes footer honestly.
5. Add the `VIZ` entry after the topic's last page and a README table row; if it is a new
   topic, add it to `TOPICS`.
6. Run the verification workflow above; measure every number the lesson quotes.
7. Commit with a message that states the mechanism and what the lesson covers.
