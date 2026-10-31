# STICKMAN CINEMA — the DeepSeek kit (v2, with parts)

Paste this whole file into DeepSeek Chat (or Claude, GPT, Gemini), add your story at the bottom, send.
The model answers with **one JavaScript block**. You paste it into `STICKMAN-PLAYER.html`, press
**Build film**, and the film plays. **● Record** gives you the video.

Long stories are handled in **parts**: three shots per reply. The player accumulates them and gives you
a **state line** to paste back, so the model never drifts and never has to write a file that is too big.

---

## HOW TO USE — short film (6–10 shots, one reply)

1. Open `STICKMAN-PLAYER.html` (Chrome or Edge). It plays a 26-second example.
2. Paste this file into DeepSeek, add your story, send.
3. Copy the `STORY = { ... }` block it returns.
4. In the player: **STORY CODE** → paste → **Build film** → **● Record**.
5. `ffmpeg -i film.webm -c:v libx264 -crf 18 -pix_fmt yuv420p film.mp4`

## HOW TO USE — long film (parts)

1. Ask for the first part (3 shots). Paste it into the player → **Build film**.
2. Press **Copy state**. Paste that line into the chat and send it with the word **continue**.
3. The model answers `PART 2` with three more shots. In the player press **Add as part**.
4. Repeat. The player shows `3 parts · 9 shots · 34s`.
5. Record only when all parts are in.

## THE "OTHER" OPTION

Every reply ends with:

> Want a change, a different location, or a different story? Reply **"other" + what you want.**

- "other: make it darker / shorter / night instead of day" → change only that, keep every shot and id.
- "other: build a school instead" → rebuild the set with `setSpec`, keep the story.
- "other: here is my own story …" → start a new `STORY`.
- "other" alone → ask one question: "A change, a location, or a new story?"

Never ignore "other", and never renumber what already exists.

Keep the same chat open: the state line is the guard, the chat history is the memory. If the chat resets,
re-paste this kit + your story + the state line.

---

## THE CONTRACT (model: obey this exactly)

**Short film:** one block, 6–10 shots, 3–6 s each.

```js
STORY = {
  title: 'Short Title',
  shots: [
    { id: 's1', dur: 5, bars: 'wide', draw(g, t, info) { /* ... */ } },
  ],
};
```

**Long film:** one block per reply, **maximum 3 shots**, `PART` instead of `STORY`:

```js
PART = {
  shots: [
    { id: 's4', dur: 4, bars: 'scope', draw(g, t, info) { /* ... */ } },
  ],
};
```

Rules that never change:

- **Only the block.** No HTML, CSS, explanation, or engine code.
- `draw(g, t, info)` is a pure function of `t`; `info.dur` is the shot length.
- `bars`: `'wide'` (2:1), `'scope'` (2.39:1), `'none'` (full frame).
- **Deterministic:** no `Math.random()`, no `Date.now()`. Use `rng(seed)`.
- **Continue ids** (`s4`, `s5`, `s6`); never renumber or rewrite earlier shots.
- Every part ends with a **STATUS line** (see the loop below) and the **other** line.

## THE PROMPT LOOP — how the model stays on the story

Every reply has exactly three parts:

```
PART 2

```js
PART = { shots: [ ... ] };
```

STATUS: parts=2 | shots=s1,s2,s3,s4,s5,s6 | total=21.5s | next=s7
NEXT: continue — or "other" + a change
```

- **`STATUS`** is the player's **Copy state** line, pasted back. It is the ground truth of what exists.
  The model must not contradict it.
- **`NEXT`** tells the user what to send.
- On the next turn, the model reads the state, checks the ids, and writes only the new shots.

If the user says **continue** with no state, the model asks for the state line or continues from the last
shot id it can see in the conversation.

## IF THE REPLY IS CUT OFF

Symptom: the block ends mid-function, the chat shows a length error, or JSON is truncated.

Send exactly this:

> Your reply was cut off. Do not restart. Continue the SAME part from the last complete shot. If no
> shot was complete, rewrite only that one shot, shorter (under 25 lines). Then give the STATUS line.

Never ask the model to “write the whole film again” — that is what causes the limit error.

## THE LOOK (eight rules)

