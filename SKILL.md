---
name: speexx-exercise-automation
description: Automate completion of Speexx language learning exercises at portal.speexx.cn. Use this skill whenever the user wants to automatically complete Speexx exercises, finish packets, get 100% scores, or mentions Speexx /  Speexx  in combination with automation, auto-completion, or exercise solving. Triggers on phrases like "complete Speexx exercises", "auto finish Speexx", "Speexx 100%", "    Speexx   ", or any request involving the Speexx learning platform.
---

# Speexx Exercise Automation

Automate completion of Speexx (  ) language learning exercises at `https://portal.speexx.cn`.

## Prerequisites

- Chrome DevTools MCP server must be running and connected
- User must be logged into Speexx (the skill will navigate to the portal; if login is needed, ask the user to complete it manually)

## Execution Flow

### Phase 1: Gather Requirements

1. Navigate to `https://portal.speexx.cn/articles/list`
2. Ask the user to navigate to the target exercise page (the first exercise they want to complete).
3. Ask the user: "    ?" (How many packets do you need to complete?) Clarify:
   - Can be a number (e.g., 6 packets) or a range (e.g., packets 12-14).
   - Pronunciation packets do NOT count toward the packet count.
4. Ask the user: "      ?" (Do you need to complete pronunciation exercises?)
   - If YES: pronunciation packets within or immediately after the target range will be completed automatically.
   - If NO: all pronunciation packets will be skipped.
5. Confirm: "       ,     Next    ?" (CRITICAL: Unless you request otherwise, I will NOT complete level test exercises. Is this OK?)

**Do NOT proceed to Phase 2 until all questions are answered.**

### Phase 2: Complete Exercises

For each exercise in each packet, follow this pattern:

1. **Identify the exercise type** — take a snapshot and read the description text and DOM structure.
2. **Execute the answer-fill workflow:**
   a. Click "Correction" to see current score
   b. Click the answer reveal button ("    " / "Press space or enter to view answers")
   c. Click "Repeat" to reset the exercise
   d. Fill in all correct answers (see `references/exercise-patterns.md` for type-specific code)
   e. Click "Correction" again to update the score
3. **Verify 100% score:**
   - If score is 100/100 → click "Next"
   - If score is NOT 100/100 → repeat step 2 from (b) through (e). The previous attempt may have left some answers unfilled or incorrectly placed. Re-read the revealed answers, click Repeat, fill again, and re-check.
   - Continue retrying until 100/100 is achieved.
4. **Exception:** Video and pronunciation exercises skip the 100% requirement (see type-specific rules).

### Phase 3: Continue Until Done

- After completing a packet, a "Great job" dialog appears — click "    " (Continue learning) to proceed to the next packet.
- Repeat Phase 2 for each packet until the requested count is reached.
- Track progress: each **standard packet** (non-pronunciation) counts as one section. Stop when the target number is met.

**Pronunciation packet handling:** Depends on the user's answer in Phase 1 step 4:
- **If user said NO:** skip all pronunciation packets (click "    " to bypass).
- **If user said YES:** pronunciation packets are completed automatically only when they fall within or immediately after the target range:
  - **Within range:** e.g., target is packets 3-6, and a pronunciation packet sits between packet 3 and packet 4 → complete it.
  - **Immediately after:** e.g., target is packets 3-6, and a pronunciation packet appears right after packet 6 → complete it.
  - **Otherwise skip:** pronunciation packets before the range or unrelated to the target range are skipped (click "    " to bypass).

## Exercise Type Overview

| Type | Description | 100% Required? |
|------|-------------|----------------|
| `type-drag-drop` | jQuery UI drag words into blanks | Yes |
| `type-scrambled-table` | Swap scrambled cells to match | Yes |
| Checkbox/Radio | Select correct options | Yes |
| Text fill-in | Type words into text inputs | Yes |
| Scrambled sentences | Reorder words into correct sentences | Yes |
| Video | Watch video | No — click Next immediately |
| Pronunciation | Speak into microphone | No — trigger mic, wait for "you can do better", click Next |

**Critical rule for non-pronunciation exercises:** Always achieve 100/100 before clicking Next.

## Key Automation Principle

Simply moving DOM elements is NOT sufficient for jQuery UI exercises. After placing drag items, you MUST invoke the internal jQuery UI droppable callback to register the drop event:

```javascript
const dropInstance = $ph.data('uiDroppable');
if (dropInstance && dropInstance.options && dropInstance.options.drop) {
  const fakeEvent = $.Event('drop');
  dropInstance.options.drop.call($ph[0], fakeEvent, {
    draggable: $drag, helper: $drag.clone(),
    position: $drag.position(), offset: $drag.offset()
  });
}
```

For sortable (scrambled sentences), trigger `sortupdate` after reordering:
```javascript
$(sentence).trigger('sortupdate');
```

For detailed code patterns for each exercise type, read `references/exercise-patterns.md`.

## Important Rules

1. **Always click Correction first** on each new exercise to reveal the answer button, then click the answer reveal button (    / "Press space or enter to view answers"), then click Repeat, then fill in answers using the revealed correct answers.
2. **After Repeat, DOM element IDs change** — always re-query elements rather than caching IDs.
3. **The Correction/Next button UIDs change** after page interactions — always take a fresh snapshot before clicking.
4. **For checkbox exercises after Repeat**, the answers are reset — re-check them via JavaScript dispatching `change` and `click` events.
5. **For drag-drop after Repeat**, the drag items return to the word bank — re-run the complete placement script.
6. **Pronunciation exercises** have 11 units per packet typically. Workflow: first check if "You can do better!" is already visible in the bottom-left corner. If YES → click Next directly. If NO → click mic button → click mic again (to stop) → verify "You can do better!" appears → click Next. No 100% score needed. **Only complete pronunciation packets if the user said YES in Phase 1 step 4, and even then, only those sandwiched within the target range or immediately following the last target packet; skip all others.**
7. **Never complete level test (    ) exercises** unless the user explicitly requests it.
