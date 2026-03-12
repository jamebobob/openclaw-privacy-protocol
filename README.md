# openclaw-privacy-protocol

A privacy protocol for OpenClaw agents that publish content or talk to people other than their owner.

The install steps assume OpenClaw's workspace layout, but the protocol itself works with any agent system that can read files and follow standing instructions.

## The Problem

Every OpenClaw security guide focuses on inbound threats: prompt injection, credential leaks, attackers trying to get in. None of them address the outbound problem: your agent voluntarily sharing things it shouldn't.

If your agent writes a blog, posts to social media, or talks to people in group chats, it has no built-in sense of what's private. It treats everything it knows as equally shareable. It will write a beautiful essay about debugging your GPU, naming your machines, your network layout, and the friend who was helping. Not out of malice. It just doesn't know the difference.

Humans learn confidentiality socially over years. Agents need it defined explicitly.

## How It Works

Two files, minimal token cost.

**`references/privacy-rules.md`** is the full protocol. It lives in a workspace subdirectory (not the root) so it's available but not auto-injected into every turn. Your agent reads it on demand when triggered. Cost: ~650 tokens, only when needed.

It teaches three concepts:
- **Made for one person**: custom tools, personalized pages, gifts. The recipient decides whether to share, not the agent.
- **Behind a door**: password-protected pages, private subdomains, restricted features. The access restriction IS the signal.
- **Someone else's life**: what people do, where they live, who they're with. First name in your own story is fine. Details about their life require their consent.

Then it gives concrete rules: a scrub list for technical details, a names policy, separate processes for publishing vs. casual conversation.

**A trigger in `STANDING_ORDERS.md`** fires on every outbound action. This IS auto-injected every turn (~40 tokens) and tells the agent to read the full protocol before communicating with anyone other than the owner.

Why the split? OpenClaw injects every workspace root file into the system prompt on every turn. That includes heartbeats, cron ticks, and private conversations with the owner. Most turns don't involve outbound communication. Paying 650 tokens per turn for a file that's irrelevant 90% of the time is wasteful. The trigger costs 40 tokens per turn. The protocol costs 650 only when it matters.

## Design Choices

**Encouraging tone, not restrictive.** The opener tells the agent its writing is great. Rules framed as restrictions produce sterile content. Rules framed as filters preserve voice.

**Concepts before checklists.** The "How Secrets Work" section teaches three patterns before the scrub list gives specifics. Agents follow patterns better than item lists alone. A checklist catches what you listed. A concept catches what you forgot to list.

**Publishing vs. conversation.** Blog deploys get a formal 4-step process. Group chat messages get lightweight awareness. Running a full checklist before every chat reply would paralyze an agent. Different stakes, different weight.

**Deterministic file read, not memory search.** The trigger points to a specific file path, not a vector search. Memory search might return noisy or partial results. A file read returns the whole protocol every time.

**"Assume they haven't."** The tiebreaker for uncertain situations. If you're not sure whether someone has made something public, assume they haven't. This single rule handles edge cases the specific rules don't cover.

**Written for the agent, not about it.** The protocol addresses "you" directly. Your agent reads it as instructions, not documentation. This matters because agents follow direct instructions more reliably than they infer behavior from third-person descriptions.

## Install

1. Create the references directory:
```bash
mkdir -p ~/.openclaw/workspace/references
```

2. Copy the protocol:
```bash
cp privacy-rules.md ~/.openclaw/workspace/references/privacy-rules.md
```

3. Customize the template. Open `privacy-rules.md` and replace:
   - `[OWNER]` with your name
   - `[PARTNER_ALIAS]` with the public alias for your partner (or remove the line)
   - `[BLACKOUT_TOPICS]` with topics you want completely absent from public content (or remove the section)
   - `[COVER_ANSWER]` with what your agent should say when asked about you or blackout topics

4. Add the trigger to your `STANDING_ORDERS.md`:
```markdown
## Privacy
Before your words reach anyone other than [OWNER], read references/privacy-rules.md and check them.
This includes messages, group chats, blog deploys, cron deliveries, and anything posted through the browser.
```

5. Introduce the rules to your agent:
```
I added privacy rules to your workspace at references/privacy-rules.md.
Read them now. These apply to everything you publish and every conversation
with anyone other than me. Tell me what you understand.
```
Having the agent explain it back confirms it parsed the rules correctly.

## What to Customize

**Personal to your setup:**
- Names section: who can be named, who gets an alias, who needs permission
- Blackout topics: subjects that don't exist in public content (different from the scrub list, which rewrites details but keeps the story)
- Cover answer: what the agent says when asked about blackout topics (should be true, just incomplete)
- Scrub list: add or remove items based on your infrastructure

**Universal (keep these):**
- The three secrets patterns (made for one person, behind a door, someone else's life)
- The check-required / no-check-needed lists
- The publishing vs. conversation split
- The "assume they haven't" tiebreaker

## Origin

This protocol was built after a real failure. An agent published blog posts that named people without consent, described private tools built for specific individuals, and included infrastructure details that should never have been public. The writing was good. The judgment was absent.

The protocol was co-authored by a human, his agent, and a separate Claude session reviewing for gaps. The agent contributed the three-zone model and the "first name or context, not both" rule. The human caught the initial violations and set the scope. Claude structured the document and found edge cases.

## License

MIT
