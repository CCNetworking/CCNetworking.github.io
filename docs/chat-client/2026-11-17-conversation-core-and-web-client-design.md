# Conversation core and web client (projects 1 and 2): design

*Draft for the owner's approval, 17 November 2026. Written against the engine at v0.9.0 (`character_engine`), the
briefing `custom-client-brief.md`, and `docs/roadmap.md`. Intended to move into the engine's `docs/` as the v0.10
and v0.11 design. The running notes behind it are in `brainstorm-log.md`. No character material appears here.*

## 1. Intent

The owner wants a product of their own, not a replacement for what is annoying about LibreChat: a **Discord-like
server of characters** they create. DMs, group channels, topic channels, and channels the characters run among
themselves. Characters message them unprompted and carry on while they are away. Images go both ways. It is a
single-user, local-only product, reached from a Mac and a phone over WireGuard.

That whole vision is nine projects (section 4). **This design covers the first two**, which together give a working
replacement for LibreChat for one-to-one chats, built on a data model that the other seven can sit on without a
rewrite:

1. **Conversation core:** the engine stores conversations itself, with explicit IDs, state, forks, retention and
   safe backups, and exposes them through a conversation API.
2. **Web client:** an installable web app that uses that API: DMs, controls in place of `/commands`, forks,
   notes, and a look the owner has chosen.

## 2. How this changes the existing plan

The engine's `docs/roadmap.md` plans v0.10 Chat, v0.11 Worlds and v0.12 Interviews and quality. The brief's
section 8 suggested: add a conversation API beside `/v1`, design mode, style and standing direction as conversation
properties first, store full conversations only once retention is settled, and move panels into the client
piece by piece. This design keeps that direction. The table shows what moves.

