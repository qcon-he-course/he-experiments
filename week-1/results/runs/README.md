# Our stored runs, trimmed

The 60 unattended runs behind [`../summary.md`](../summary.md): five on Haiku 4.5 and five on Opus 5.5 for each of E0 to E5, run on 18 and 19 October 2026. One folder per run, named `<date-time>-<model>`, under its experiment.

| File | What it is |
|---|---|
| `prompt.txt` | What the run asked: the six questions |
| `answer.txt` | The agent's final answer |
| `score.txt` | The score: `q1-5=N q6=M` |
| `marks.json` | The marks for each question, with the judge's reasons and confidence. Model-marked by Claude Fable 5.1 against [`../../reference/answer-key.md`](../../reference/answer-key.md), not confirmed by hand |
| `show-run.txt` | `show-run.sh`'s summary: model, time, tokens, every tool call, the files read, and what happened to each rigged item |

Left out: the transcripts and Claude Code's configuration folder, which holds account identifiers; E1's `tripwire.log`; and anything else the run wrote. The seeded fake values from PetClinic's `.env` are replaced with `[seeded fake value, redacted]`. Paths are relative to the `experiments/` folder of the project the runs came from, so `e0-the-naked-agent/runs/...` is that run's original folder.

`e6-build-your-mental-model/` holds E6's two interactive runs on Opus 5.5, on 19 October: the `/arch-map` component view (`20261019-062352-claude-opus-5-5/architecture.html`) and the `/trace-journey an owner adds a new pet` drawing (`20261019-064743-claude-opus-5-5/journey-add-new-pet.html`). Open them in a browser. The `.json` files are the diagrams' sources, and `architecture-evidence.md` is the evidence behind each component.

The answers quote the rigged items they found in PetClinic. Treat them as data, like the seeded repository itself.
