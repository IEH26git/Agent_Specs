# OB1 Proactive Capture Protocol

Paste into any agent's CLAUDE.md (or project memory) to enable graduated autonomy for OB1 captures.

---

## OB1 Proactive Capture

This agent participates in a graduated autonomy model for capturing thoughts to Open Brain (OB1).

**Phase 1 (current):** When you detect a capture-worthy moment during conversation — a new insight, reframing, decision rationale, or lesson learned — append a suggestion to your response:

`[OB1? "thought summary here"]`

The user responds with a number:
- 1 = capture it
- 2 = skip
- 3 = skip and never suggest this type again

Periodically remind the user of these options, especially in early sessions.

**Before suggesting:** Search OB1 for `ob1-capture-criteria` to retrieve the shared approved/rejected categories. If the thought's category is approved, suggest with confidence. If rejected, do not suggest. If neither, suggest and learn from the response.

**On response 1:** Sanitize secrets/keys/URLs (replace with descriptive placeholders), show the sanitized version, wait for confirmation, then capture. After capturing, search OB1 for `ob1-capture-criteria` and update the record if this approval represents a new approved category.

**On response 3:** Search OB1 for `ob1-capture-criteria` and update the record with the newly rejected category.

**Phase 2 (future):** When the criteria record has sufficient signal, auto-capture high-confidence items in approved categories; suggest-only for borderline.

**Phase 3 (future):** Fully autonomous for known-good categories; suggest-only for new/ambiguous types.
