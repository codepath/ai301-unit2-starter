# Eval set and harness

20 scored reproduction packages, 4 unscored calibration packages from
the class activity, instructor gold labels, and a runnable harness that
grades any rubric-plus-evidence-guide pair against the set.

## The one command

From this directory, with the Claude Code CLI installed (you have it if
you have been building skills):

    python3 run_eval.py --rubric path/to/your-rubric.md \
        --evidence path/to/your-evidence-guide.md

Point `--rubric` at your filled rubric and `--evidence` at your filled
evidence guide. The shipped templates (`../skill/rubric.md`,
`../skill/references/evidence-guide.md`) are empty on purpose, and the
skill refuses to grade without them, so write your checks, your verdict
rule, and your evidence map first. Your voice guide is never part of an
eval run: eval mode ignores it by design.

Useful flags: `--limit 3` for a quick smoke run, `--only
pkg-07,pkg-12` to re-grade just the named packages, `--workers N` to
change parallelism (default 5), `--include-calibration` to also grade
the four calibration packages from the in-class activity (they are
never scored), `--out results.json` to keep the full per-check results.

`--only` is the flag for the revise loop: when a full run disagrees on
two packages, re-run only those two while you adjust your components
(about $0.20 per package instead of about $4 for a full run), then
confirm with one full run at the end. Partial runs never print a bar
verdict; only a full 20-package run can pass.

Partial runs also cannot show the category floor, and a revision that
loosens a check can flip a package that agreed before. So when a
revision loosens a check, add canaries to the `--only` list: one
already-agreeing package from each single-package category the change
could touch (the `disclosure` category is the live case), so a flip
shows up at $0.20 instead of on your confirming full run.
`--include-calibration` composes with `--only`, so a calibration
package your class already resolved works as a free trap check too.

Every run grades with Sonnet 5.5 (`claude-sonnet-5-5`), the course's
standard model; there is no model flag. This keeps every student's run
(and the instructor stability runs that set the bar) on the same grader,
and Sonnet is the model your course credit is budgeted for. The `model:`
line at the top of `skill/SKILL.md` names the same model, so when you
run the skill yourself in Claude Code it grades on Sonnet 5.5 too,
whatever model your session uses. Leave that line in.

## Saving the run you commit

Add `--save-run eval-run.txt` to your confirming full run and the harness
writes the file for you:

    python3 run_eval.py --rubric path/to/your-rubric.md \
        --evidence path/to/your-evidence-guide.md \
        --save-run eval-run.txt

The file holds the same text you watched on screen, written as UTF-8 on
every platform, under a short header the harness fills in: the pinned
model, the tool it graded, and a fingerprint of each file that went into
the run. That file is what you upload to your course repo.

Partial runs refuse to write it. A `--limit` or `--only` run says so and
leaves the file untouched, so a cheap re-run can never overwrite the full
run you already saved.

Do not hand-edit the file. Where you account for the runs it took to get
there is your write-up in the phase file, not the transcript.

## What the output means

One line per package while grading, then a table:

    item    gold    verdict  agree  note
    pkg-01  accept  accept   yes
    pkg-07  reject  accept   NO     graded accept
    ...
    categories: clear-accept 8/8  disclosure 0/1  no-evidence 4/4  unfollowable-comms 3/3  wrong-target 4/4
    agreement: 19/20 scored items  (bar: 18/20: below the bar; category floor unmet: no match in disclosure)

- `gold` is the instructor label from `gold-labels.json`.
- `verdict` is what the skill decided with YOUR rubric and evidence
  guide.
- The `note` column names the checks your rubric failed the package
  on, which is where to look when you disagree with a gold label.
- The `categories` line tallies matches per eval-set composition
  category. Passing needs at least one match in every category (the
  category floor): a rubric that cannot see a whole category, however
  well it does elsewhere, is missing a check the set was built to
  force. The 1-package `disclosure` category is the live case. Partial
  runs print the tallies for just the packages graded, plus a reminder
  that only a full run decides the bar and the floor.
- The agreement line is the score the grading bar reads. The bar:
  agreement of 18 of 20 or better passes (exactly 18 passes), AND the
  category floor holds. The 4 calibration packages are never scored.
  Full bar details, including the human read of your components: the
  Unit 2 Assignment tab on the course portal.

Disagreements are the feedback loop: open the package the table names,
reread the artifacts against the issue, and decide whether your check's
pass condition or your evidence map is what needs to move. Then re-run;
retries are unlimited.

## What is in a package

Each `packages/*.md` file is self-contained: a real issue's context
(title, body excerpt, thread highlights, and a repo-facts block with
the repo's stated bug-report template asks and contribution policy,
including any AI-use policy), plus a candidate claim comment and a
candidate repro report, all frozen on the capture date stamped at the
top. The issue contexts are real; every candidate comment and report is
instructor-authored, so no real stranger's writing is ever graded here.
The harness never touches GitHub; the skill grades the bundle text, so
every run sees the same world and your score cannot drift because a
thread moved on.

Every bundle also has a `.json` twin with the same frozen content in
structured form (issue, thread highlights, repo facts, claim comment,
repro report as fields). Read whichever you prefer; they are the same
snapshot. `packages_to_json.py` regenerates the JSON from the markdown.

## Do not grade the live issue

The `source:` line in each package names the real issue
(`owner/repo#number`). It is deliberately not a link: the real issue
has kept moving since the capture date, with new comments, new linked
PRs, sometimes a fix already merged. That is exactly why the snapshots
exist. The gold labels describe the snapshot, not today's GitHub. When
the live page and the bundle disagree, the bundle wins.

## Format note

This markdown-plus-manifest layout is the browsable form of the eval-set
format used in production model evals: one record per item with input,
gold label, and metadata (usually JSONL), a judge prompt (here, the
skill plus your rubric and evidence guide), and a scoring script (here,
`run_eval.py`). If you later meet an eval framework at work, you
already know the parts.
