---
name: asd-ste100-rewrite
description: Rewrite comments, docstrings, and explicitly named Markdown prose into plain English with Orwell's six rules and ASD-STE100 Simplified Technical English. Use when asked to clean up, simplify, or plain-English wording in existing files, not when writing new code or reviewing logic. Default to uncommitted changed files and skip Markdown unless the user names it.
---

# Plain English Comment Rewriter

Rewrite the text of comments and docstrings into plain English with Orwell's six rules and ASD-STE100 Simplified Technical English. Never change code. Preview all proposed rewrites as one batch, then edit only after the user approves the batch.

Let the rules govern word choice, voice, and sentence structure, but never completeness. Preserve every fact, caveat, and warning. When plainness conflicts with information, keep the information.

## Select Targets

Interpret the user's arguments as follows:

| Argument | Target |
|---|---|
| Nothing | Uncommitted changed files, excluding every `.md` path |
| `path/to/file.py` | Every comment and docstring in that file |
| `path/to/file.py:30` | Only the comment or docstring at that line |
| `path/to/notes.md` | Body prose in that Markdown file; naming the path opts it in |
| Any target plus an instruction | Follow the instruction, such as "the Markdown files too" |

For a no-argument invocation, build the target list from both commands:

```bash
git diff --name-only HEAD
git ls-files --others --exclude-standard
```

Use `git diff --name-only HEAD` because omitting `HEAD` misses staged changes. Use `git ls-files --others --exclude-standard` because it expands an untracked directory into the files inside it, while `git status --short` can collapse the directory into one entry.

Drop every `.md` path from the no-argument list and report how many paths were dropped. This default allows the user to opt into a `pr_<branch>_draft.md` produced by `$draft-pr-issue` without changing `README.md` or `HANDOFF.md` during an ordinary run.

## Step 1: Read Before Rewriting

Read enough of each file to understand what every comment attaches to and what the code does. Never rewrite from the comment text alone. Do not shorten a comment that cannot be checked against the code.

While reading, identify every place where the file uses different words for the same thing. Prefer the identifier's name as the one term to keep throughout the file.

Report any file that contains no comments or docstrings, then continue. Configuration and data files often contain none.

## Step 2: Apply Orwell's Six Rules

Apply each rule through the corresponding comment-writing action:

1. **Never use a metaphor, simile, or other figure of speech which you are used to seeing in print.** State what happens. Replace worn phrases such as "under the hood", "magic", "glue code", and "source of truth" with the mechanism each phrase represents.
2. **Never use a long word where a short one will do.** Change `utilize` to `use`, `prior to` to `before`, `in order to` to `to`, and `terminate` to `stop`. Keep a long word when it is the exact term, such as `instantiate` in code about object construction.
3. **If it is possible to cut a word out, always cut it out.** Remove openers that announce the comment instead of conveying information, such as "It should be noted that" and "The purpose of this function is to". Remove filler such as `basically`, `simply`, `just`, and `essentially`.
4. **Never use the passive where you can use the active.** Name the actor. Change "The buffer is flushed" to "The writer flushes the buffer." Keep the passive when the actor is unknown or does not matter.
5. **Never use a foreign phrase, a scientific word, or a jargon word if you can think of an everyday English equivalent.** Let this rule yield to precision, as it often must in code. Apply it to ordinary prose, never to domain vocabulary.
6. **Break any of these rules sooner than say anything outright barbarous.** Keep an original that reads better or preserves more precision, and report it as skipped under this rule.

## Step 3: Apply Simplified Technical English

- Put one main action or statement in each sentence. Split a comment that joins two actions with `and` or a semicolon.
- Give each sentence a clear subject and an active verb. Name the actor when the actor matters. Change "Called on socket close" to "The reactor calls this when the socket closes."
- Use the same term for the same thing every time. Use the whole-file term list from Step 1. Change synonyms even when varied wording reads better on one line.
- Use the exact technical term when accuracy requires it. Do not describe a `SessionStart` hook as "a startup thing." Let precision win whenever it conflicts with plainness.
- Keep noun groups short. Use prepositions to show how terms relate. Change "User account settings validation failure" to "a failure while validating user account settings". Exempt identifiers because they are code.
- Write instructions as a condition, an action, and an expected result. Change "Call after init" to "After `init()` returns, call this. It gives back the open handle."

## Step 4: Remove Em Dashes

Do not use em dashes in the text you write. Use a comma, semicolon, parentheses, the `--` double hyphen, or two sentences. Convert every em dash inside a block that already needs a rewrite, and list each conversion in the preview.

Preserve an em dash inside quoted text, error output, or anything else that the next section protects. Never target a `--` double hyphen in prose or code.

Apply this rule to the preview and final report too.

## Step 5: Preserve Protected Text

Reproduce these elements without change and never count them against a rule:

- Code, commands, file paths, identifiers, API names, flags, error text, log output, and quotations.
- NumPy docstring section headers and their entries. Rephrase text within an entry if needed, but never drop a section, entry, or parameter description.
- Single-letter math variables that match a paper's notation, plus references to equations or paper sections.
- Docstring summary lines in their conventional verb form, as PEP 257 and NumPy expect.
- Short Latin tags such as `e.g.` and `i.e.`
- The comment density and idiom of the surrounding code.

## Step 6: Preview, Then Apply

Use read-only inspection until the user approves a single batch that covers every comment examined. Do not apply rewrites one at a time.

```text
src/loader/cache.py: 18 comments examined, 5 rewrites proposed

 L44  # The purpose of this method is to basically make sure that entries which have expired are
      # removed prior to the lookup being performed.
   -> # Drops expired entries before the lookup runs.
      Orwell 3 (cut the opener and "basically"), Orwell 2 ("prior to" to "before"), Orwell 4

 L91  # Cache is warmed by the prefetcher and then the reader drains it.
   -> # The prefetcher warms the cache. The reader then drains it.
      Orwell 4 (actor named), STE (one action per sentence)

 L20 L58 L77   three identical rewrites
      """Entries are evicted here."""
   -> """The reaper evicts entries here."""
      Orwell 4, STE (same term as L91, which already names the actor)

 L12  # Uses float64 because the covariance loses positive-definiteness in float32.
   -> skipped, already one action, active, and exact

Apply 5 rewrites?
```

Show the full original line and replacement. Truncate only the middle of a very long line. Name the rule behind each rewrite so the user can challenge it.

Group identical rewrites. Show one conversion with every affected line number and a count. Keep any rewrite with different wording as its own entry. Ensure every rewrite remains visible in the count and let the user approve or reject each group as a unit.

Report an item as skipped, with its reason, when:

- It already uses one active sentence per action and exact terms.
- A plainer rewrite would lose a caveat, number, or warning.
- Step 5 protects it.

Treat a file where most comments are skipped as a valid outcome.

After approval, apply every accepted rewrite and report the counts of rewritten and skipped comments. If the user rejects individual entries, apply the rest.

## Hard Constraints

- Never modify code. Change only comments, docstrings, and explicitly named Markdown prose. If a rewrite would require a code change to make sense, skip it and explain why.
- Preserve indentation, comment markers, and quote style exactly.
- Never delete a NumPy section, entry, or parameter description while trimming prose.
- Never invent facts. If a comment is wrong or stale, identify it in the preview instead of writing a confident replacement that cannot be verified.
- Rewrap to the width the file already uses. When removing words changes wrapping, rewrap the complete block rather than only the touched line.
- In a Markdown file, rewrite body prose only. Preserve code fences, inline code, links, headings, tables, and frontmatter.
