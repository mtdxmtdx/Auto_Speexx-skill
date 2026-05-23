# Speexx Exercise Patterns — Code Reference

Detailed JavaScript automation patterns for each exercise type on Speexx.

---

## General Workflow (Retry Loop)

For every non-video, non-pronunciation exercise, execute this loop until 100/100:

```
Correction (check score)
  → Answer Reveal (see correct answers)
  → Repeat (reset exercise)
  → Fill Answers (type-specific JS pattern below)
  → Correction (verify score)
  → If NOT 100: repeat from Answer Reveal step
  → If 100: click Next
```

**Why retries are needed:** The first fill attempt may miss answers or trigger UI callbacks incompletely. Re-reading the revealed answers and re-filling ensures correctness.

---

## 1. Drag-Drop Fill-in-Blank (`type-drag-drop`)

**DOM structure:** `.drag-drop.ui-draggable` items in a word bank, `.drag-drop-placeholder.ui-droppable` as drop zones within `.item` containers.

**Workflow:**
1. Click Correction → answer reveal → Repeat
2. Identify correct answers from the revealed state (snapshot before Repeat)
3. After Repeat, re-place all drag items

**Placement code:**
```javascript
const $ = window.$ || window.jQuery;
const answers = ['word1', 'word2', ...]; // correct answer for each placeholder

$('.exercise-items .item').each(function(i) {
  const $ph = $(this).find('.drag-drop-placeholder');
  if (!$ph.length) return;

  const answer = answers[i];
  const $drag = $('.drag-drop.ui-draggable').filter(function() {
    return $(this).text().trim() === answer;
  });
  if (!$drag.length) return;

  $drag.detach().appendTo($ph);

  const dropInstance = $ph.data('uiDroppable');
  if (dropInstance && dropInstance.options && dropInstance.options.drop) {
    const fakeEvent = $.Event('drop');
    dropInstance.options.drop.call($ph[0], fakeEvent, {
      draggable: $drag,
      helper: $drag.clone(),
      position: $drag.position(),
      offset: $drag.offset()
    });
  }
});
```

---

## 2. Scrambled Table (`type-scrambled-table`)

**DOM structure:** `.scrambled-cell` elements (both `ui-draggable` and `ui-droppable`) inside `.scrambled-cell-container` within `.item` containers.

**Workflow:**
1. Click Correction → answer reveal → Repeat
2. Identify correct cell-to-item mapping
3. After Repeat, swap cells between containers

**Swap code:**
```javascript
// Mapping: item index → correct cell text
const correctMapping = {
  0: 'text A',
  1: 'text B',
  2: 'text C',
  // ...
};

const items = document.querySelectorAll('.exercise-items .item');
const cellMap = {};
items.forEach((item, idx) => {
  const cell = item.querySelector('.scrambled-cell');
  if (cell) cellMap[cell.textContent.trim()] = { cell, currentItem: idx };
});

for (let i = 0; i < Object.keys(correctMapping).length; i++) {
  const targetText = correctMapping[i];
  const info = cellMap[targetText];
  if (!info || info.currentItem === i) continue;

  const targetContainer = items[i].querySelector('.scrambled-cell-container');
  const sourceContainer = items[info.currentItem].querySelector('.scrambled-cell-container');
  const currentCellInTarget = targetContainer.querySelector('.scrambled-cell');

  targetContainer.appendChild(info.cell);
  if (currentCellInTarget && currentCellInTarget !== info.cell) {
    sourceContainer.appendChild(currentCellInTarget);
  }

  // Update tracking
  if (currentCellInTarget && cellMap[currentCellInTarget.textContent.trim()]) {
    cellMap[currentCellInTarget.textContent.trim()].currentItem = info.currentItem;
  }
  cellMap[targetText].currentItem = i;
}
```

---

## 3. Checkbox / Radio Selection

**DOM structure:** Standard `<input type="checkbox">` or `<input type="radio">` within `<label>` elements.

**Workflow:**
1. Click Correction → answer reveal → note which options are checked
2. Click Repeat → all options reset
3. Re-check the correct options via JavaScript

