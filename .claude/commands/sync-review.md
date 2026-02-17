# Sync Review

Process sync inbox and produce outbound packets.

## Steps

1. **Check inbox:** Glob `_shared/sync/inbox/*.md` for unprocessed packets.

   - If packets exist: read each one and process:
     - **Techniques:** Check against `_shared/TECHNIQUES.md`. Add new, update existing.
     - **System Improvements:** Evaluate and implement if applicable.
     - **Template Feedback:** Note for future template upstream.
     - Move processed packet to `_shared/sync/archive/`
   - If no packets: skip.

2. **Produce outbound:** Read `_shared/sync/staging.md`.

   - If non-empty: consolidate into a sync packet at `_shared/sync/outbox/YYYY-MM-DD-[topic].md`
     - Use the packet template from `SYNC_PROTOCOL.md`
     - Run the sanitization checklist — verify every item
     - Clear `staging.md` after successful production
   - If empty: skip.

3. **Report:** Summarize what was processed (inbox) and produced (outbox).