1. **Pure black figures**, one coloured garment each, never grey, never outlined.
2. **Rim light on every figure**, on the side facing the key light — 4–5 px, in `M.rim`.
3. **One key light per shot**, visible in frame (window, sun, lamp, screen).
4. **60–75 % of the frame dark**, 1–3 bright accents; the figure stands in front of the brightest one.
5. **Every shot moves**: a slow push, a drift, a roll.
6. **Atmosphere**: `dust()` indoors, `rain()` outdoors, `glow()` around every light.
7. **Hard cuts**; titles, numbers and faces are the punctuation.
8. **One idea per shot.** A reaction → `bigFace`; a number → `data()`; an explanation → `setVoid`.

## THE API (exact)

```js
W, H                       // 1920, 1080
GOLD, CREAM, ACCENT, INK   // '#ffd47e', '#fff1dc', '#ff6f61', '#0e0b12'
MOODS                      // night, dusk, golden, warm, cool -> { sky[], key, rim, wall, floor, dark, practical, fog }

cam(t, dur, { push, dx, dy, roll })   // always use one
layer(g, cam, depth, fn)              // 1 = characters, <1 far, >1 foreground

setStreet(g, t, cam, mood)   // -> { GY, M, CX }
setRoom(g, t, cam, mood)     // -> { GY, M, CX }   bedroom / apartment
setHouse(g, t, cam, mood)    // -> { GY, M, CX }   house exterior, garden, lamp
setOffice(g, t, cam, mood)   // -> { GY, M, CX }   desks, monitors, glass wall
setLab(g, t, cam, mood)      // -> { GY, M, CX }   shelves, monitor bank, desk
setRoof(g, t, cam, mood)     // -> { GY, M, CX }   parapet, skyline, water tank
setVoid(g, t, cam)           // -> { GY, M, CX }   blueprint grid
setSpec(g, t, cam, spec)     // -> { GY, M, CX }   build a NEW location (see below)

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



## TIME OF DAY

Pass the mood as the last argument to any set, or as `spec.mood`:

| mood | when | light |
|---|---|---|
| `day` | noon, overcast or clear | cold white key, bright sky |
| `dawn` | first light | pale rose key, long shadows |
| `golden` | sunset hour | big orange sun, amber rim |
| `dusk` | after sunset | mauve shadows, warm windows |
| `night` | night | deep navy, sodium lamps, blue rim |
| `storm` | rain, thunder | grey-blue, hard flashes |
| `warm` | interior at night | lamps, amber, soft falloff |
| `cool` | interior, screens | monitor glow, blue-white |
| `void` | blueprint | navy grid, gold accents |

## A NEW LOCATION FROM PARTS

For anything not in the list — a school, a hospital, a warehouse, a rooftop bar — do **not** invent
coordinates. Pass a spec:

```js
const { GY, M } = setSpec(g, t, c, {
  type: 'exterior', mood: 'golden', ground: 0.8, sunX: 0.78, sunY: 190,
  buildings: [
    { house: true, x: 0.5, w: 560, h: 340, color: '#4a3340', chimney: true },   // a house
    { x: 0.16, w: 320, h: 620, cols: 4, rows: 7, lit: 0.45 },                    // a tower
  ],
  props: [
    { t: 'tree', x: 0.1, s: 1.6 }, { t: 'lamp', x: 0.72, h: 300 },
    { t: 'car', x: 0.86, s: 0.9, color: '#2a2030' },
  ],
});
```

Interior:

```js
const { GY, M } = setSpec(g, t, c, {
  type: 'interior', mood: 'cool', ground: 0.8,
  window: { x: 0.5, w: 660, h: 430 },
  props: [
    { t: 'bed', x: 0.24, w: 440 }, { t: 'desk', x: 0.72, w: 560 },
    { t: 'shelf', x: 0.45, w: 380 }, { t: 'door', x: 0.08 },
    { t: 'chair', x: 0.6 }, { t: 'plant', x: 0.92 },
  ],
});
```

**Buildings:** `x` 0–1 across the frame, `w`/`h` in pixels, `cols`/`rows` windows, `lit` 0–1,
`roof: 'gable'` for a pitched roof, `house: true` for a house with a roof and two lit windows.

**Props:** `tree` · `lamp` · `car` outdoors; `bed` · `desk` · `shelf` · `door` · `chair` · `plant`
indoors. Every prop takes `x` (0–1); most take `w`, `s`, `h`, `color`.

**Or go one level lower:** `building`, `house`, `winGrid`, `skyline`, `tree`, `lamp`, `car`,
`bedProp`, `deskProp`, `shelfProp`, `doorProp`, `chairProp`, `plantProp` — all take `(g, …)` and draw
in world coordinates.

## SHOT RECIPES

**Establishing wide**

```js
{ id: 's1', dur: 5, bars: 'wide', draw(g, t, info) {
  const c = cam(t, info.dur, { push: 0.08, dx: -30 });
  const { GY, M } = setStreet(g, t, c, 'dusk');
  layer(g, c, 1, (g) => figure(g, { x: W * 0.64, y: GY, s: 1.15, facing: -1, pose: POSES.stand,
    expr: 'awe', outfit: { top: 'hoodie', color: '#d64a33' }, rim: { color: M.rim, dir: [0.7, -0.7], w: 5 } }));
  dust(g, t, 26, '#ffd9b0');
  title(g, t, ['THE LAST', 'LIGHTKEEPER'], info.dur, H * 0.36);
} }
```

**Two-shot**

```js
{ id: 's2', dur: 5, bars: 'wide', draw(g, t, info) {
  const c = cam(t, info.dur, { push: 0.05, dx: 20 });
  const { GY, M } = setRoom(g, t, c, 'warm');
  layer(g, c, 1, (g) => {
    figure(g, { x: W * 0.38, y: GY, s: 1.55, facing: 1, pose: POSES.stand, expr: 'determined',
      outfit: { top: 'hoodie', color: '#d64a33' }, rim: { color: M.rim, dir: [0, -1], w: 4.5 } });
    figure(g, { x: W * 0.62, y: GY, s: 1.55, facing: -1, pose: POSES.stand, expr: 'smile',
      outfit: { top: 'labcoat' }, rim: { color: M.rim, dir: [0, -1], w: 4.5 } });
  });
  dust(g, t, 40);
} }
```

**Giant face**

```js
{ id: 's3', dur: 4, bars: 'scope', draw(g, t, info) {
  bigFace(g, W / 2, H * 0.57, 310, { expr: t > 1.2 ? 'determined' : 'awe',
    rimColor: MOODS.cool.rim, side: -1, color: '#d64a33' });
  dust(g, t, 20);
} }
```

**Number card**

```js
{ id: 's4', dur: 3.5, bars: 'none', draw(g, t, info) {
  const c = { z: 1.5, x: -40, y: -100 };
  g.save(); g.filter = 'blur(20px)'; setStreet(g, t, c, 'night'); g.restore();
  g.fillStyle = 'rgba(8,8,14,.62)'; g.fillRect(0, 0, W, H);
  data(g, '29', 'CAME BEFORE HIM', 'ALL OF THEM GONE');
} }
```

**End card**

```js
{ id: 's6', dur: 4, bars: 'none', draw(g, t, info) {
  g.fillStyle = grad(g, 0, H, ['#120d1a', '#241528', '#3d2226']); g.fillRect(0, 0, W, H);
  FXPool(g, W / 2, H * 1.02, W * 0.45, 240, '#ff9a5a', 0.3);
  text(g, 'What would YOU do?', W / 2, H * 0.3, { size: 56, color: CREAM, font: 'Georgia, serif' });
  text(g, 'LIKE & SUBSCRIBE', W / 2, H * 0.62, { size: 28, color: GOLD, spacing: 6 });
} }
```

## STORY SHAPE

Cold open → title → escalation (2–3 beats) → the cost or turn (a face) → resolution → end card.
The visuals show what the words *mean*, not what they say.

## SELF-CHECK (model: before sending)

1. Only one block; `STORY` for a short film, `PART` with **≤ 3 shots** for a long one.
2. Every figure has a `rim`; every shot calls `cam()`.
3. Each shot has `dust`/`rain`/`glow`.
4. One `bigFace` and one number or title per film (not per part).
5. No `Math.random`, no `Date.now`, no HTML.
6. Ids continue from the state; nothing earlier is rewritten.
7. The reply ends with `STATUS:` and `NEXT:` — and the "other" line so the user can change direction.

---

## STORY TO ADAPT

<<< paste your story, script or paragraph here >>>

If the story is long, add this line: **“Do this in 3-shot parts. Start with PART 1 and end with STATUS.”**