**Selection code:**
```javascript
const correctAnswers = ['option text 1', 'option text 2', ...];

document.querySelectorAll('input[type="checkbox"], input[type="radio"]').forEach(cb => {
  const label = cb.closest('label');
  const text = label ? label.textContent.trim() : '';
  if (correctAnswers.includes(text)) {
    cb.checked = true;
    cb.dispatchEvent(new Event('change', { bubbles: true }));
    cb.dispatchEvent(new Event('click', { bubbles: true }));
  }
});
```

---

## 4. Text Fill-in

**DOM structure:** `<input type="text">` or `<textbox>` elements within sentence text.

**Workflow:**
1. Note word bank (usually displayed at top: "word1 · word2 · word3 · ...")
2. Fill each textbox with the correct word from the word bank
3. If unsure, click Correction → answer reveal to see correct answers

**Fill code:**
```javascript
const answers = ['word1', 'word2', ...];
const textboxes = document.querySelectorAll('input[type="text"], input:not([type])');

textboxes.forEach((tb, i) => {
  if (i < answers.length) {
    tb.value = answers[i];
    tb.dispatchEvent(new Event('input', { bubbles: true }));
    tb.dispatchEvent(new Event('change', { bubbles: true }));
  }
});
```

---

## 5. Scrambled Sentences

**DOM structure:** `.scrambled-sentence.ui-sortable` containers with `.scrambled-block` child elements. jQuery UI sortable.

**Workflow:**
1. Click Correction → answer reveal (note: sometimes no reveal button; deduce order from context)
2. If answer button available, click it to see correct order
3. If no answer button, deduce correct sentence order from the words
4. Reorder blocks within each sentence container

**Reorder code:**
```javascript
const correctOrders = [
  ['Word1', 'Word2', 'Word3', '.'],
  ['WordA', 'WordB', 'WordC', '.'],
  // ...
];

const sentences = document.querySelectorAll('.scrambled-sentence');
const $ = window.$ || window.jQuery;

sentences.forEach((sentence, i) => {
  if (i >= correctOrders.length) return;
  const blocks = Array.from(sentence.querySelectorAll('.scrambled-block'));
  const textToBlock = {};
  blocks.forEach(b => { textToBlock[b.textContent.trim()] = b; });

  correctOrders[i].forEach(word => {
    const block = textToBlock[word];
    if (block) sentence.appendChild(block);
  });

  if ($ && $(sentence).data('uiSortable')) {
    $(sentence).trigger('sortupdate');
  }
});
```

**Note:** Some blocks may contain multi-word phrases (e.g., "immediately to" or "backed up"). Always check actual block `.textContent` before mapping. Punctuation (`.` `,`) is always its own block.

---

## 6. Video Exercises

**Just click Next.** No interaction needed. No 100% requirement.

Often marked with "Watch the video and keep on studying with the exercises." Video exercises typically appear as Exercise 1 of a packet, with subsequent exercises (2, 3, 4) being the actual practice.

---

## 7. Pronunciation Exercises

**URL pattern:** `/pronunciation-packet/` instead of `/standard-packet/`.

**Workflow (per unit):**
1. Take snapshot → check if "You can do better!" (English) or "     ！" (Chinese) is already visible in the bottom-left corner
2. **If "You can do better!" is already visible** → click Next directly, skip to next unit
3. **If NOT visible** → find mic button ("       " / "Press space or enter to start recording") → click it → wait briefly → click it again (to stop recording)
4. Take snapshot → verify "You can do better!" appears. If not, click mic one more time.
5. Click Next

**No 100% requirement for pronunciation.**

Each pronunciation packet typically has 11 units. Each unit follows this same pattern. The key optimization: some units may already show "You can do better!" from a previous session — in that case, just click Next without touching the mic.

---

## Common Pitfalls

1. **Stale UIDs:** Every DOM change can invalidate button UIDs. Always take a fresh snapshot before clicking.
2. **Correction button timeout:** If Correction click times out, take a fresh snapshot and try the new UID.
3. **Repeat resets everything:** After Repeat, drag items return to word bank and IDs change. Never cache selectors across Repeat.
4. **Missing answer button:** Some exercise types don't show an answer reveal button. In those cases, deduce answers from the word bank or context clues.
5. **"Great job" dialog:** After completing all non-video exercises in a packet, a dialog appears. Click "    " to proceed to the next packet.
6. **Punctuation in scrambled sentences:** Periods, commas are always separate blocks — include them as separate entries in the correct order array.
