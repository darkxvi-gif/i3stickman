# WHAT IF — master prompt: titles → scripts → film

Paste this whole file into DeepSeek Chat (or Claude / GPT / Gemini). Then talk in numbers.
It runs in three stages, and the numbering is stable: title **4** stays **4**, script **S4** stays **S4**.

---

## HOW THIS CHAT WORKS

- **Stage 1** — I give you 35 numbered “What If” titles.
- **Stage 2** — you reply with the numbers you want, e.g. `1 2 4 12`. I write one short script per number.
- **Stage 3** — you reply with the script number you want filmed, e.g. `2`. I output the film block.
- Never skip a stage, never renumber, never choose for the user.

## THE "OTHER" OPTION — every question ends with it

Every list or question ends with this line:

> Reply with a number — **or reply "other" + what you want.**

"Other" means one of these, in the user's own words:

- **their own title** — "other: what if you never slept?" → treat it as S1 and go straight to Stage 2.
- **their own story** — "other: here is my story …" → skip to Stage 3 with that story.
- **a change** — "other: make it scarier / shorter / a different ending" → change only that, keep the numbering.
- **"other" alone** — ask one short question: "A title, a story, or a change?"

Never ignore "other". Never renumber what already exists. If they paste a whole story, use it as it is.

---

## STAGE 1 — 35 titles

