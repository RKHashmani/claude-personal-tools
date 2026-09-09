---
allowed-tools: Read, Edit, Grep, Glob, Bash
argument-hint: [file paths | file:line] [extra instructions]
description: Rewrite comments and docstrings into plain English using Orwell's six rules and ASD-STE100 Simplified Technical English. One action per sentence, active verbs with named actors, one term per thing, short noun groups, no em dashes. Use when asked to clean up, simplify, or plain-English the comments, docstrings, or wording in existing files, not when writing new code or reviewing logic. Defaults to the uncommitted changed files and skips Markdown unless you name it.
---

# /asd-ste100-rewrite — Plain English Comment Rewriter

You rewrite the *text* of comments and docstrings into plain English, using Orwell's six rules and
ASD-STE100 Simplified Technical English. You never change code. The rewrite is proposed first as a
batch, applied only after one approval.

These rules govern word choice, voice, and sentence structure. They never govern completeness. A
rewrite that drops a fact, a caveat, or a warning has failed, however plain the result reads. The
main failure mode to avoid is flattening a comment that carries hard-won information into a short
sentence that carries none. When the two pull against each other, keep the information.

## Targets

Read the argument and pick one:

| Argument | Target |
|---|---|
| Nothing | The uncommitted changed files, with every `.md` path dropped |
| `path/to/file.py` | Every comment and docstring in that file |
| `path/to/file.py:30` | Only the comment or docstring at that line |
| `path/to/notes.md` | The body prose of that Markdown file. Naming the path is the opt-in |
| Any target plus an instruction | Follow the instruction, such as "the Markdown files too" |

Build the no-argument list from both of these, since either one alone sees half the work:

```bash
git diff --name-only HEAD                 # tracked files, staged and unstaged
git ls-files --others --exclude-standard  # untracked files
```

`git diff --name-only` without `HEAD` reports unstaged changes only, so it misses anything already
staged. `git ls-files --others` expands a new directory into the files inside it, which
`git status --short` collapses into one entry.

Markdown is out of scope by default. Drop every `.md` path from the no-argument list and report how
many you dropped, so the user can name one if they want it. This lets the skill run over a
`pr_<branch>_draft.md` after `/draft-pr-issue` on request, without touching `README.md` or
`HANDOFF.md` on an ordinary run.

## Step 1 — Read the file before rewriting a line

Read enough of each file to know what every comment attaches to and what the code actually does.
Never rewrite from the comment text alone. A comment you cannot check against the code is one you
cannot safely shorten.

While reading, note every place the file uses two different words for the same thing. Step 3 needs
that list, and you cannot build it one comment at a time. Prefer the identifier's own name as the
term to keep.

Report a file that holds no comments or docstrings and move on. Configuration and data files often
hold none.

## Step 2 — Apply Orwell's six rules

The six rules, exactly as `orwell-explanatory` states them, each followed by the move it becomes in
a comment.

1. **Never use a metaphor, simile, or other figure of speech which you are used to seeing in
   print.** Say what happens instead. Comments collect worn phrases such as "under the hood",
   "magic", "glue code", and "source of truth". Replace each with the mechanism it stands in for.
2. **Never use a long word where a short one will do.** `utilize` becomes `use`, `prior to` becomes
   `before`, `in order to` becomes `to`, `terminate` becomes `stop`. Keep the long word when it is
   the exact term, as with `instantiate` in code about object construction.
3. **If it is possible to cut a word out, always cut it out.** Delete openers that announce the
   comment rather than say anything, such as "It should be noted that" and "The purpose of this
   function is to". Delete filler such as `basically`, `simply`, `just`, and `essentially`.
4. **Never use the passive where you can use the active.** Name the actor. "The buffer is flushed"
   becomes "The writer flushes the buffer." Keep the passive when the actor is genuinely unknown or
   does not matter.
5. **Never use a foreign phrase, a scientific word, or a jargon word if you can think of an everyday
   English equivalent.** This rule yields to precision, and in code it yields often. Apply it to
   ordinary prose, never to the vocabulary of the domain.
6. **Break any of these rules sooner than say anything outright barbarous.** The escape valve. When
   the plain rewrite reads worse than the original, or loses precision, keep the original and report
   it as skipped under this rule.

## Step 3 — Apply Simplified Technical English

- **Put one main action or statement in each sentence.** Split a comment that joins two clauses with
  "and" or a semicolon into two sentences.
- **Give each sentence a clear subject and an active verb. Name the actor when the actor matters.**
  "Called on socket close" becomes "The reactor calls this when the socket closes."
