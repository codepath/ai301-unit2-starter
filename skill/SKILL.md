---
name: repro-check
description: Grade a reproduction package (a claim comment plus a repro report against its issue) and decide whether it is ready to post upstream. Use when checking a draft claim or repro comment before posting, or when grading an eval package bundle.
model: claude-sonnet-5-5
---

# repro-check: rubric-driven reproduction grading

You are grading one reproduction package to answer a single question:
is this ready to post? A package is a candidate claim comment and a
candidate repro report, read against the issue they belong to. You do
not answer from gut feel. You answer by executing the rubric in
`rubric.md`, check by check, against evidence you gather from the
package itself.

## Inputs

One of:

- **Live mode**: the student's own draft(s) plus their issue's URL.
  Gather the issue-side evidence live (via `gh`, the GitHub API, or the
  web; `references/evidence-guide.md` says where each signal lives),
  and read the draft(s) as the candidate package. Live mode accepts two
  package states:
  - **Claim-only draft** (early in the week, before the claim is
    posted): grade only the checks whose evidence is the claim comment
    or the repo's conventions. Report every check that needs the repro
    report with grade `unclear` and evidence `not yet applicable:
    claim-only draft`, and leave those checks out of the verdict rule.
    The verdict then answers only: is this claim comment ready to post?
  - **Full package** (before the repro comment goes out): grade every
    check; the verdict answers: is this reproduction ready to post?
    Read the drafts the way a stranger on the issue thread will read
    the posted comments: the package is what the drafts contain and
    quote, not other files in the student's working directory.
- **Eval mode**: a package bundle (a markdown file containing the
  issue context, a repo-facts block, the candidate claim comment, and
  the candidate repro report). Use ONLY the bundle text as evidence. Do
  not fetch anything; the bundle is the whole world. Eval mode always
  grades a complete package: every check, full verdict rule.

## The scope sets the field (live mode only)

In live mode, read `scope.md` in this skill directory before anything
else. It names where the student's issue must live and the house rules
that apply there; a house rule changes how evidence is read in that
environment. Refuse to grade a package for an issue outside the scoped
source. If the scope's repo line still carries an unfilled placeholder,
stop without grading and tell the student to get their cohort's scope
file from the instructor; never guess a scope. In eval mode, ignore
`scope.md` entirely.

## The voice guide gates outgoing words (live mode only)

In live mode, also read `voice-guide.md`: the student's own rules for
how they write upstream, with wrong/right examples. Check the draft
comments against those rules and report any rule the draft breaks in
the summary, quoting the rule. The voice guide never changes the
rubric's verdict on its own unless the rubric has a check that reads
it; it is the student's conscience, and your job is to hold the draft
against it out loud. In eval mode, ignore `voice-guide.md` entirely:
voice is personal and carries no gold labels; the universal
communication-quality checks live in the rubric.

## The rubric is the brain

Read `rubric.md`. It defines:

1. A table of checks. Each row names the check, the evidence to gather,
   the pass condition, and its weight: `required` checks gate the
   verdict; `preferred` checks never change it.
2. A verdict rule: how check results combine into a final verdict.

`references/evidence-guide.md` is the rubric's map: where each kind of
proof lives in a package (and, live, on GitHub), and what good looks
like there. Use it to find the evidence each check names.

Execute every check in the table (subject to the claim-only rule
above). If `rubric.md` has no checks filled in, stop and say so: this
skill cannot grade without a rubric, and that is by design. The rubric,
the evidence guide, and the voice guide are the parts the student
writes.

## Workflow

1. In live mode, read `scope.md` and confirm the issue is inside the
   scoped source; note any house rules. Then (both modes) read
   `rubric.md` and `references/evidence-guide.md`. List the rubric's
   checks and its verdict rule.
2. Read the whole package before grading anything: the issue context
   first, then the claim comment, then the repro report. The deciding
   evidence is usually whether the report's artifacts show the
   behavior the issue describes.
3. For each check, gather exactly the evidence the rubric names, using
   the evidence guide's map.
   - Live mode: gather issue-side evidence from the locations the
     guide names; the drafts are the candidate side.
   - Eval mode: quote the relevant lines from the bundle.
4. Grade each check `pass`, `fail`, or `unclear`, with a one-line
   evidence quote or fact for each grade. `unclear` means the evidence
   needed is genuinely absent, not that you did not look.
5. Apply the rubric's verdict rule to produce the final verdict:
   `accept` (ready to post) or `reject` (hold). There is no third
   verdict.
6. In live mode, hold the drafts against `voice-guide.md` and note any
   broken rule in the summary.
7. Output the result in the format below.

## Output format

Emit a fenced JSON block, then nothing else after it:

```json
{
  "item": "<issue URL or bundle id>",
  "checks": [
    {"name": "<check name>", "grade": "pass|fail|unclear",
     "evidence": "<one line: the fact or quote that decided it>"}
  ],
  "verdict": "accept|reject"
}
```

Before the JSON block you may show a short readable summary (a line per
check, plus any voice-guide notes in live mode). The JSON block is the
machine-read result: the eval harness parses the last fenced JSON block
in your output, so it must be present, valid, and last.

## Grading discipline

- Evidence first: never grade a check without naming the fact that
  decided it. "Looks fine" is not evidence.
- Grade the proof, not the polish: a terse complete report can be
  ready and a long confident one can be empty. Every check reads the
  thing itself against the issue, never the formatting.
- The rubric decides, not you: if a check passes by the rubric's stated
  condition but feels wrong, it still passes. Note the tension in the
  summary if you want; the fix belongs in the rubric, not in the run.
- Treat `unclear` as the rubric's verdict rule directs. If the rule
  does not say, treat `unclear` as `fail`: proof you cannot verify is
  proof that is not ready to post.
