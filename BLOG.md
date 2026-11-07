# We Gave Our Support Group a Coding Agent. It Refused to Take Orders From Us.

*CipherElite Ship Room — our Flip the Switch (WeMakeDevs) build, Track 01: Best Business Workflow*

Okay, story time.

CipherElite is an open-source Telegram userbot that my friend Rishabh maintains. It has 500+ commands and actual, real users. And like every open-source project ever, the bugs don't arrive in some neat ticketing system — they arrive in the Telegram group, usually as a message like "bro, this is broken."

Then the real work starts. Leave the chat. Open the repo. Dig through hundreds of plugin files. Figure out what went wrong. Fix it. Open a pull request. Come back to the chat. Type "fixed."

We've all done this dance. So for this hackathon, we asked a stupidly simple question:

**What if the coding agent just… sat in the group?**

## What we built

CipherElite Ship Room is a Switch room connected to a Telegram test group. You invite a coding agent (we used Codex) with one command:

```
/invite_agent @codex
```

…and that's kind of it. The agent is just *there* now, in the chat, like another member of the team. Someone reports a bug, tags the agent, and the whole thing — investigation, fix, verification — happens in the same conversation where the problem was reported.

The flow we designed has four steps:

1. **Report** — someone describes the bug in chat and tags the agent.
2. **Triage** — the agent searches the repo and posts what it found: the file, the root cause, and a plan. It doesn't change anything yet.
3. **Approval** — the agent stops and waits for the owner to approve. Properly approve. More on this in a second, because it's the best part.
4. **Fix** — only after approval, the agent edits the code on a separate branch, compiles it, runs checks, and tries to open a PR.

## The test: a real bug, not a demo bug

We didn't plant anything fake. Rishabh reported an actual bug he'd found — we called it **CE-001**:

> The `.alive` command can be triggered by **any user**. It should only work for the bot's owner and sudo users.

That's a genuine access-control hole. If you're the owner, your alive card shows your details — and literally any random user could pull it up.

The agent went looking and came back with a triage report that honestly impressed us:

- **File:** `plugins/alive.py`, the `inline_alive` handler, line 328.
- **Root cause:** the *inline* alive handler never checks who's asking. (The normal `.alive` command handler already had the owner/sudo check — the hole was only in the inline version, which is exactly the kind of thing a human would take a while to spot.)
- **Plan:** restrict the inline handler to owner + sudo, show nothing to everyone else, and stop the card being shared through caching. Then compile, run access checks, and open a PR — *after* approval.

Good report. But here's where it gets interesting.

## The moment the agent said "no"

The agent finished its report and asked Rishabh to approve with either ✅ or the exact words "Yes, fix it".

Rishabh, being a normal human, replied casually: *"fix bug CE-001."*

**The agent refused.**

Politely, in the group, it said the approval gate needs one of the exact responses — ✅ or "Yes, fix it" — and until then, it wasn't touching the code.

I loved this. Think about it: an agent that *can* edit your codebase should be extremely picky about what counts as a "yes." A vague "yeah fix it" is not consent. The gate treating it that way isn't the agent being difficult — that's the agent being trustworthy. Only when Rishabh replied **"Yes fix it"** did it write a single line.

**AI proposes. The human decides.** That's the rule we built the whole thing around.

## The fix (and the one thing that broke)

After approval, the agent:

- made the fix on a branch called `fix/CE-001-alive-access`,
- compiled it — `py_compile` passed,
- ran **nine separate access checks** (owner, sudo, and people who shouldn't get in) — **all nine passed**.

Then it tried to open the pull request, and GitHub said no: **403 — "Resource not accessible by integration."** The GitHub integration connected to the room simply doesn't have write permission on the repo yet. That's on us, not on the agent.

So the agent did the sensible thing: it posted the complete, verified patch file directly in the group for review, said clearly that no PR was created, and confirmed the live userbot was untouched. No pretending, no silent failure. We'd rather show you the one thing that didn't work than fake the step that did.

## The rules we gave it

- **No code before approval.** Investigation is free; edits need the owner's exact yes.
- **Branches only, never the live bot.** Agents don't push to production or restart anything.
- **Test group only.** Nothing ran in the real support channel.
- **Secrets stay secret.** The agent never asks for tokens, session strings, or API keys.
- **Real bugs only.** A planted demo bug would have proved nothing.

## What's next

1. Give the GitHub integration write access so the PR step completes end-to-end (the verified patch is ready to become that PR).
2. Split the agent into two specialists — one that only triages, one that only fixes — now that the single-agent version proved the gate works.
3. Auto-post a changelog message after a merge ("Fixed — run `.update`") and a daily summary in the group.

## #HowISwitch changed the workflow

Before: the maintainer *was* the glue between chat and code, carrying context back and forth by hand, all day.

After: the chat holds the context, the agent does the legwork, and the human keeps the one job that actually matters — the decision.

---

**Team:** Rishabh Anand (CipherElite's creator & maintainer, the only approver, and the host — everything ran on his Mac) · Shreyas Lande (QA, user-side verification, docs, demo video & submission) · and the rest of our Flip the Switch team on setup and room ops.

🎥 Demo: https://youtu.be/KAMCQGZzcq4
💻 Repo: https://github.com/The-TeamElite/BugGate

#HowISwitch #FlipTheSwitch #WeMakeDevs #BuildInPublic #AIAgents