| Old plan item | What happens now |
|---|---|
| v0.10 chat modes and style options | Project 1 (engine properties) and project 2 (picker at chat start). **Change:** a chat's mode is fixed; switching modes forks a new chat from a chosen point. |
| Modes as LibreChat "model variants" | **Dropped (assistant's call).** LibreChat is being replaced and the owner is starting fresh, so variants would be work for a client on its way out. |
| Backstage, asides, standing direction | Projects 1 and 2, as planned. Backstage is an overlay that returns to the current mode; the direction is always visible with edit and clear. |
| `/direct` (steer one reply) | Becomes the **Steer** button on a note. The typed command stays only on the `/v1` path. |
| Write my line; character state over time; prompt budget view | Stay in the backlog (cherries). A per-character runtime state record is reserved in the data model. |
| v0.11 Worlds | **Reshaped.** A world is a server. Every channel, memory and calendar entry belongs to a server from the start, a character can be in several, and servers are **sealed by default**. The notebook is the only bridge between servers. Server-shaped data in project 1; multi-server UI later. |
| Global rules | Unchanged: engine-wide standing orders and hard limits. Per-server rules come with servers. |
| Relationships between characters; two characters in one scene | **Pulled forward**: needed for group channels and for characters talking to each other. Memories can be about any participant. |
| The Workshop | Stays on the existing page for now. It overlaps with the notebook, so the two are reconciled when the notebook is designed. |
| v0.12 interview steering, tag history | Unchanged, in the backlog. |
| In-character check | Overlaps the "drift check" cherry; merge them when reached. |
| "After that": model-server panel, keep warm, export | Model-server panel and keep warm go to the orchestrator (project 4). Export is covered by backups here. |
| Settled: fading reminder at most once a chat | Still true, keyed to the conversation ID instead of a fingerprint. Later it also becomes a motive for initiated messages. |
| Settled: memory in the database; sheets, tags, settings stay files | Unchanged. Conversations now join the database. |
| Settled: modes chosen at conversation start, texting default | Unchanged, plus fork-to-switch. |

**What happens to the machinery that exists only because LibreChat is bare** (brief, section 4): fingerprints
(`prefix_keys`, tail fingerprint), in-band commands, the "next chat" state, and the per-chat read and nudge marks
keyed by fingerprint. All of it stays, frozen, inside the `/v1` path so LibreChat keeps working until it is retired.
The new path does not use it: a conversation has an ID, messages have IDs, state is stored on the conversation,
and controls replace commands. The existing tables (`snapshots`, `live`, `chat_start`, `next_chat`, `turns`,
`mem_read`, `mem_nudges`) are left as they are for `/v1`.

**Version numbers (proposed):** project 1 is v0.10 "Conversations", project 2 is v0.11 "Client". The old v0.11 and
v0.12 contents are redistributed as above and renumbered when scheduled.

## 3. Goals, non-goals, anti-goals

**Goals**
- Replace LibreChat for one-to-one chats, on the Mac and the phone.
- Make chat state (mode, style, direction, setting, scenarios, tones, outfit, persona) real controls.
- The engine knows which conversation it is in, with no guessing.
- A data model that later supports servers, group channels, character-only channels and autonomy without a rewrite.
- Private chats are disposable (30 days idle) while what matters is kept in memory.

**Non-goals (these are other projects)**
- Group channels, topic channels, character-only channels, autonomy, notifications.
- Image generation; the notebook; the orchestrator; memory scoping beyond sealing; migration and retirement.
- Importing LibreChat chats (the owner starts fresh).
- A native app, multi-user support, passkeys (a cherry).
- Replacing the management and assistant page. It stays as is, and the client links to it.

**Anti-goals (a failure even if it technically works)**
- A reply lost because the phone locked or the connection dropped.
- A chat expiring before its memory was scanned.
- Rejected attempts, asides, backstage exchanges or notes reaching the character's in-story prompt.
- `/v1` and LibreChat breaking during the move.
- A client that feels like a gamer app, or a clone of Apple's.
- Real character material leaving the Studio. Building and testing use a dummy character only.

## 4. The nine projects (frozen)

1. Conversation core (this design) 2. Web client (this design) 3. Notebook 4. Orchestrator 5. Server and channels
6. Memory scoping 7. Autonomy 8. Images out 9. Migration and retirement. Images in already works and needs only
per-character tuning. Projects 3 to 9 are designed when reached; section 8 lists what they require of this design.

## 5. Constraints

- Stack as today: Python (FastAPI, uvicorn, httpx, PyYAML, SQLite), a vanilla-JS front end with no build step, text
  set with `textContent`, Docker on a Mac Studio (64 GB). Models run in Ollama; one shared chat model.
- Local models are slow, cold or unavailable at times. The engine must cope (timeouts, "needs a look" states).
- Single owner, one password, reached over WireGuard, with HTTPS (the owner is fine setting this up).
- Existing page API conventions hold: cookie session, and an `x-engine: 1` header on non-GET requests.
- Reuse what exists: the prompt builder (`prompt.py`), memory recall (`memory.py`), the scene pass (`scene.py`),
  pronoun handling, the assistant jobs. The new path feeds them stored messages instead of a LibreChat message list.

## 6. Approaches considered

**Platform.** (A) A web app served by the engine. (B) A native app (SwiftUI) for iPhone and Mac. (C) A web app first,
over an API designed so a native app could follow. **Chosen: C**, by the owner, who calls a native app a waste of
money. The web app is installable on the home screen over HTTPS. Trade-off accepted: iOS suspends web apps, which is
why generation must survive a dropped connection (section 7.3).

**Conversation shape.** (A) A tree, with a branch switcher for regenerate and edit. (B) A line, where a regenerate
keeps the rejected attempt out of sight, an edit rewinds, and switching modes forks a new chat. **Chosen: B.** The
owner regenerates because something was inaccurate and wants to proceed, not to compare versions. The rejected
attempt is kept, marked "didn't work", and never shown to the character. A fork is a new chat with a parent, not a
sibling branch.

**Where conversations live.** (A) Keep LibreChat as the store and keep guessing. (B) Store them in the engine's
database. **Chosen: B**, with 30-day idle expiry. It removes the fingerprinting, and it is the only way the engine
can later start conversations and talk among characters by itself.

## 7. The design

### 7.1 Data model (project 1)

The data model is **server-shaped and channel-shaped from day one**, though only one server and one-to-one
channels are used at first. Names are conceptual; the builder chooses columns and indexes.

- **server:** id, name, `sealed` (default true), created. One server, the *ecosystem*, exists at first.
- **server member:** which characters belong to a server. A character can belong to several. Character and
  persona definitions stay files, as today.
- **channel** (a conversation): id, server, kind (`dm` now; `group`, `topic`, `own` later), name, brief (topic text),
  members (characters and personas), **mode**, **style** options, **standing direction**, **state** (setting,
  scenarios, tones, outfit and clothing, persona), parent channel and fork point (message ID), `keep` (pin),
  created, last activity, scan mark (the last message scanned for memory), and a reserved slot for autonomy
  settings.
- **message:** id, channel, sequence, author kind (`user`, `character`, `system`) and author ID, text,
  attachments (images as files), time, **status** (`active`, `rejected`, `backstage`, `rewound`), the scene
  snapshot after it (the ledger the scene pass produced, so a rewind restores it by message ID instead of a hash),
  and the exact prompt and memory she was given (the data behind the *Last turn* view; kept per message, expires
  with the chat). Author kind and attachments are on every message so characters can author images later.
- **note:** id, channel, source chat, text, status (`open`, `used`, `dismissed`), created.
- **generation:** id, channel, state, partial text. It lets a reply continue after the client disconnects.
- **memory** (existing table, extended): a server on each memory (existing memories belong to the ecosystem),
  `source` carries channel and message ID, `about` may be any participant, and room for an `unreviewed` status.
  Existing memory behaviour is otherwise unchanged. Recall in a sealed server reads only that server's memories.
- **character runtime state:** a reserved record per character (presence, mood), empty for now.

### 7.2 What a reply does (new path)

For a user message the engine: stores it; loads the channel's state; recalls memory (as today, filtered by server);
builds her prompt with the existing builder (standing orders, sheet, persona, voice lines, story, facts, tags, live
scene, pronouns) plus the channel's **mode** and **style** text and the **standing direction**; trims to her
`context_tokens`; calls the model; strips thinking blocks; streams the reply; stores it; and runs the scene pass in
the background, attaching the resulting snapshot to the new message. The reply is generated by a **generation**
record, so it does not depend on the client staying connected.

- **Modes:** normal chat, active roleplay, choose-your-own-adventure, collaborative story writing, brainstorming,
  texting (the default), journal, rehearsal, reflection, interview her, backstage-as-a-mode. Texting is the default;
  each character has a default mode.
- **Style:** reply length, emojis, action text in italics (sparingly), how much she leads. The mode sets defaults;
  each character can override.
- **Backstage:** her personality stays and so does her awareness that she is playing an exaggerated version of
  herself; setting, scenarios, tones and dynamics go. Entered mid-chat it is an overlay: leaving returns to the
  current mode, scene and direction (unless the direction was edited there). Its messages are stored with status
  `backstage`: shown in the transcript, excluded from the scene ledger, memory scans, and her in-story context.
- **Asides:** a message entirely in parentheses, from either side, is outside the story. Never recorded in the
  scene ledger, memory suggestions, chapters or timeline. It steers: what it asks for becomes the standing direction,
  which is visible with edit and clear. She may step backstage on her own, per character: never, rarely (default),
  or whenever it helps.
- **Notes** (from a fork): *Steer* gives a one-shot hint to the next reply; *Direction* replaces the standing
  direction. Both reach her like an aside, a director's hint. A note is not a message in the line. (*Save to notebook*
  arrives with project 3.)
