# I Read the GFS Commit Log After Letting an Agent Work All Night

## Metadata

| Field | Value |
|-------|-------|
| **Title** | I Let an AI Agent Loose on a Real Database for 8 Hours. Here's What It Did. |
| **Series** | GFS Use Cases |
| **Status** | `draft` |
| **Target Length** | 4–5 min |

**Summary**
> What would an AI agent actually do with a real database and total freedom overnight? This video runs that experiment: full read/write access, one open-ended instruction, eight hours unattended. The only reason it's a fun experiment instead of a disaster waiting to happen is GFS, which commits every agent action as a timestamped, reversible checkpoint. The next morning, we read the log like a diary and find out together.


## Production Setup

### Demo Stack

| Tool | Version / Notes |
|------|-----------------|
| `gfs` CLI | installed, authenticated, tracking Vibey Store DB |
| `claude` CLI | Claude Code, headless mode |
| Docker | running, non-root user inside container |
| Terminal | clean prompt, font ≥ 18, dark theme |

### Assets

- [ ] Vibey Store database already pulled and committed as baseline on `main`
- [ ] Agent session already completed — `gfs log` shows real overnight commits
- [ ] A few interesting commits pre-identified: one smart one, one weird one
- [ ] `gfs status` confirmed on `main` before recording

### Pre-Roll Checklist

- [ ] Terminal: one pane open, history cleared
- [ ] `gfs log` output looks good — enough commits to tell a story
- [ ] Notifications off
- [ ] OBS: all scenes tested
- [ ] Clock hidden from taskbar


## Scene Types

| Tag | Layout | When to use |
|-----|--------|-------------|
| `[INTRO]` | Brand animation | Auto, no action needed |
| `[CAM]` | Full-frame webcam, no screen | Greeting, concept explanations, CTA |
| `[SCREEN]` | Full screen, no cam | Dense code the viewer needs to read without distraction |
| `[SCREEN+CAM]` | Screen with small PiP webcam | Actively running commands or writing code |
| `[SPLIT]` | Cam beside screen, side by side | Talking about something on screen without actively coding |
| `[OUTRO]` | Brand animation | Auto, no action needed |


## Script


### [CAM] Introduction

**line:**
- greet: "Hi, my name is Jess from the Guepard community building team"
- today's video is a genuine experiment, not a tutorial
- what the viewer will see: an agent that ran unsupervised all night, and reading what it actually did


### [INTRO]


### [CAM] The Hook

**line:**
- nobody ever actually answers this question: if you gave an AI agent a real database and total freedom, what would it *do*?
- not "what could go wrong" — what would it genuinely decide is worth doing?
- tidy things up? invent a feature? surprise you?

**line:**
- this video is that experiment: full read/write access, one vague instruction, eight hours alone overnight
- we don't know what's in the log yet — we find out together


### [CAM] Why This Isn't Scary

**line:**
- letting an agent loose on real data for eight hours sounds like a stunt you'd never do
- GFS is why it's relaxed: every action the agent takes becomes its own timestamped, reversible commit
- same model Git gives code — nothing is permanent until we say so
- full freedom for the agent, complete audit trail and one-command undo for us


### [SCREEN+CAM] The Setup

**line:**
- show the starting state: GFS tracking the database, clean baseline on main

**action:**
```bash
gfs status
gfs log -n 1
```

**line:**
- the prompt was deliberately vague — a tight task gives one boring commit, vagueness forces the agent to make its own calls for hours
- each call lands as its own commit with a message — that pile is the whole video

**action:**
```bash
claude -p "You have full read/write access to this store's database. \
Do whatever you think will make this business better. Check back in 8 hours." \
  --dangerously-skip-permissions \
  --output-format stream-json
```

**line:**
- that was last night — now it's morning
- mention `--dangerously-skip-permissions` runs non-root inside the container, not on bare metal


### [SCREEN+CAM] Reading the Log

**line:**
- pull the full history: every action, in order, in the agent's own words

**action:**
```bash
gfs log
```

**line:**
- scroll through and react naturally — pick out the first interesting commit message, read it aloud


### [SPLIT] Narrating the Night

**line:**
- pick 2–3 commits that tell a story: one obviously smart move, one judgment call, one genuinely weird or unexpected one
- for each: read what the agent said it was doing, then show what actually changed

**action:**
```bash
gfs checkout <commit_hash>
gfs query "SELECT ..."
```

**line:**
- point at the result on screen, react — did it actually do what it claimed?


### [SCREEN+CAM] One Decision Under the Microscope

**line:**
- pick the most interesting schema move — a new table, a dropped column, something the agent invented
- diff that single commit against the one before it

**action:**
```bash
gfs schema diff <commit>~1 <commit> --pretty
```

**line:**
- this is the receipt: the agent made that call entirely on its own


### [SCREEN+CAM] The Full Picture

**line:**
- zoom out: everything eight hours of unsupervised work added up to, in one diff

**action:**
```bash
gfs schema diff <baseline_commit> HEAD --pretty
```

**line:**
- hop back to the present — nothing was lost by time-traveling through the log

**action:**
```bash
gfs checkout main
```


### [CAM] The Verdict

**line:**
- end on the fun question, not a warning: was the agent actually any good?
- what was genuinely smart, what was a reasonable call, what was delightfully weird
- the experiment ends with cherry-picking: keep what you like, revert what you don't, because every move is its own commit

**line:**
- the quiet point underneath all of this: GFS is what made this a fun experiment instead of a risk
- full freedom plus a complete reversible record turns "you'd never dare" into "let's see what happens"
- teaser: a few follow-up experiments this opens up — agent vs. agent, different prompts same database, using the commit log as a code review artifact


### [OUTRO]


## Notes & Cuts

- The log walkthrough is unscripted — react to what's actually there, don't stage it
- Pre-identify the 2–3 best commits before recording so you know where to land
- The weird/unexpected commit is the emotional peak of the video — give it room
- `--dangerously-skip-permissions` will raise eyebrows: explain the non-root container context briefly, don't skip it
- "Let it cook, read the receipts later" is a good throughline phrase — use it in the hook and echo it in the verdict
