# tech-viz

Interactive D3 visualisations of technical concepts that are easier to see than to read.

**Live:** https://mstoepel.github.io/tech-viz/

## Visualisations

| # | Topic | Page | What it teaches |
|---|-------|------|-----------------|
| 01 | Git | [git/three-trees.html](git/three-trees.html) | Working directory, index, HEAD, and why `add` / `restore` / `reset` are all the same move |
| 02 | Git | [git/branching.html](git/branching.html) | Branches as pointers, fast-forward vs merge commit, rebase, detached HEAD |
| 03 | Git | [git/remotes.html](git/remotes.html) | Local vs origin, why `origin/main` is stale until fetch, rejected push, pull as fetch+merge or rebase, force push |
| 01 | Docker | [docker/layers.html](docker/layers.html) | Content-addressed layers, cache invalidation, Dockerfile ordering, copy-on-write, prune |
| 02 | Docker | [docker/networking.html](docker/networking.html) | Network namespaces, the default vs user-defined bridge, published ports as DNAT through the host, embedded DNS, bind addresses, network isolation, host mode; every connection attempt traced to where it fails |
| 01 | Kubernetes | [k8s/reconciliation.html](k8s/reconciliation.html) | kubectl writes desired state; controllers run per tick and close the gap: self-healing, scaling, node loss, rolling update, undo, garbage collection |
| 01 | Databases | [db/transactions.html](db/transactions.html) | Two interleaved transactions; dirty, non-repeatable and phantom reads, row locks, and a serialization failure, one isolation level at a time |
| 01 | AI systems | [ai/agent-loop.html](ai/agent-loop.html) | The agent loop: the model as a stateless function over the whole context, tool calls and results as text, retries, runtime stop conditions, memory as a tool, compaction |
| 01 | Observability | [otel/traces.html](otel/traces.html) | OpenTelemetry traces: spans linked by a traceparent header; dropped headers, uninstrumented services, latency attribution, error status, head-based sampling, log correlation |
| 02 | AI systems | [ai/transformer.html](ai/transformer.html) | Attention as a weighted lookup: scores, causal mask, softmax and temperature, head types (sink, duplicate, induction, pronoun), the residual stream, and the KV cache |
| 03 | AI systems | [ai/diffusion.html](ai/diffusion.html) | Diffusion on a 2D point cloud: fixed forward noising on a cosine schedule, the exact Bayes denoiser as the model, DDIM sampling, step count, classifier-free guidance, memorisation vs generalisation |
| 04 | AI systems | [ai/post-training.html](ai/post-training.html) | Post-training on a categorical policy over seven answers: SFT cross-entropy, Bradley-Terry reward model from biased labeler comparisons, RLHF policy gradient with a KL leash, reward hacking, the closed-form optimum, DPO |
| 01 | Spark | [spark/lazy-plans.html](spark/lazy-plans.html) | Lazy evaluation, the logical plan, a four-rule optimizer (filter merge, predicate pushdown, column pruning, projection collapse), exchanges, stages, tasks, shuffle partitions, broadcast joins |
| 02 | Spark | [spark/shuffle.html](spark/shuffle.html) | A tick-driven scheduler: driver, executors and cores, tasks per partition, the timeline, the 200-partition default, adaptive coalescing, broadcast vs sort-merge, skew and the straggler, AQE skew split, caching |

Candidates for the next round: DNS resolution, TLS handshake, B-tree indexes, consistent
hashing, git object store.

The transformer page is the first numeric-simulation sandbox: the terminal is still there for
the lesson, but the controls are also clickable and the state is a tiny forward pass rather
than a command log. Post-training and diffusion should follow that shape.

## Conventions

- **One folder per topic, one self-contained HTML file per concept.** No build step,
  no bundler. D3 and fonts load from CDN; everything else is inline. Open the file in a
  browser and it works.
- **The index is a manifest.** `index.html` renders from two arrays, `TOPICS` and `VIZ`.
  Adding a visualisation is one entry in `VIZ`; adding a topic is one entry in `TOPICS`.
  Entries without a `path` (or with `status: "planned"`) render as dimmed placeholders.
- **Sandbox, not slideshow.** Each page has a terminal where the learner types real
  commands. A guided lesson sits on top, but the whole command registry is always open.
- **Engine shape.** Every sandbox is four separable parts: an immutable `state`, a registry
  of pure `(state, args) -> {state, out}` reducers, a parser, and a D3 renderer that only ever
  diffs state against the DOM. A lesson is data (`LESSON` array), not code.
- **Semantic colour.** Pick two or three colours that mean something in the model (in the
  git sandbox: amber = unstaged, green = staged, purple = refs) and keep everything else quiet.

## Running locally

Open any HTML file directly, or serve the root so the index links work:

```bash
python -m http.server 8000
```

## Deploying

GitHub Pages, deployed from the `main` branch root. No workflow needed because there is
nothing to build.