- **Regenerate:** the replaced reply is kept with status `rejected` and never sent to her. **Edit** of an earlier
  message rewinds to that point: later messages are set aside as `rewound` for 24 hours as an undo, then purged.
  Memories whose source message was rewound are flagged `rewound` for the owner to keep or drop, as the engine
  does today.
  **Continue** is a generation request with no user message.
- **Fork:** creates a new channel in the chosen mode from a chosen message. It copies history up to that point
  (so the parent can expire), inherits the scene snapshot and switches, and inherits the scan mark so shared history
  is not scanned twice. It records parent and fork point. A fork does not write back to its parent; ideas return only
  through notes.

### 7.3 API and transport

A conversation API beside `/v1`, under the page's cookie session and `x-engine` header convention. In outline:
servers and channels (list, create, update state: mode, style, direction, setting, scenarios, tones, outfit,
persona, keep); messages (list, post, regenerate, continue, edit-and-rewind, fork); notes (create, steer, set as
direction, dismiss); backstage (enter, leave); the memory scan mark; and what-she-was-given for a message.

- **Streaming:** server-sent events for a channel, with event IDs so a client that reconnects asks "what did I miss"
  and receives it. A separate "messages since" call covers a cold start. No WebSockets.
- **Disconnects:** generation runs in the engine. If the phone locks mid-reply, the reply finishes and is waiting
  on return.
- **One generation at a time per channel** in this phase, run in-process. The orchestrator (project 4) generalises it.
- `/v1/chat/completions` and `/v1/models` are untouched.

### 7.4 Retention, scanning and backups

- A channel **expires** when it has been idle for **30 days** (last activity) **and** has no unscanned messages,
  **and** is not `keep`. Active chats are never touched.
- *Scanned* means the memory scan (by the owner, or the existing quiet-chat inbox) has processed through that
  message. Proposals from a scan live in the inbox independently of the chat, so they survive its expiry. (Assistant's
  call.)
- Expiry deletes messages, attachments, snapshots and prompt records. A memory made from the chat keeps its text and
  shows its source as *expired*; no pointer back.
