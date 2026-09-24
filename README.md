# AI301 Unit 2 starter: repro-check

Materials for Unit 2 of AI301 (claim and reproduce). This repo holds
the week's runnable artifacts: the repro-check skill and its eval
harness. All instructions live on the course portal (Overview,
Activity, and Assignment tabs for Unit 2). This repo is the package
those pages tell you to install and run.

## What's here

- `skill/`: the repro-check skill for Claude Code. It's complete
  except for three files that ship as templates: `skill/rubric.md`,
  `skill/references/evidence-guide.md`, and `skill/voice-guide.md`.
  Filling them is the Unit 2 deliverable.
- `eval/`: the eval harness, the gold labels, and 24 frozen packages
  (20 scored, plus the 4 from the in-class activity). See
  `eval/README.md` for the full run and output guide.

## Install the skill

Clone this repo, then copy `skill/` into Claude Code's skills folder:

    git clone https://github.com/codepath/ai301-unit2-starter.git
    mkdir -p ~/.claude/skills/repro-check
    cp -R ai301-unit2-starter/skill/* ~/.claude/skills/repro-check

Edit `rubric.md`, `references/evidence-guide.md`, and `voice-guide.md`
inside the installed copy. The eval and live runs below both read from
that copy, so they always grade with the same files.

## Run the skill

The skill runs in two modes. Both need the Claude Code CLI (`claude`)
installed and signed in.

### Eval mode: grade the 20 practice packages

Run from this repo's `eval/` folder:

    cd ai301-unit2-starter/eval
    python3 run_eval.py \
        --rubric ~/.claude/skills/repro-check/rubric.md \
        --evidence ~/.claude/skills/repro-check/references/evidence-guide.md

Add these flags as you need them:

| Flag | What it does | When to use it |
|---|---|---|
| `--limit 3` | Grades only the first 3 | Checking that your setup works |
| `--only pkg-07,pkg-12` | Grades only the ones you name | Re-checking the ones you got wrong, about $0.20 each instead of about $4 for a full run |
| `--include-calibration` | Also grades the 4 from the class activity (never scored) | A free extra check, and it works with `--only` |
| `--save-run eval-run.txt` | Writes the run to a file | Your final full run. This is the file you submit. `--limit` and `--only` runs refuse to write it. |
| `--out results.json` | Saves every check's result as JSON | Digging into why one failed |
| `--workers N` | Grades N at once (default 5) | Rarely needed |

`python3 run_eval.py --help` lists every flag. `eval/README.md`
explains the output table and the passing bar.

### Live mode: check a comment before you post it

Live mode grades a comment you wrote for your real issue before it
goes up on GitHub. You'll use it twice this unit:

1. **On your claim comment**, before you post it.
2. **On your repro comment**, before you post it. The skill grades it
   together with your claim comment, since the two are read as a pair.

Save each draft as a file first (for example `claim.md` and
`repro.md`). Then, from the folder that holds them, run the skill in
either of these ways.

**Inside a Claude Code session.** Start `claude` in that folder, then
type the skill's name as a slash command:

    /repro-check grade my claim comment in claim.md for issue <URL>

You can also ask in plain words, such as "use repro-check to grade my
claim comment in claim.md for issue <URL>". A session is the easier
option when you're revising, because you can ask follow-up questions
about a failed check and re-run after each edit.

**As one command from your terminal.** Put the same request in quotes
after `claude`:

    claude "repro-check: grade my claim comment in claim.md for issue <URL>"

For the repro comment, name both files in the same request:

    /repro-check grade my claim comment in claim.md and repro comment in repro.md for issue <URL>

**Reading the result.** The last thing the skill prints is a block
like this:

```json
{
  "item": "https://github.com/owner/repo/issues/123",
  "checks": [
    {"name": "names-the-issue", "grade": "pass",
     "evidence": "quotes the error message from the issue body"},
    {"name": "steps-rerunnable", "grade": "fail",
     "evidence": "step 2 says 'set up the project' with no commands"}
  ],
  "verdict": "reject"
}
```

- `verdict` is the answer. `accept` means post it, and `reject` means
  revise it first.
- `checks` shows why. There's one entry per check in your rubric, so
  the names match whatever you called your checks. The `evidence` line
  is what decided each check, so read the failed ones first.

When you grade only your claim comment, the checks about the repro
comment show `"grade": "unclear"` and
`"evidence": "not yet applicable: claim-only draft"`. That's normal,
and those checks don't count toward the verdict.

If the output doesn't end with this block, the skill never ran and
Claude answered on its own. Check that the skill is installed at
`~/.claude/skills/repro-check/`, then try again.
