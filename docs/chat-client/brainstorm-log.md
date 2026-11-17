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
- A conversation expires only when it is idle for **30 days** AND has no unscanned memory. Active chats never expire.
- A per-chat "keep" pin exempts a chat from expiry.
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

### Day-2 nice-to-haves (brainstormed after the freeze; none promoted)

**Presence and feel**
- Character presence and status (online, away, asleep) derived from her rhythm, shown in the sidebar.
- Typing indicators, natural delays, "seen" marks; characters reacting to messages with emoji, Discord-style.
- Per-character and per-channel look: avatars drawn from the trained look, accent colours, a sticker/emote set generated from it.
- Voice: local text-to-speech voice messages from characters; local speech-to-text dictation.

**Catching up and finding things**
- "While you were away" digest per channel, with a catch-up button.
- Search across live chats, memories and the notebook; a quick switcher (Cmd-K).
- Bookmarks/pins inside a chat; a story timeline per channel built from chapters.
- Shared gallery of images per character or channel, with favourites and keep.

**Writing and steering aids**
- Write my line (roadmap) and "continue for me" for a few turns.
- Scene HUD: a card showing current setting, outfit, scenarios and tones, with a state history you can scrub back through.
- Prompt budget view (roadmap), with suggestions to trim.
- Image redo with variations, edit-the-prompt-before-generating, reference-image upload.

**Character quality**
- Drift check: replay saved scenarios against a character after a sheet edit and flag changes in voice.
- Side-by-side compare of two sheet versions, or two models, on the same prompt.
- Relationship map between characters; free-form character state over time (trust, mood).

**World**
- In-world clock and calendar: birthdays, planned get-togethers and scripted events (a movie night) that drive autonomy.
- Anniversaries and dated callbacks ("a year since...") as motives for initiated messages.
- Per-channel notification rules and do-not-disturb schedules.

**Operations and safety**
- Engine status page: queue, model load, errors, a "needs a look" inbox.
- Backup restore UI and integrity checks.
- Passkey (WebAuthn) login, auto-lock after idle, quick-hide screen for the app switcher.
- Accessibility pass: font size, screen reader, keyboard shortcuts.
- Export a chat or chapters as Markdown; later, the novel path.

### What the owner wants most from the day-2 list

The theme is an **artificial ecosystem of characters the owner creates**: they have presence, make images for
themselves and each other, and live a shared calendar.

- **Presence and feel: huge.** Avatars, status, and characters generating images for themselves and for each other.
- **Catching up:** the "while you were away" digest, with the owner able to **edit and curate what is committed
  to memory** from what happened in their absence.
- **Continue button:** yes.
- **Scene HUD:** interesting. New idea from it: a **visual checklist of objects, wardrobe items and so on**,
  and an **interactive builder for image-generation prompts** from those items.
- **Character quality tools (drift check, compare):** "perfect".
- **In-world calendar:** really fun.

### Foundation notes (so projects 1 and 2 do not block these)

- Messages have an author type (user, character, system) and can carry image attachments from any author,
  so characters can send images to each other.
- Per-character runtime state (presence, mood) is a first-class record, even if empty at first.
- Outfit and worn items in the scene state are references to structured items, not free text, so a wardrobe
  checklist and a prompt builder can sit on them later.
- A "continue" action is a normal generation request with no user message.
- Characters reacting to each other's images needs a vision-capable model: the model question is no longer
  irrelevant.
- **Decision (owner):** memories from away-time chats are committed automatically, and the owner curates by
  selectively deleting what they dislike. (This replaced an earlier "proposed, pending review" idea and a
  two-tier recent/committed idea.)
- Refinements proposed by the assistant (not yet confirmed): auto-committed away-time memories carry an
  "unreviewed" marker and enter at low importance; the catch-up digest is the place to review and delete them;
  a deleted memory is remembered as dismissed so it is not re-learned from the same chat; character-only
  channels can be rewound to a point, with memories made after it flagged keep/drop (the existing rewind
  behaviour); memories can be about any participant (characters about each other), not just the user.
- Because memory is committed automatically, "unscanned" no longer holds an away-time chat from expiring; the
  hard ceiling on never-expiring chats matters less.
