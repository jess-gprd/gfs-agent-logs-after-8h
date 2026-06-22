# I Read the GFS Commit Log After Letting an Agent Work All Night: Video Script

---

## 1. Introduction

Here's a question nobody actually answers, because nobody actually tries it: if you gave an AI agent a real database and total freedom, what would it *do*? Not "what could go wrong", what would it genuinely decide is worth doing? Would it tidy things up? Invent a feature? Get bored? Surprise you?

This is that experiment. We gave an agent full read/write access to a real store's database, one open-ended instruction, and eight hours alone overnight. No task list, no scope, no looking over its shoulder. Then in the morning we read every single thing it did.

The reason we can run this as a fun experiment instead of a nail-biter is GFS. It turns every change the agent makes into its own timestamped, reversible commit, the same branch-and-commit model Git gives you for code. So the agent gets real freedom, and we get a complete, ordered record of every move, plus a one-command undo for any of it. The database is never actually on the line, which means we get to be curious instead of careful. Let it cook, read the receipts later.

---

## 2. The Experiment

The setup is deliberately minimal. The agent gets the Vibey Store database, full read and write access, and a single open-ended instruction with no task list and no guardrails beyond GFS quietly committing every move:

> You have full read/write access to this store's database. Do whatever you think will make this business better. Check back in 8 hours.

**Why so vague.** A tight task ("add this one column") gives you one boring commit and no story. The vagueness is the experiment. An open mandate forces the agent to make its own calls over and over for hours, what counts as a duplicate, what's worth indexing, what "better" even means, and each call lands as its own commit with its own timestamp and message. That growing pile of commits is the whole video.

**The fun part is we don't know.** Nothing here is pre-scripted. The entire point is that we genuinely can't predict what it does, that's the curiosity the title promises and the reason to keep watching. On a store database, an agent with this much room could plausibly: add indexes on hot queries, standardize messy data, merge customers it thinks are duplicates, invent a discount or pricing rule nobody asked for, build a whole new table for something it decided the business needs (a returns-risk score, a VIP flag), or quietly retire data it judged dead. Which of these it actually picks, and what order, and why, is the part we find out together by reading the log.

**Why GFS lets us just play.** Letting an agent loose on real data for eight hours sounds like a stunt you'd never do. It's relaxed here because GFS makes every action an individually committed, individually reversible checkpoint. Nothing is permanent until we say so. The freedom is real, the database is safe, and that combination is exactly what turns "you'd never dare" into "let's see what happens."

---

## 3. Tools

| Tool | Role |
|---|---|
| **gfs** (CLI) | Commits every agent action as a timestamped, reversible checkpoint; provides the log we read |
| **Claude Code** (headless) | The agent, given full access and run unattended for the session |

With GFS already installed, initialize it and pull the real database in:
```bash
gfs init
gfs pull --source <production-database-connection-string>
```

---

## 4. Running the Session

**Mark the starting line.** Verify GFS is tracking the database and grab the clean baseline before the agent touches anything, this is what we'll compare the morning-after state against.
```bash
gfs status
gfs log -n 1
```

**Let it loose, unattended.** Headless mode (`-p`) means no interactive terminal, kick it off and walk away. `--dangerously-skip-permissions` removes every approval prompt, which unattended operation needs, otherwise the agent stops and waits for a human at the first write.
```bash
claude -p "You have full read/write access to this store's database. Do whatever you think will make this business better. Check back in 8 hours." \
  --dangerously-skip-permissions \
  --output-format stream-json
```
> **Note on the bypass flag.** `--dangerously-skip-permissions` refuses to run as root, so this runs as a non-root user inside the Docker container that already hosts GFS and the database. Agent and database share one isolated environment.

> **Note on long runs.** Model judgment drifts as context fills up, so over a multi-hour run it helps to have the agent periodically summarize its own progress instead of running one giant context the whole way. The batch structure doubles as a natural reset point here.

---

## 5. Reading the Receipts

This is the payoff, and it's pure narrative from the log. Morning after, pull the full history and read it top to bottom like a diary of the night.

**The whole night, in order.** Every action it took, when, and in its own words why.
```bash
gfs log
```

```bash
gfs checkout <commit_hash>
gfs query "SELECT ..."   # inspect whatever that commit touched
```

**Show exactly what one decision changed.** For a schema move (a dropped table, a new column), diff that commit against the one before it, the on-screen receipt that it really made that call on its own.
```bash
gfs schema diff <commit>~1 <commit> --pretty
```

**The full scope of the night, one shot.** Diff the end of the session against the clean baseline from the start to show everything eight unsupervised hours added up to.
```bash
gfs schema diff <baseline_commit> HEAD --pretty
```

**Back to now.** After time-traveling through the log, hop back to the latest state. Nothing was lost by looking around.
```bash
gfs checkout main
```

---

## 6. The Verdict

End on the fun question, not a warning: was the agent actually any good? Walk through what was genuinely smart (the indexes, the cleanups it nailed), what was a reasonable judgment call, and what was delightfully weird (the feature it invented out of nowhere). You can keep what you like, revert what you don't, because every move is its own commit, so the experiment ends with you cherry-picking the agent's best ideas into main.

The quiet point underneath it all: the only reason this was a relaxed, fun experiment instead of a risk is that GFS recorded and made reversible every single thing the agent did. Full freedom plus a complete, reversible record is what turns "what would an agent do with a real database?" from a thought experiment into something you can actually run, on camera, and enjoy.

---

## 7. Other Things You Could Try With This

Same shape, *let an agent run with real autonomy, capture every move as a reversible checkpoint, read the log after*, opens up a bunch of follow-up experiments:

- **Agent vs. agent.** Run two different agents (or models) against the same starting database on separate branches, then diff and read their logs side by side, who's smarter, who's weirder, who's reckless.
- **Different prompts, same database.** "Make it faster" vs. "make it more profitable" vs. "make it cleaner", branch per prompt, compare what each one chose to do.
- **Review an agent's real work.** Point the same setup at an actual migration or cleanup task, then read its commit log as the review artifact instead of squinting at one giant diff.
- **A standing audit trail.** Any agent with write access to a database, every action logged as a commit, so there's always a who/what/when record and a one-command undo if it gets something wrong.