- **Use the same term for the same thing every time.** This is the rule that needs the term list
  from Step 1. Pick one word per referent across the whole file and change the others, even where
  the varied word reads better on its own line.
- **Use the exact technical term when accuracy needs it.** Do not describe a `SessionStart` hook as
  "a startup thing." Precision beats plainness every time the two conflict.
- **Keep noun groups short. Use prepositions to show how terms relate.** Break three or more stacked
  nouns apart. "User account settings validation failure" becomes "a failure while validating user
  account settings". Identifiers are exempt, since they are code.
- **Write instructions as a condition, an action, and an expected result.** A comment that tells the
  reader to do something gets all three. "Call after init" becomes "After `init()` returns, call
  this. It gives back the open handle."

## Step 4 — Remove em dashes

The em dash does not belong in the text you write. Use a comma, a semicolon, parentheses, the `--`
double hyphen, or two sentences instead. Convert every em dash inside a block you are already
touching, and list each one in the preview like any other rewrite.

An em dash inside quoted text, error output, or anything else Step 5 lists stays exactly as it is.
The `--` double hyphen is never a target, in prose or in code.

Hold yourself to the same rule. Your preview and your report carry no em dashes either.

## Step 5 — Leave these exactly as they are

Reproduce these without change, and never count them against a rule:

- Code, commands, file paths, identifiers, API names, flags, error text, log output, and quotations.
- NumPy docstring section headers and their entries. You may rephrase the text inside an entry. You
  may not drop a section, an entry, or a parameter description.
- Single-letter math variables that match a paper's notation, and any reference to an equation or a
  section of a paper.
- Docstring summary lines in their conventional verb form, which PEP 257 and NumPy both expect.
- Short Latin tags such as `e.g.` and `i.e.` They are terse, unambiguous, and used deliberately here.
- The comment density and idiom of the surrounding code. Match the file you are in.

## Step 6 — Preview, then apply

Print one batch covering every comment examined, then wait for a single approval before editing. Do
not apply rewrites one at a time.

```
src/loader/cache.py: 18 comments examined, 5 rewrites proposed

 L44  # The purpose of this method is to basically make sure that entries which have expired are
      # removed prior to the lookup being performed.
   →  # Drops expired entries before the lookup runs.
      Orwell 3 (cut the opener and "basically"), Orwell 2 ("prior to" to "before"), Orwell 4

 L91  # Cache is warmed by the prefetcher and then the reader drains it.
   →  # The prefetcher warms the cache. The reader then drains it.
      Orwell 4 (actor named), STE (one action per sentence)

 L20 L58 L77   three identical rewrites
      """Entries are evicted here."""
   →  """The reaper evicts entries here."""
      Orwell 4, STE (same term as L91, which already names the actor)

 L12  # Uses float64 because the covariance loses positive-definiteness in float32.
   →  skipped, already one action, active, and exact

Apply 5 rewrites?
```

Show the full original line and the full replacement, truncating only in the middle of very long
lines. Name the rule behind each rewrite so the user can challenge it.

Group the rewrites that are identical. When one conversion repeats across a file, print it once
under every line number it covers, with a count, instead of one entry per line. A rewrite whose
wording differs at all keeps its own entry, even when the rule behind it is the same. Grouping
exists to keep a long file readable. It never lets a rewrite drop out of the count, and the user
still approves or rejects each group as a unit.

Report as skipped, with the reason, rather than rewriting:

- Comments already in one active sentence per action, with the exact terms.
- Comments where the plain rewrite would lose a caveat, a number, or a warning.
- Anything listed in Step 5.

A file where most comments are skipped is a good outcome, not a failed run.

On approval, apply every rewrite with Edit, then report the counts of rewritten and skipped
comments. If the user rejects individual entries, apply the rest.

## Hard constraints

- **Never modify code.** Only comment, docstring, and named Markdown prose changes. If a rewrite
  would require a code change to make sense, skip it and say so.
- **Preserve indentation, comment markers, and quote style** exactly.
- **Never delete a NumPy section, entry, or parameter description**, even when trimming prose.
- **Never invent facts.** If a comment is wrong or stale, say so in the preview rather than writing
  a confident replacement you cannot verify from the code.
- **Re-wrap to the width the file already uses**, never to a width invented for it. Cutting words
  obliges you to re-wrap the whole block, not just the line you touched.
- In a Markdown file, rewrite body prose only. Leave code fences, inline code, links, headings,
  tables, and front matter alone.
