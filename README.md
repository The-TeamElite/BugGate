# BugGate

A Telegram support bot that actually checks your code before bothering you.

We built this for our own problem. Bug reports come in the support group, get lost in chat, and then we have to go hunt for the bug in the repo ourselves. BugGate fixes that flow.

It connects a Telegram group to a Codex agent through Switch. Switch handles all the communication between the bot and the agent — we didn't build any direct link.

**Simple rule: No tag, no response. No approval, no fix.**

Repo: [The-TeamElite/BugGate](https://github.com/The-TeamElite/BugGate)

## How it works

1. Someone finds a bug and tags the agent in the group.
2. The agent opens the repo we gave it in Switch and checks if the bug is real.
3. If it's not real, it just says so in the group. End of story.
4. If it's real, it tags the owner and explains what's wrong and where.
5. Owner says "fix it" — only then the agent writes the fix.
6. Fixed code is uploaded in the same group, with the changes.
7. Owner checks it and pushes it to GitHub himself.

That's it. The owner stays in control the whole time.

### Example

From our CipherElite support group:

> @codex I have found a bug in the CipherElite userbot: the 'alive' command is accessible to all users, whereas it should be restricted to the server owner and sudo users.

BugGate checked the repo, confirmed the `alive` command really had no owner/sudo check, tagged the owner, and waited. Fix was made only after the owner approved it.

## Setup

You need: Switch Console, a Codex account, and a Telegram account with a username.

Full docs: [Switch quickstart](https://docs.switchagents.ai/switch-rooms/getting-started/index)

**1. Start Switch**
Install Switch Console, open it, tap Get started → On this computer → Start. Wait till it says local server is running.

**2. Connect Codex**
Go to Settings → Agent providers → Codex. It should show **Signed in**. If it says Not signed in, copy the sign-in command from there, run it in terminal, then Recheck.

**3. Make the Telegram bot**
Talk to @BotFather, send `/newbot`, set a name and a username ending in `bot`. Save the token somewhere safe, you only get it once.

Important: before adding the bot anywhere, go to `/mybots` → your bot → Bot Settings → Group Privacy → **Turn off**. If you do this later, you'll have to remove and re-add the bot.

**4. Connect Telegram in Switch**
In Switch Console, open your server's Home page → Connect → Telegram. Paste the bot token and username, give it a name, and Connect.

**5. Add the bot to your group**
Add the bot to your support group. Switch will auto-create a room for that group. No admin rights needed for a group.

Then link your own account: send one message in the group, go to Home in Switch Console → Link account → select yourself.

**6. Invite the agent**
In the group, type:

```text
!list-agents
!invite-agent @agent-name
!set-alias @agent-full-name @codex
```

The last line is optional, it just makes `@codex` work as a short name in that room.

Test it by tagging the agent with a bug report. If it replies without checking the repo, your room instructions are missing — see next section.

## Agent introduction we set in Switch

This goes in the room's Instructions in Switch. Every agent in the room reads it. Just replace the project name and repo link with yours.

```text
You are BugGate, the support agent for this Telegram group.

Project: CipherElite — a Telegram userbot.
Repository: <paste your GitHub repo link here>
This group is the project's support group. The owner is also in this group.

Your job:
- Help users and the developer with bug and error reports.

Rules:
1. Reply only when you are tagged, like @codex. Ignore normal chat.
2. When someone reports a bug, check the repository first. Don't guess.
3. If the bug is not real, say so in the group. Don't escalate it.
4. If it is real, tag the owner and explain the issue clearly.
5. Don't fix anything on your own. Wait for the owner to say fix it.
6. After approval, upload the fixed code in this group with the changes.
7. Never push to GitHub yourself. The owner will verify and do that.
```

## Why this helps

**For the owner:** No more digging through chat and code for every report. You only get tagged when a bug is already verified, with an explanation.

**For users:** Just send one message in the group you already use. No issue template, no GitHub account needed. And you can see if your report was real or not.

**For any project:** Same setup works anywhere — you just need a Telegram group, a repo, and a maintainer. We used CipherElite as our example because that's where we face this daily.

## Hackathon

Built for [Flip the Switch](https://www.wemakedevs.org/hackathons/switch) — WeMakeDevs x DevTools Academy x Switch, Nov 4–7, 2026.

**Track 01 — Best Business Workflow**

We picked this track because this is not a demo bot. It's a real workflow — user, owner and agent finishing a bug together in one chat. Switch is the core of it: without Switch there's no room, no connection, and the agent never hears the report.



Team: The TeamElite


---

## Demo — the CE-001 bug (live run, Nov 7, 2026)

🎥 **Demo video:** https://youtu.be/KAMCQGZzcq4
📝 **Blog post:** [BLOG.md](https://github.com/The-TeamElite/BugGate/blob/main/BLOG.md)

The demo shows a real end-to-end run in the test group (8:48–8:53 PM IST):

1. `/invite_agent @codex` brings the coding agent into the Telegram group.
2. **The report:** CipherElite's `.alive` command is accessible to **all users** — it should be restricted to the server owner and sudo users.
3. **Triage (CE-001):** the agent searched the repo and pinpointed `plugins/alive.py`, `inline_alive` handler, **line 328** — the inline handler has **no owner/sudo check** and returned the shared alive card to any requester (the `.alive` message handler already uses the owner/sudo decorator). Fix plan: restrict the inline handler to owner/sudo, return no results to others, and make results personal (no shared caching).
4. **The approval gate:** the agent stopped and asked for ✅ or "Yes, fix it". A casual "@codex fix bug CE-001" was **refused** — only the exact approval unlocked the code. *AI proposes. The human decides.*
5. **Fix + verification:** branch `fix/CE-001-alive-access`; inline alive now checks owner/sudo and disables shared caching. `python -m py_compile` **passed** and **nine isolated access checks passed** (owner / sudo / unauthorized).
6. **Honest blocker:** PR creation was blocked — GitHub returned **403 "Resource not accessible by integration"** (the connected integration lacked repository write access). The agent posted the verified patch (`CE-001-alive-access.patch`) in the group instead. **No PR was created, and the live userbot was not changed.**

## Team

- **Rishabh Anand** — lead developer of CipherElite, only approver, host
- **Shreyas Lande** — QA / verification, documentation, demo video, submission
- **Chetan Anand & Himanshu** — team (setup, room ops, docs)
