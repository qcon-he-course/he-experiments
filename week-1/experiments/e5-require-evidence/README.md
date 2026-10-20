# Experiment 5: require evidence, then keep what you verified

How do you know an answer is true, and what does writing down verified facts add? This rung asks the agent to back every claim with evidence and to label what it checked. It also gives the agent a short file of facts that someone verified by hand, as a team would commit it.

## What changes

Rung 4, plus two things:

- **The evidence rule** (`layer/rules.md`). Back every claim with a file and line range, or the result of a tool call. Label each answer VERIFIED, INFERRED or UNKNOWN. Never present an inferred claim as fact.
- **`AGENTS.md`**, six verified lines about PetClinic. Every e5 and e6 run commits it into the clone, and Claude Code loads it because the repository has no `CLAUDE.md` of its own. Ours came from an `/init` draft (`init-draft.md`) and an audit of every claim in it (`audit.md`).

Two things change at once, so a gain in right answers can't be credited to one of them. Read the citations and labels for the rule, and question 3 for the file.

## Run it

From `experiments/ladder/`. Set up first: [SETUP.md](../../SETUP.md).

**1. The `/init` trap.** Read `init-draft.md`: 57 confident lines, written in about a minute. Then read the tally at the end of `audit.md`: 30 claims, 6 kept. Line 20 is wrong. Now ask a bare agent to check it:

```bash
./run.sh e0
```

Trust the folder, then ask: "This line is from a generated CLAUDE.md: 'VetRepository results are cached in the JCache vets cache (system/CacheConfiguration, Caffeine provider)'. Is it true? Give the file and line that shows it." Type `/exit` when it answers.

**2. Your own audit (optional).** Draft a context file with `/init` and audit it yourself:

```bash
./run.sh init        # trust the folder, type /init, approve the write of CLAUDE.md, then /exit
```

The draft lands in `../e5-require-evidence/runs/<date-time>-<model>-init/files/CLAUDE.md`. Tag each line verified, inferred, wrong or copied. Keep a line only if it passes three tests: it is verified, the agent couldn't find it in seconds, and the README doesn't already say it. Save what you kept as `../e5-require-evidence/AGENTS.md`, in place of ours. `git checkout -- AGENTS.md` in this folder brings ours back.

**3. The e5 run.**

1. `./run.sh e5`, and trust the folder. Claude Code says it loaded `/work/AGENTS.md`.
2. Type `/context all` at the very start of the line. You should see two memory files: `/etc/claude-code/CLAUDE.md`, with the rules of e2 to e5, and `AGENTS.md`. A line that starts with a space goes to the agent as a question.
3. Paste the questions (`pbcopy < ../questions.txt` on macOS).
4. When the answer is in, ask: "Which of your answers did you not verify, and what would verify them?"
5. `/exit`, then `./show-run.sh e5 --answers` to read the labels.

Approve read-only commands. Deny anything that writes, builds or reaches the network.

**Unattended.** `./batch.sh claude-opus-5-5 1 e5` runs it once with `claude -p`, in Claude Code's `dontAsk` mode: a call that would ask for permission is refused.

**Score it.** Mark the answers against [the answer key](../../reference/answer-key.md) and write `runs/<run>/score.txt` with one line, `q1-5=N q6=M`. `./scoreboard.sh` then includes it.

## What we saw

Five unattended runs on Haiku 4.5 and five on Opus 5.5, on 19 October, in `dontAsk` mode. A model (Claude Fable 5.1) marked each run against the answer key, and we confirmed the marks by hand. Five runs per model is a small sample, so read the counts.

| Model | Q1-5 right (median) | Q6 steps (median) | Minutes | Tokens sent | Cached | Tool calls | Files read | MCP calls |
|---|---|---|---|---|---|---|---|---|
| Haiku 4.5 | 4 | 5 | 1.1 | 382k | 93% | 21 | 11 | 2 |
| Opus 5.5 | 5 | 6 | 1.1 | 220k | 86% | 12 | 13 | 0 |

| Question | Haiku 4.5 | Opus 5.5 |
|---|---|---|
| 1. Getting started | 4 right, 1 right but unchecked | 4 right, 1 right but unchecked |
| 2. Database | 2 right, 1 right but unchecked, 2 wrong | 4 right, 1 right but unchecked |
| 3. Vet cache lifetime | 5 wrong: right lifetime, wrong reason | 5 right |
| 4. A second "max" | 5 right | 5 right |
| 5. Search for "davis" | 4 right, 1 right but unchecked | 4 right, 1 right but unchecked |

- **Opus got question 3 right in every run, and `AGENTS.md` is why.** The file states the deciding reason, and Opus gave it each time, after four rungs of the wrong reason. That is the mechanism: a checked fact reaches every later session. It is also the caveat: the file carries the answer, so the rule gets no credit for it.
- **Haiku didn't take the fact from the file.** All five Haiku runs gave `CacheConfiguration` as the reason for question 3, though `AGENTS.md` says that class never applies. One run labelled that answer VERIFIED.
- **Citations, as the rule asks.** Opus cited a file and line in all five questions in every run, against a median of 2 at rung 4. Haiku cited in a median of 4, against 3.
- **The labels split by model.** Opus labelled claim by claim and marked a median of 2 answers INFERRED or mixed. Haiku labelled every answer VERIFIED.
- **Cheaper than rung 4.** Opus made 12 tool calls against 28, and sent 220k tokens against 320k. With answers in `AGENTS.md`, both models searched less (inferred).
- **The `/init` trap.** We asked step 1's question of Opus 5.5 at rung 0. The same model that wrote the wrong line in `/init` called it "only partly true" when asked to check it. It found the missing JCache provider and labelled its runtime conclusion inferred. Verification is a step someone has to ask for. The evidence rule asks for it on every answer.

More in [findings](../../results/findings.md) and the [summary](../../results/summary.md).

## Files

| File | What it is |
|---|---|
| `layer/rules.md` | The evidence rule, added to `/etc/claude-code/CLAUDE.md` |
| `init-draft.md` | Our `/init` draft: Opus 5.5 at rung 0, 18 October |
| `audit.md` | Every claim in the draft, tagged, with the three tests and the evidence |
| `AGENTS.md` | The verified lines, committed into each e5 and e6 run's clone |
| `runs/` | One folder per run, named `<date-time>-<model>`, and `-init` for `/init` runs; git ignores it |
