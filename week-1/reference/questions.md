# The six questions

Every run of every experiment asks these six questions about the seeded PetClinic. In an unattended run, `run.sh` asks them for you. In an interactive run, paste them yourself from `../experiments/questions.txt`.

Use this sheet to record what the agent said and how sure you were, before you check the answers in [`answer-key.md`](answer-key.md).

## Start a bare agent

From `experiments/ladder/`:

```bash
./run.sh e0
```

"Bare" means Claude Code as it comes, inside the container: no context file, no skills, no extra tools. Each later rung adds one thing.

## Ask these six questions

Paste them as one message, exactly as written. Don't add hints and don't correct the agent.

```text
Answer these questions about this codebase:

1. What is this project about? How do I get started?
2. When you start PetClinic with no options, which database does it use, and what decides that?
3. How long does PetClinic keep the vet list in its cache?
4. Jean Coleman already has a pet called Max. Can she add another pet called "max"?
5. You search owners for "davis", all lower case. What do you see?
6. Trace what happens when an owner books a visit for a pet, from submitting the form to the row in the database. List each step in order, with the class and method involved.
```

## Record your answers

Fill in the first three columns before the answers are shown. Confidence is how sure you are that the agent's answer is right, from 1 (a guess) to 5 (certain).

### Before: the bare agent

| Question | Agent's answer, in a few words | Your confidence, 1 to 5 | Right? |
|---|---|---|---|
| 1. What it is, how to start it | | | |
| 2. Database | | | |
| 3. Vet cache, how long | | | |
| 4. A second pet called "max" | | | |
| 5. Searching for "davis" | | | |
| 6. Booking a visit (steps right, of 6) | | | |

### After: a later rung

Same questions, same wording, same model. Note which rung.

| Question | Agent's answer, in a few words | Your confidence, 1 to 5 | Right? |
|---|---|---|---|
| 1. What it is, how to start it | | | |
| 2. Database | | | |
| 3. Vet cache, how long | | | |
| 4. A second pet called "max" | | | |
| 5. Searching for "davis" | | | |
| 6. Booking a visit (steps right, of 6) | | | |

## Afterwards

Count how many answers fell in each box, for each run:

| | Agent right | Agent wrong |
|---|---|---|
| You were sure (4 or 5) | | |
| You were unsure (1 to 3) | | |

The box that costs most in real work is "sure, and the agent was wrong".