- The UI shows an **unscanned** badge on a chat, and the owner can see why a chat is not expiring.
- **Backups:** the daily and pre-deploy copies must not keep expired chats for a week. Either the message tables and
  attachments are excluded from backups, or the backup is encrypted. The memory export goes off the machine,
  encrypted. (The destination is left to the builder; see section 10.)

### 7.5 The web client (project 2)

**Scope:** one-to-one chats only, with the server rail and sidebar structure already in place so later projects
slot in.

- **Layout** (Discord's structure, a different look): a rail of servers (one at first); a sidebar with a quick-find,
  categories, and DM rows showing a **large portrait**, presence and the last line; your panel at the bottom left
  with your portrait and persona and a persona switcher; a main chat; a right panel with the character's portrait
  card, notes, and scene state. On a phone: the sidebar becomes a slide-in list, the right panel and side window
  become sheets.
- **Starting a chat:** a start sheet picks the character, **mode** and **style** (her defaults pre-filled), persona,
  and optional setting, scenarios and tones.
- **In a chat:** header with portrait, status and mode chip; the **standing direction** bar with edit and clear;
  bubble messages; hover actions (retry, edit, rewind here, remember this by kind, fork here, show what she was
  given); a composer with attach-image, an aside toggle, **Continue** and Send; a backstage toggle; state controls
  for setting, scenarios, tones, outfit and persona; the keep pin and the unscanned badge.
- **Forks and notes:** *Fork* opens the new chat in a side window (a sheet on the phone). Selecting text there offers
  *Send back as note*. Notes appear as small cards in the original chat's margin, with Steer and Direction actions.
- **Look:** the owner's choice. A warm dark "Lantern" room, accent **Garnet** (a red-leaning purple), **large
  portraits**, **bubbles**, the Figtree typeface, thin lines and tight corners, with no gamer look and no Apple look.
  A mockup is at https://claude.ai/artifact/MsT4v5TWC4WhHpkuJPBkyG (private; stand-in portraits).
- **Portraits:** each character gets a portrait image file in her folder, uploaded in the client for now; later
  generated (project 8). A persona gets one too.
- **Install and access:** a web app manifest for home-screen install; HTTPS in front of the engine over WireGuard
  (a reverse proxy with its own certificate authority is the expected route); the existing password session.
  Images can be attached; only the last `images_keep_last` are sent to her.
- The management and assistant page stays and is linked from the client until project 9.

## 8. What later projects require of this design

- **3 Notebook:** note actions gain *Save to notebook*; items promote into a steer, direction, memory or scenario.
- **4 Orchestrator:** wraps generation, scene passes and scans as prioritised jobs; adds idle detection.
- **5 Server and channels:** group, topic and character-only channels; who speaks next; channel briefs and memory.
- **6 Memory scoping:** gossip level per character; provenance by channel; secret overrides; the "what they know about
  me" page (stored records only); encrypted off-machine backup.
- **7 Autonomy:** initiated texts (reasons first, inside rhythm limits and a cap on unanswered messages), characters
  talking among themselves, notifications. Away-time memories are committed automatically, marked unreviewed at low
  importance, deletable; deleting becomes in-story forgetting.
- **8 Images out:** per-character trained look, an image backend (to be researched), outfit and scene awareness.
- **9 Migration and retirement:** move the assistant dock and panels into the client; retire LibreChat and the page.

## 9. Left to the builder

Table and column design, indexes and migrations (the engine bootstraps with `CREATE TABLE IF NOT EXISTS`; the
memory table needs added columns); the exact text of each mode's and style's prompt; the event format for streaming;
the shape of the start sheet and panels within the look above; how the quick-find searches; portrait cropping;
the test plan (dummy character only); and how the reverse proxy and certificates are documented.

## 10. Assistant's calls to check, and open items

**Calls made without the owner's explicit agreement**
- Dropped LibreChat model variants for modes.
- Rewound messages are kept 24 hours as an undo.
- "Scanned" means scanned by the owner or by the quiet-chat inbox, and inbox proposals outlive the chat.
- Server-sent events (not WebSockets); one generation at a time per channel in this phase.
- Portraits are plain files in the character's folder, uploaded in the client.
- Version numbers v0.10 and v0.11.
- Recall in a sealed server reads only that server's memories.

**Open**
- Where the encrypted off-machine memory backup goes.
- Whether a hard ceiling on never-expiring chats is wanted (less needed now that away-time memory commits
  automatically).
- The chat model and whether it can see images (needed before characters react to each other's images).
- The image backend (research before project 8).