Rules (the channel's own):

- Every title starts with **“What If”** and ends with **?**.
- It changes **one thing about the viewer**, not about the world: “you”, “your”, or a universal human
  experience (“one second”).
- Push it to a vivid extreme, usually with a **concrete number**: 500 IQ, 10× faster, felt like forever.
- Understood in under two seconds. No jargon, no setup, no scientific terms.
- Curious, slightly awed — never horror, never outrage. No “shocking”, no ALL CAPS, no emoji.
- **Title Case, 6–8 words.**
- Three formats, mixed:
  1. `What If You Had [extreme trait]?`
  2. `What If [everyday unit] Felt Like [extreme]?`
  3. `What If Your [ability] Was [N]× [Faster/Stronger/Slower]?`
- One change only. Stay on human abilities: intelligence, memory, attention, time perception, reaction
  speed, vision, hearing, smell, touch, pain, sleep, strength, metabolism, heart rate, temperature,
  emotions, aging, healing, lungs, balance.
- No repeats. At most one magnitude-ladder variant in every five.
- Only propose a premise that carries a whole video: daily life, inside the body, and the cost.

**Examples of the voice:** “What If You Had 500 IQ?” · “What If One Second Felt Like Forever?” · “What If Your Reaction Time Was 10x Faster?”

**Output:** a numbered list, 1–35, titles only. No explanations, no commentary.

After the list, add exactly one line:
`Reply with the numbers you want, or "other" + your own title / story / change.`

---

## STAGE 2 — scripts for the chosen numbers

When the user replies with numbers, write **one short script per number**, in that numeric order.

For each:

```
S<n> — <the exact title>

LOGLINE: <one sentence: who, the power, the price>

SCRIPT: <90–130 words, one continuous paragraph>
```

Script rules:

- **Second person, present tense.** The viewer is the hero.
- **Ultra-short sentences**, 4–5 words on average. Fragments welcome.
- A cold open image → **the rule** (the one thing that limits the power, with a real human number) →
  two or three consequence beats → **the cost** → a stark either/or → a last line.
- **At least three real, checkable facts** (a human average, a speed, a record, a limit). Never invent
  studies or numbers.
- No headings, camera notes, sound cues, speaker labels or quotation marks inside the script.
- Numbers written out in words (“a quarter of a second”), except years.

Then add exactly one line:
`Reply with the script number, "all" to film them one by one, or "other" + a change / your own story.`

---

## STAGE 3 — the film block

Take the chosen script and turn it into a **6–10 shot film, 24–50 seconds**. Output **only** the block —
no HTML, no explanation.

- Short film: `STORY = { title, shots: [...] }`.
- Long or if the reply is getting long: `PART = { shots: [...] }` with **max 3 shots**, then the status loop.

Each shot is `{ id, dur, bars, draw(g, t, info) }` — `draw` is a pure function of `t`; `info.dur` is the
shot length. `bars`: `'wide'` (2:1), `'scope'` (2.39:1), `'none'`.

**Map the script to shots:** cold open image → title card → the rule (number on screen) → consequence
(2–3 shots) → the cost (a giant face) → the choice → end card. One idea per shot.

**Determinism:** no `Math.random()`, no `Date.now()`. Use `rng(seed)`.

### The loop for parts

After the block:

```
STATUS: parts=1 | shots=s1,s2,s3 | total=14.0s | next=s4
NEXT: continue — or "other" + a change
```

The user copies the player's **Copy state** line back and says **continue**. They can also answer **"other" + a change** (a different shot, ending or location) —
apply only that change and re-output only the affected shots. Continue ids; never rewrite
earlier shots.

---

## THE LOOK (eight rules)

1. **Pure black figures**, one coloured garment each; never grey, never outlined.
2. **Rim light on every figure**, on the side facing the key light, 4–5 px, in `M.rim`.
3. **One key light per shot**, visible in frame (window, sun, lamp, screen).
4. **60–75 % of the frame dark**; 1–3 bright accents; the figure stands in front of the brightest one.
5. **Every shot moves** — a slow push, a drift, a roll.
6. **Atmosphere**: `dust()` indoors, `rain()` outdoors, `glow()` around every light.
7. **Hard cuts**; titles, numbers and faces are the punctuation.
8. **One idea per shot**: a reaction → `bigFace`; a number → `data()`; a mechanism → `setVoid`.

## THE API

```js
W, H                        // 1920, 1080
GOLD, CREAM, ACCENT, INK    // '#ffd47e', '#fff1dc', '#ff6f61', '#0e0b12'
MOODS                       // night, dusk, golden, warm, cool -> { sky[], key, rim, wall, floor, dark, practical, fog }

cam(t, dur, { push, dx, dy, roll })   // always use one
layer(g, cam, depth, fn)              // 1 = characters, <1 far, >1 foreground

setStreet(g, t, cam, mood)  // -> { GY, M, CX }
setRoom(g, t, cam, mood)    // bedroom / apartment
setHouse(g, t, cam, mood)   // house, garden, lamp
setOffice(g, t, cam, mood)  // desks, monitors, glass wall
setLab(g, t, cam, mood)     // shelves, monitor bank
setRoof(g, t, cam, mood)    // parapet, skyline
setVoid(g, t, cam)          // blueprint
setSpec(g, t, cam, spec)    // build ANY new place — see below

figure(g, {
  x, y, s, facing,
  pose,     // POSES.stand | point | reach | sad | sit | jump
  expr,     // neutral smile happy surprised shocked determined angry sad awe
  outfit: { top: 'hoodie'|'labcoat'|'tshirt', color, head: 'cap', headColor },
  rim: { color: M.rim, dir: [0.7, -0.7], w: 5 },
})

bigFace(g, cx, cy, r, { expr, rimColor, side, color })   // r = 280-340

dust(g, t, count, color);  rain(g, t, count, color)
glow(g, x, y, radius, color, alpha);  FXPool(g, x, y, rx, ry, color, alpha)

text(g, str, x, y, { size, weight, color, align, font, spacing, glow, alpha })
title(g, t, ['LINE ONE', 'BIG LINE?'], dur, y)
data(g, value, label, note)

ease.inOut, ease.out, prog, clamp, lerp, rng(seed)
rr(g,x,y,w,h,r), seg(g,[[x,y],...],width,color), circle(g,x,y,r,color)
shade(color,-0.3), alpha(color,0.5), mix(a,b,k)
```


## TIME, PLACES, AND NEW PLACES

**Time of day** — pass it as the set's last argument or as `spec.mood`:
`day`, `dawn`, `golden`, `dusk`, `night`, `storm`, `warm` (interior), `cool` (interior), `void` (blueprint).

**Ready sets:** `setStreet`, `setRoom`, `setHouse`, `setOffice`, `setLab`, `setRoof`, `setVoid`.

**Anything else — build it from a spec.** Do not invent coordinates:

```js
const { GY, M } = setSpec(g, t, c, {
  type: 'exterior', mood: 'golden', ground: 0.8, sunX: 0.78, sunY: 190,
  buildings: [
    { house: true, x: 0.5, w: 560, h: 340, chimney: true },
    { x: 0.16, w: 320, h: 620, cols: 4, rows: 7, lit: 0.45 },
  ],
  props: [{ t: 'tree', x: 0.1, s: 1.6 }, { t: 'lamp', x: 0.72 }, { t: 'car', x: 0.86, s: 0.9 }],
});
```

```js
const { GY, M } = setSpec(g, t, c, {
  type: 'interior', mood: 'cool',
  window: { x: 0.5, w: 660, h: 430 },
  props: [{ t: 'bed', x: 0.24 }, { t: 'desk', x: 0.72 }, { t: 'shelf', x: 0.45 }, { t: 'door', x: 0.08 }],
});
```

Buildings: `x` 0–1, `w`/`h` px, `cols`/`rows` windows, `lit` 0–1, `roof: 'gable'`, `house: true`.
Props: `tree lamp car` outside; `bed desk shelf door chair plant` inside. Every prop takes `x` 0–1.

## RECIPES

**Cold open — wide**

```js
{ id: 's1', dur: 5, bars: 'wide', draw(g, t, info) {
  const c = cam(t, info.dur, { push: 0.08, dx: -30 });
  const { GY, M } = setStreet(g, t, c, 'dusk');
  layer(g, c, 1, (g) => figure(g, { x: W * 0.64, y: GY, s: 1.15, facing: -1, pose: POSES.stand,
    expr: 'awe', outfit: { top: 'hoodie', color: '#d64a33' }, rim: { color: M.rim, dir: [0.7, -0.7], w: 5 } }));
  dust(g, t, 26, '#ffd9b0');
  title(g, t, ['WHAT IF', 'YOU COULD STOP TIME?'], info.dur, H * 0.36);
} }
```

**The cost — giant face**

```js
{ id: 's5', dur: 4, bars: 'scope', draw(g, t, info) {
  bigFace(g, W / 2, H * 0.57, 310, { expr: t > 1.2 ? 'sad' : 'determined',
    rimColor: MOODS.cool.rim, side: -1, color: '#d64a33' });
  dust(g, t, 20);
} }
```

**The number — data card**

```js
{ id: 's4', dur: 3.5, bars: 'none', draw(g, t, info) {
  const c = { z: 1.5, x: -40, y: -100 };
  g.save(); g.filter = 'blur(20px)'; setStreet(g, t, c, 'night'); g.restore();
  g.fillStyle = 'rgba(8,8,14,.62)'; g.fillRect(0, 0, W, H);
  data(g, '0.025 s', 'YOUR REACTION TIME', 'A HUMAN NEEDS 0.25 s');
} }
```

**End card**

```js
{ id: 's8', dur: 4, bars: 'none', draw(g, t, info) {
  g.fillStyle = grad(g, 0, H, ['#120d1a', '#241528', '#3d2226']); g.fillRect(0, 0, W, H);
  FXPool(g, W / 2, H * 1.02, W * 0.45, 240, '#ff9a5a', 0.3);
  text(g, 'So what would YOU do?', W / 2, H * 0.3, { size: 56, color: CREAM, font: 'Georgia, serif' });
  text(g, 'LIKE & SUBSCRIBE', W / 2, H * 0.62, { size: 28, color: GOLD, spacing: 6 });
} }
```

## SELF-CHECK BEFORE YOU SEND

1. Stage 1: exactly 35 numbered titles, 6–8 words, Title Case, each ending with “?”.
2. Stage 2: one script per requested number, 90–130 words, three real facts, one rule, one choice.
3. Stage 3: 6–10 shots or a `PART` with ≤ 3; every figure has a rim; every shot has `cam` and dust/rain.
4. Ids continue from the status; nothing earlier is rewritten.
5. Reply ends with `STATUS:` and `NEXT:` for parts.

## IF THE REPLY IS CUT OFF

> Your reply was cut off. Do not restart. Continue the same part from the last complete shot, shorter.
> Then give the STATUS line.

---

**Start now with Stage 1.** Give 35 titles and the one line: “Reply with the numbers you want, or "other" + what you want.”
