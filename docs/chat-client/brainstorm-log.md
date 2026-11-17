# Custom chat client: brainstorm log

Working notes, not an approved design. Source: `custom-client-brief.md` (v0.9.0) plus the design conversation.
Status: project list **frozen at nine**. New ideas go under "Cherries on top" below.

## Product shape

A Discord-like server for the character engine: DMs, group channels, topic channels, and character-only channels.
Characters can message the user unprompted and talk among themselves while the user is away. Images in both
directions. Single user, local models only, reached over WireGuard.

## Projects (frozen, in build order)

1. **Conversation core**: engine-side store of channels and messages, per-channel state, retention, backups, conversation API.
2. **Web client**: installable web app (HTTPS, phone-friendly), controls instead of `/commands`.
3. **Notebook**: the user's private drafting space (isolate, flesh out, apply elsewhere).
4. **Orchestrator**: job queue with priority classes and idle detection.
5. **Server and channels**: DMs, groups, topic channels, character-only channels, who speaks next.
6. **Memory scoping**: gossip levels, memory provenance, secret overrides, encrypted off-machine backup, "what they know about me" page.
7. **Autonomy**: initiated texts, characters talking among themselves, push notifications.
8. **Images out**: per-character trained look, image backend, her decision to send a picture.
9. **Migration and retirement**: move assistant dock and management panels into the client; retire LibreChat and the old page.

Only 1 and 2 are the commitment (a working LibreChat replacement for DMs). 3 to 9 are additive. Only 1 and 2
get a full design now; 3 to 9 are designed when reached.

## Decisions so far

**Platform**
- Web app served by the engine, installable on the phone, HTTPS over WireGuard. No native app (not worth the cost).
- Conversation API beside `/v1`. LibreChat keeps working via `/v1` until retired; its fingerprinting and in-band
  `/command` code stays frozen there.
- Start fresh in the new client. No import of LibreChat chats.

**Conversations**
- Stored in the engine as a line of messages with IDs. Regenerate keeps the replaced reply as a hidden
  "didn't work" attempt, never shown back to the character. Editing an earlier message rewinds to that point.
- Generation continues engine-side if the client disconnects; the finished reply is there on return.
- Data model must be channel-shaped from day one (participants, not "the character"; a field for where each
  memory came from), even though only DMs ship first.

**Retention**
- A conversation expires only when it is idle for the set period AND has no unscanned memory. Active chats never expire.
- The engine records the last scanned message per conversation; messages past the mark mean "unscanned".
- Visible "unscanned" badge on chats.
- Memories from an expired chat just say "expired"; no pointer back.
- Backups exclude the message tables or are encrypted. Memory export goes off-machine, encrypted.

**Modes and forks**
- A chat's mode is fixed. The only mid-chat switch is backstage, an overlay that returns to the current mode,
  scene and standing direction (unless edited while backstage).
- Switching modes forks a new chat from a chosen point, opened as a side window. Fork = new chat with a parent
  (parent and fork point recorded), copying history up to the fork point. Inherits scene state and switches.
  Scan marks carry over so a shared prefix does not produce duplicate memories.
- Backstage exchanges stay visible in the transcript but are flagged out-of-story: excluded from scene ledger,
  memory scans, and her in-story context.

**Notes and notebook**
- A note is a small card in the original chat's margin, created from the side chat, editable before sending.
  Not a message in the line. The character only sees it if the user acts on it (as an aside-style director's hint).
- Main actions: steer the next reply; make it the standing direction; save to the notebook. Notes expire with their chat.
- The notebook is private to the user, spans all channels and characters, never expires, is included in the
  encrypted backup. Items can be promoted into a steer, standing direction, character memory, or a scenario/setting.

**Autonomy**
- Blend: an impulse pass decides whether she has a reason to reach out, inside rhythm limits; follow-ups from
  events feed motives later.
- Limits: per-character initiative setting (never, rare, sometimes, often), quiet hours, cap on unanswered messages.
- Mostly texts have a real reason; occasional light texts allowed for high-initiative characters (assistant's call).
- Characters can also be assigned to topic channels with a brief and a channel memory.

**Memory and knowledge**
- Per-character gossip level (sealed, discreet, chatty, loose), set on the character. The knower's level governs
  whether she lets something slip. Per-memory override (secret / may share).
- "What they know about me" page: stored records only (facts, relationship text, where things stand, with origin
  channel shown, editable/deletable).

**Images**
- Per-character trained look (the user will do the training), outfit and scene state feed the prompt, she decides
  when to send a picture; per-character setting for how readily.
- Images in (sending pictures to her) already works; remaining work is tuning each character's reactions.
- Images in expired chats expire with them unless kept.
- No image tool chosen yet; to be researched before project 8. No second machine; one shared chat model.

**Orchestration**
- Priority classes: live reply, reply dependencies, autonomous turns, image generation, housekeeping.
- Idle detection (active, idle, away): background work and image batches ramp up when idle.

## Open items

- Idle period before a chat expires (not yet chosen); whether a per-chat "keep" pin exists.
- Hard ceiling on "unscanned, never expires" (optional).
- Which model, its size, and whether it can see images (user said not important now).
- Image backend choice (research later).

## Cherries on top (backlog; not promoted until a project finishes)

- In-character "how she sees me" read as an extra view next to the stored records.
- Analysis of rejected ("didn't work") attempts: patterns, and tuning character style notes from them.
- Tuning each character's reaction to images (character work, not a build).
- Roadmap items: `/direct` (partly covered by the steer button), write my line, free-form character state over
  time (trust, mood), prompt budget view, worlds and global rules, relationships between characters, the
  Workshop (free-form creative assistant), interview steering, tag history, in-character check that learns.
- Selective import of a few specific LibreChat chats (only if wanted later).
