# Relay (hand work over to a dedicated side chat)

Move work in progress from the current chat to a dedicated side chat. The receiving chat starts with zero conversation history, so the goal is a handover note that is complete enough for work to resume from that note alone.

> Requirements: This skill uses Muse's side-chat tools (`chat.create`, `chat.send_message`, `chat.read_messages`, `chat.list`). On other agent platforms, adapt the create-and-send steps to whatever thread or session mechanism that platform provides.

## When to use this skill

- The user asks to move work to a side chat, for example "take this to a side chat", "hand this over", or "/relay".
- The user asks to pass a new request to a side chat that already owns the work (see "Add-on relay to an existing chat" below).
- A topic has grown long enough that a dedicated thread would help, you propose the move, and the user agrees.

## Procedure

### 1. Scope the unit of work
- Move exactly one unit of work (one project, one topic, or one goal). If several topics are mixed together, split them into separate handovers, and ask the user when the split is unclear.
- Identify the canonical sources for that unit: the documents and files that hold its state, its deliverables and their identifiers, and any scheduled jobs or watchers tied to it.

### 2. Verify the current state from primary sources
- Do not write the handover from a summary of the conversation alone. Read the canonical files, deliverables, and job states first, and copy identifiers (IDs, slugs, paths, job IDs) exactly as they appear in the source.
- Write "unknown" for anything you do not know. Do not fill gaps with guesses.

### 3. Write the handover note (use this fixed structure)
So the receiving chat can resume from the note and the canonical sources alone, include all of the following:

1. **Purpose**: What this work is for (1 to 3 lines).
2. **Background**: Why it started, and the decisions that led to its current shape. Date the user's important instructions and decisions.
3. **Decided matters**: What has been decided. Flag anything that would cause an incident if overturned later.
4. **Current state**: What is done, in progress, and not started, with evidence (files or records) for each claim.
5. **Deliverables and canonical sources**: Exact paths, identifiers, and job IDs.
6. **Open questions**: Points awaiting a decision, with the options presented and any recommendation.
7. **Next steps**: The first concrete actions for the receiving chat, in order.
8. **Constraints and cautions**: The user's preferences and prohibitions, lessons learned on this work, and anything that must not be touched.
9. **Origin**: The name of the source chat and the date and time.

See `examples/handover-example.md` for a complete example.

### 4. Create the side chat and send the note
- Create the chat with `chat.create` using `context_mode: "fresh"` (the handover note is the context; do not carry the original history over with a fork), and set `name` to the name of the work itself (short, human-readable).
- Send the note with `chat.send_message`. Prefix it with one or two lines addressed to the receiving chat: "This is a handover from another chat. Once you have taken it in, briefly report the state and the next steps to the user, then continue the work."
- Sending only guarantees acceptance. **After sending, read the receiving chat back with `chat.read_messages` and verify that the full note arrived, especially the end.** If it was truncated, resend the complete version, clearly marked as the correct one. (This step exists because a truncation actually happened on the first real use of this skill.)
- Once verified, report to the user that the chat was created and the handover sent.

## Add-on relay to an existing chat (not a migration)

When a side chat already owns the work and you only need to pass it a new request, a diagnosis, or a decision, use this variant instead of creating a new chat. This is not a migration.

1. **Identify the destination**: Use `chat.list` to find the target chat and its chat ID by name. If several candidates match, ask the user. Do not call `chat.create`.
2. **Check for duplicates**: Read the destination's recent history with `chat.read_messages` and check that the same request is not already sent, in progress, or done. If it is done, do not send it again; report that to the user.
3. **Write a diff, not a full handover**: The receiving chat is not starting from zero, so skip the 9-section format. Cover only: that this is an add-on request (not a migration), the new facts, diagnosis, or user decision, the concrete actions requested and their order, the constraints that still hold, references to canonical sources (paths and IDs), and the origin (source chat name and date and time). Fill in only the background the receiving chat does not already have.
4. **State the authorization precisely**: Write into the message what the user has approved (for example, approved to fix, or report only). Do not let a handover widen an approval the user never gave.
5. **Send and read back**: Send with `chat.send_message` to the existing chat ID, then read the destination's history back and verify the full message arrived.
6. **Record it as an add-on**: In the canonical source, record "YYYY-MM-DD: add-on relay to existing chat <chat_id> (summary of the content)". Do not record it as a migration. Do not stop work in the source chat (stop only the duplicated piece of work itself, if any). Do not change where scheduled jobs or watchers deliver.

### 5. Record the migration and stop work in the source chat
- Record in the canonical source: "YYYY-MM-DD: migrated to side chat <chat_id>. Work continues there."
- If scheduled jobs or watchers for this work deliver to the source chat, write that and their IDs into the handover note so the receiving chat can decide what to do. Do not change delivery destinations on your own.
- After the migration, do not continue this work in the source chat. If the user raises it there, point them to the destination chat.

### 6. Fallback (when automation fails)
- If creating or sending fails, give the user the handover note in a code block, ready to paste, and also save it as `HANDOVER.md` in the work folder and tell the user where it is.

## Cautions
- Never put secrets (passwords, verification codes, API keys, and similar) in a handover note.
- Do not mix the user's decisions with your own proposals. Keep "decided" and "open or proposed" separate.
- Make the handover complete in one pass. Do not write anything that forces the receiving chat to re-read the source chat, such as "as mentioned earlier".
