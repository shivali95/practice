# Devotion Vertical Prototype — Implementation Plan

> **For Claude:** REQUIRED SUB-SKILL: Use superpowers:executing-plans to implement this plan task-by-task.

**Goal:** Build a single-file interactive HTML prototype for the JioBharatIQ Devotion vertical — 5-faith onboarding, light-mode home with all 3 modules, dark Sacred Mode playback, Calendar tab, Ritual Builder, Seasonal Companion, and pre-canned AI chat responses.

**Architecture:** Single `devotion-prototype.html` file. Vanilla JS state machine (`currentFaith`, `currentScreen`, `onboardingStep`). JDS CSS custom properties throughout. Seven screen sections, one visible at a time via `screen.active` class. Sacred Mode is a full-screen overlay toggled via `body.sacred-mode` class.

**Tech Stack:** HTML5, CSS3 (custom properties, grid, flexbox), Vanilla JS (ES6+), JioType from CDN, HelloJio state videos from CDN, JDS tokens inline.

**Design reference:** `docs/plans/2026-03-04-devotion-vertical-prototype-design.md`
**Source doc:** `DEVOTION VERTICAL — USE CASE DETAILS DOCUMENT.pdf`
**Screenshot ref:** Attached screenshot in conversation (JioBharatIQ home, dark navy, ritual card, Panchang, chat input)

---

## Pre-flight

Before starting, confirm:
- Preview server running: `python3 /tmp/serve_jio.py` on port 8080
- Working file path: `/Users/shivali.wason/Documents/devotion/devotion-prototype.html`
- After each task: copy file to `/tmp/devotion-prototype.html`, take preview screenshot to verify

---

## Task 1: Base Scaffold + JDS Tokens + Bottom Shell

**Files:**
- Create: `/Users/shivali.wason/Documents/devotion/devotion-prototype.html`

**What to build:**
Full HTML scaffold with:
- All JDS CSS custom property tokens in `:root`
- JioType @font-face declarations from CDN (400/500/700/900)
- Device frame (390×844px, border-radius 44px, shadow)
- `body.sacred-mode` class stub (background: #000)
- Faith accent variable: `--faith-accent: #FF6B35` (Hindu default)
- 7 screen divs: `#screen-onboarding`, `#screen-home`, `#screen-sacred`, `#screen-completion`, `#screen-calendar`, `#screen-ritual`, `#screen-seasonal` — all `display:none` except onboarding
- Bottom nav (Home / Calendar / Ritual / You) — positioned absolute at bottom of frame
- Chat input bar (pinned above bottom nav, always visible on non-onboarding screens)
- Status bar (time, signal, battery SVGs)
- JS state: `let state = { faith: null, region: null, language: 'Hindi', deity: null, duration: 'regular', step: 1, screen: 'onboarding', sacredMode: false }`
- `showScreen(name)` function that hides all, shows target
- `setFaithAccent(faith)` that updates `--faith-accent` CSS var

**Faith accent map (in JS):**
```js
const FAITH_ACCENTS = {
  hindu:    '#FF6B35',
  muslim:   '#2D8A4E',
  sikh:     '#1A5FB4',
  jain:     '#8B1A1A',
  christian:'#6B3FA0'
};
```

**JDS token block (CSS :root):**
```css
--primary-50: #3535f3;
--primary-20: #e8e8fc;
--secondary-50: #f7ab20;
--sparkle-50: #1eccb0;
--sparkle-20: #cff6ef;
--error-50: #fa2f40;
--success-50: #25ab21;
--grey-100: #141414;
--grey-80: #474747;
--grey-60: #757575;
--grey-40: #e0e0e0;
--grey-20: #f5f5f5;
--surface-ghost: #eeeeef;
--white: #ffffff;
--faith-accent: #FF6B35;
```

**Verify:** Copy to /tmp, open http://localhost:8080/devotion-prototype.html — should see empty device frame, bottom nav visible.

**Commit:**
```bash
git add devotion-prototype.html
git commit -m "feat: base scaffold, JDS tokens, screen shell, bottom nav"
```

---

## Task 2: Onboarding — Step 1 (Faith Selection)

**Files:**
- Modify: `devotion-prototype.html` — populate `#screen-onboarding`

**What to build:**
- JioBharatIQ logo + wordmark centred at top
- Progress dots row (5 dots, step 1 active = filled primary)
- Headline: "Aapki aastha kya hai?" (title-l, JioType Black)
- Subtext: "Hum aapka daily ritual personalise karenge" (body-m, grey-60)
- Faith cards — 2×2 grid + 1 centred below:

```
[ Hindu 🪔 ]  [ Muslim ☪️ ]
[ Sikh  ☬  ]  [ Jain   🔱 ]
      [ Christian ✝️ ]
```

Each card: 156×96px, border-radius 16px, surface-ghost bg, faith icon SVG (no emoji — use inline SVG paths), faith name bold, faith name in native script below (smaller, grey-60). On tap: border 2px var(--faith-accent), bg becomes faith accent at 12% opacity, store `state.faith`.

- Continue button: appears after faith selected, primary-50 bg, "Aage badhen →", fixed at bottom of screen content (above bottom nav area in onboarding)

**Faith SVG icons (inline — no emoji):**
- Hindu: diya/lamp SVG
- Muslim: crescent + star SVG
- Sikh: khanda SVG
- Jain: hand/ahimsa SVG
- Christian: cross SVG

**JS:**
```js
function selectFaith(faith) {
  state.faith = faith;
  setFaithAccent(faith);
  // highlight selected card, show continue btn
  document.querySelectorAll('.faith-card').forEach(c => c.classList.remove('selected'));
  document.querySelector(`.faith-card[data-faith="${faith}"]`).classList.add('selected');
  document.getElementById('ob-continue').classList.add('visible');
}
```

**Verify:** Screenshot — 5 faith cards visible, tapping Hindu highlights it in saffron, Continue button appears.

**Commit:**
```bash
git commit -m "feat: onboarding step 1 — faith selection with 5 faiths"
```

---

## Task 3: Onboarding — Steps 2–4 (Region / Language / Sect / Duration)

**Files:**
- Modify: `devotion-prototype.html`

**What to build:**

Each step as a `div.ob-step` inside `#screen-onboarding`, shown/hidden by step number.

**Step 2 — Region + Language:**
- Conditional region block (only visible if `state.faith === 'hindu'`):
  - Section label: "Aapka kshetra?"
  - Two large toggle cards: "Purnimant · North India" / "Amant · South & West India"
- Language section label: "Prathamik bhasha?"
- Chip grid (wrap): Hindi · English · Urdu · Punjabi · Gujarati · Tamil · Telugu · Kannada · Malayalam · Bengali · Marathi
- Selected chip: primary-50 bg, white text
- Secondary language prompt: "Doosri bhasha?" + same chips (optional, grey border selected)
- Skip link top-right

**Step 3 — Sect/Deity (faith-specific content via JS):**
Content swapped by `state.faith`:
- Hindu: "Aapke kul devata?" label + 2×3 grid of deity cards (Ganesh, Hanuman, Shiva, Vishnu, Devi, Krishna) — each with SVG icon + name
- Muslim: "Preferred Qari voice?" — 3 cards with name + "Preview" button (tap shows toast "Playing 10-sec sample...")
- Sikh: "Gurbani style?" — 2 large cards: Classical Raag / Contemporary Kirtan
- Jain: "Aapki parampara?" — 2 cards: Shvetambar / Digambar
- Christian: "Denomination?" — 3 cards: Catholic / Protestant / Orthodox

**Step 4 — Duration:**
- 3 stacked cards:
  - Quick · 5–8 min · "Weekday ki jaldi"
  - Regular · 10–18 min · "Mera daily practice" ← pre-selected border
  - Extended · 20–30 min · "Weekend, sukoon se"
- Toggle row: "Weekend mein alag duration?"
- If ON: second duration row slides down (same 3 options)
- Store `state.duration`, `state.weekendDuration`

**Step transitions:**
```js
function nextStep() {
  state.step++;
  document.querySelectorAll('.ob-step').forEach(s => s.classList.remove('active'));
  document.querySelector(`.ob-step[data-step="${state.step}"]`).classList.add('active');
  updateProgressDots();
  // slide animation: translateX -100% → 0
}
function prevStep() { state.step--; /* reverse slide */ }
```

**Verify:** Screenshot each step — faith-specific content shows correctly for Hindu vs Muslim.

**Commit:**
```bash
git commit -m "feat: onboarding steps 2-4 — region, language, sect/deity, duration"
```

---

## Task 4: Onboarding — Step 5 (Ritual Preview) + Transition to Home

**Files:**
- Modify: `devotion-prototype.html`

**What to build:**

Step 5 — Ritual Preview screen:
- Headline: "Aapka daily ritual" (title-l)
- Subtext: generated based on faith + deity + duration
- Ritual sequence card (surface-ghost bg, border-radius 20px):
  ```
  Opener  ◉  [Track 1 name]         3:24
  Main 1  ◉  [Track 2 name]         7:40
  Main 2  ◉  [Track 3 name]         4:10
  Closer  ◉  [Track 4 name]         2:15
             ─────────────────────
             Total · 15 min · 4 tracks
  ```
- Track rows: track number dot (faith accent), track name bold, duration right-aligned, grey-60
- Two CTAs stacked:
  - Primary: "Shuru karein →" full width, primary-50 bg
  - Ghost: "Customize karein" full width, border primary-50

**Ritual content map (JS object):**
```js
const RITUAL_TEMPLATES = {
  hindu: {
    ganesh: ['Ganesh Vandana', 'Hanuman Chalisa', 'Om Namah Shivaya', 'Ganesh Aarti'],
    hanuman: ['Hanuman Chalisa', 'Bajrang Baan', 'Ram Dhun', 'Hanuman Aarti'],
    shiva: ['Om Namah Shivaya', 'Shiva Tandav', 'Mahamrityunjaya', 'Shiva Aarti'],
    vishnu: ['Vishnu Sahasranama', 'Hare Krishna', 'Narayan Stuti', 'Vishnu Aarti'],
    devi: ['Durga Chalisa', 'Jai Ambe Gauri', 'Lalita Sahasranama', 'Devi Aarti'],
    krishna: ['Hare Krishna Mahamantra', 'Achyutam Keshavam', 'Govind Bolo', 'Aarti Kunj Bihari'],
  },
  muslim: { default: ['Fajr Azaan', 'Surah Al-Fatiha', 'Ayat ul Kursi', 'Dua After Prayer'] },
  sikh: { default: ['Japji Sahib', 'Anand Sahib', 'Rehras Sahib', 'Ardas'] },
  jain: { default: ['Navkar Mantra', 'Logassa', 'Bhaktamar Stotra', 'Aarti'] },
  christian: { default: ['Morning Prayer', 'Psalm 23', 'Hymn', 'Blessing'] },
};
```

"Shuru karein" → calls `completeOnboarding()`:
```js
function completeOnboarding() {
  localStorage.setItem('devotion_setup', JSON.stringify(state));
  showScreen('home');
  buildHomeScreen();
}
```

**On page load:** Check `localStorage.getItem('devotion_setup')` — if exists, restore state and go straight to home (returning user). Add "Reset / Start over" link in You tab.

**Verify:** Complete full onboarding Hindu > Ganesh > Regular → should land on home screen.

**Commit:**
```bash
git commit -m "feat: onboarding step 5 — ritual preview, transition to home, localStorage persist"
```

---

## Task 5: Home Screen — Greeting Banner + Ritual Card

**Files:**
- Modify: `devotion-prototype.html` — populate `#screen-home`

**What to build:**

**Top bar:**
- JioBharatIQ logo (coloured grid dots SVG) + "JioBharatIQ" bold + "Devotion AI · Online" grey-60 subtitle
- Right: profile icon button + notification bell button (both surface-ghost pill)

**Greeting banner card:**
- Full width, border-radius 20px, background: `var(--faith-accent)` at 15% opacity with `var(--faith-accent)` as border (1px)
- OR: Gradient bg from faith accent to primary (stronger version)
- Content:
  - Salutation line: faith-specific (Hindu: "Shubh Pratham, [name] 🙏" → use SVG not emoji), small, faith accent colour
  - Tithi headline: "Shukla Panchami · Margashirsha" (title-l, Black weight, grey-100)
  - Subtext: "Aaj Ganesh Chaturthi ke baad pehla mangal din hai" (body-s, grey-60)
- Hardcoded for demo: March 4, 2026 — Shukla Panchami

**Aaj Ka Ritual card:**
- Section label: "Aaj Ka Ritual" (label-m, grey-60, uppercase)
- Card: surface-ghost bg, border-radius 20px, padding 16px
- Left: deity icon square (faith accent bg, 52px, border-radius 14px, SVG icon white)
- Centre: ritual name bold + duration + "X tracks · Sacred Mode ready" grey
- Right: Play button circle (primary-50, 52px, white triangle SVG)
- Below: horizontal scroll chip row — track names, active track highlighted with faith accent border
- "Hanuman Chalisa" chip gets star prefix (pinned track indicator)

**Tapping play button** → `enterSacredMode()`

**Verify:** Screenshot — greeting banner in saffron, ritual card with Ganesh icon, play button, track chips.

**Commit:**
```bash
git commit -m "feat: home screen — greeting banner + aaj ka ritual card"
```

---

## Task 6: Home Screen — Panchang Widget + Seasonal Strip + JBIQ Chips

**Files:**
- Modify: `devotion-prototype.html`

**What to build:**

**Aaj Ka Panchang widget:**
- Section label: "Aaj Ka Panchang"
- Card: white bg, border 1px grey-40, border-radius 20px
- Collapsed state (default):
  - Left: tithi bold (primary-50) + date grey
  - Right: "Poora Dekhen" link (primary-50, small)
  - Below: 2 info chips in a row:
    - "ⓘ Ekadashi 6 din mein" (primary-20 bg, primary-50 text)
    - "Rahu Kaal 10:30–12:00" (secondary-20 bg, secondary-50 text)
- Expanded state (tap "Poora Dekhen"):
  - Grid of detail rows: Tithi / Paksha / Nakshatra / Yoga / Rahu Kaal / Sunrise / Sunset
  - Each row: label grey-60 left, value grey-100 bold right
  - "Band karein" link to collapse

**Faith-specific Panchang data (JS object):**
```js
const PANCHANG_DATA = {
  hindu: {
    tithi: 'Shukla Panchami', paksha: 'Shukla', nakshatra: 'Rohini',
    rahuKaal: '10:30–12:00', sunrise: '6:48 AM', sunset: '6:31 PM',
    ekadashi: '6 din mein', note: 'Margashirsha Maas'
  },
  muslim: {
    fajr: '5:14 AM', dhuhr: '12:38 PM', asr: '4:02 PM',
    maghrib: '6:31 PM', isha: '7:48 PM', hijri: '4 Sha\'ban 1447', jummah: 'Kal'
  },
  sikh: {
    nanakshahi: '20 Magh 556', gurpurab: 'Guru Gobind Singh · 12 din mein',
    amritVela: '4:00–6:00 AM', hukamnama: 'Ang 782'
  },
  jain: {
    parva: 'Shukla Panchami', fastingRule: 'Regular ahar allowed',
    nextParva: 'Ekadashi · 6 din mein'
  },
  christian: {
    liturgical: 'Ordinary Time · Week 8', sunday: 'Next Sunday: 8th Sunday',
    nextFeast: 'Ash Wednesday · 4 din mein'
  }
};
```

**Navratri Seasonal Strip** (visible since Navratri is active in demo):
- Full-width strip below Panchang: faith accent left border (4px)
- "Navratri · Day 5 of 9" label bold left
- 9 small circles right: 5 filled (faith accent), 4 empty
- Tap → `showScreen('seasonal')`

**JBIQ se Poochhein section:**
- Section label: "JBIQ se Poochhein"
- Horizontal chip scroll (quick action chips, styled with primary-20 bg, primary-50 text, border-radius 20px):
  - Hindu: "Aaj ka Rahu Kaal?", "Day 5 ki Devi?", "Hanuman Chalisa pin karein", "Kal ka ritual", "Navratri vrat rules"
  - (other faiths have their own set — switched by `state.faith`)
- Tapping a chip fires `sendChatMessage(chipText)`

**Verify:** Screenshot — Panchang card with chips, seasonal strip showing Day 5/9, JBIQ chips row.

**Commit:**
```bash
git commit -m "feat: home — panchang widget (collapsed/expanded), seasonal strip, JBIQ chips"
```

---

## Task 7: Sacred Mode (Dark Full-Screen Overlay)

**Files:**
- Modify: `devotion-prototype.html`

**What to build:**

`#screen-sacred` — black bg (`#000`), full height of device frame.

Layout (top to bottom):
- **Top bar:** Small "Sacred Mode" label centre (white, 11px, letter-spacing 0.15em), X button top-right (white, 32px tap target)
- **Track progress area (centre):**
  - Current track number / total (e.g., "2 / 4", grey-60)
  - Track name (white, display-s, Black weight, centre)
  - Artist name (grey-40, body-m, centre)
  - Faith accent colour glow behind track name (radial gradient, 20% opacity)
- **Lyrics scroll area:**
  - Scrolling text, white 15px, line-height 1.8
  - Current line highlighted: faith accent colour, slightly larger
  - Toggle icon top-right of lyrics area to hide/show
- **Progress bar:**
  - Thin bar (4px height), full width, grey-80 track, faith accent fill
  - Animated: grows from 0% to 100% over 30-sec demo timer
  - Current time left / total time (grey-60, small)
- **Controls:**
  - ← Prev · ⏸ Pause/Play · → Next (white SVG icons, 40px tap targets, spaced evenly)
  - Play/Pause toggle on tap
- **Track list pills** (bottom, horizontal scroll):
  - 4 small pills: track names, current = faith accent bg white text, others = white bg/border
  - Tap to "jump" to that track (updates track name display)

**Demo timer:** 30-second auto-advance per track → after 4 tracks (2 min total demo) → `exitSacredMode()` → show completion screen.

**enterSacredMode():**
```js
function enterSacredMode() {
  state.sacredMode = true;
  state.currentTrack = 0;
  showScreen('sacred');
  startSacredModeTimer();
  setAvatarState('speaking'); // HelloJio video if present
}
```

**exitSacredMode(manual):** X button → confirm dialog: "Ritual rokein?" → Yes → `showScreen('home')`.

**Completion (auto):** All tracks done → `showScreen('completion')`.

**Verify:** Screenshot in Sacred Mode — dark screen, track name glowing, progress bar, lyrics visible.

**Commit:**
```bash
git commit -m "feat: sacred mode — dark overlay, track progress, lyrics, controls, 30s demo timer"
```

---

## Task 8: Completion Screen

**Files:**
- Modify: `devotion-prototype.html`

**What to build:**

`#screen-completion`:
- Background: dark → fades to white over 1.5s (CSS animation on entry)
- Centre content:
  - Faith-appropriate greeting (large, title-xl, Black):
    - Hindu: "Shubh Panchami"
    - Muslim: "Alhamdulillah"
    - Sikh: "Waheguru Ji Ka Khalsa"
    - Jain: "Jai Jinendra"
    - Christian: "Praise the Lord"
  - Completion icon: animated checkmark circle (faith accent stroke, draws in via CSS)
  - Wisdom quote: 1 line, grey-80, italic, body-m, attribution grey-60 small
  - Calendar note (if relevant): chip — "Ekadashi 6 din mein" (info style)
- Two CTAs:
  - "Done" → `showScreen('home')`, mark today's ritual complete
  - "Aur Sunein →" → `showToast('Extended content — coming soon')`
- "Ritual Complete ✓" badge appears on home ritual card after returning

**Wisdom quotes map (JS):**
```js
const WISDOM = {
  hindu: { quote: "Karmanye vadhikaraste, ma phaleshu kadachana", attr: "— Bhagavad Gita 2.47" },
  muslim: { quote: "Verily, with hardship comes ease", attr: "— Quran 94:6" },
  sikh: { quote: "Ik Onkar — There is one God", attr: "— Guru Granth Sahib" },
  jain: { quote: "Ahimsa paramo dharma", attr: "— Jain Agamas" },
  christian: { quote: "I can do all things through Christ who strengthens me", attr: "— Philippians 4:13" },
};
```

**Verify:** Screenshot — fading dark-to-light, faith greeting, wisdom, Done button.

**Commit:**
```bash
git commit -m "feat: completion screen — faith greeting, wisdom, dark-to-light fade, Done CTA"
```

---

## Task 9: Calendar Tab

**Files:**
- Modify: `devotion-prototype.html`

**What to build:**

`#screen-calendar`:
- Top bar: "Aaj Ka Panchang" title + settings icon
- **Today card (expanded):** Full detail card at top — all Panchang fields for current faith (from PANCHANG_DATA). Each row: icon + label + value.
- **Week strip:** Horizontal 7-day row. Each day: day name (3 chars) + date number + coloured dot if event. Today = primary-50 bg pill. Tap day → updates today card to that day's data (stub for non-today dates).
- **Month grid:** 7-col grid. Each cell: date number + coloured dots below (≤3). Today highlighted. Festival dates have faith accent dot. Vrat dates have secondary-50 dot. Tap month header left/right to navigate (prev/next month stubs — show toast "Calendar data coming soon").
- **Upcoming section:** "Aane wale din" label + list of 5 upcoming dates:
  ```
  Ekadashi       Mar 10  (6 din mein)
  Holi           Mar 14  (10 din mein)
  Navratri ends  Mar 12  (8 din mein)
  Chaitra Navami Mar 17
  Ram Navami     Mar 17
  ```
  Each as a row card: coloured left border (faith accent for festivals, secondary for vrats).

**Hardcoded calendar data for March 2026:** Mark ~8 notable dates with dots.

**Verify:** Screenshot — today card with Panchang details, week strip, month grid with dots, upcoming list.

**Commit:**
```bash
git commit -m "feat: calendar tab — today card, week strip, month grid, upcoming events"
```

---

## Task 10: Ritual Builder

**Files:**
- Modify: `devotion-prototype.html`

**What to build:**

`#screen-ritual`:
- Top bar: "Mera Ritual" title + save button (primary-50, "Sachchao")
- **Current sequence** as draggable track cards:
  - Each card: track number circle (faith accent) + track name + duration + 3 action icons (↕ drag handle, 📌 pin, 🚫 block)
  - Pin icon: tap → toggles pinned state (faith accent pin icon, "Pinned" label appears). Max 3 pins — if exceeded: toast "Pin sirf 3 tracks tak ho sakta hai"
  - Block icon: tap → confirm toast "Is track ko block karein?" → Yes → card gets strikethrough, moves to bottom, grey opacity
  - Drag handle: visual only (no real DnD needed — tap up/down arrows instead for prototype)
- **+ Track jodhein** button (dashed border card, faith accent dashed):
  - Tap → opens inline panel with category chips: Vandana · Stuti · Chalisa · Aarti · Mantra
  - Below: list of 6 sample track cards per category, each with name + duration + "Preview" button (toast "30-sec preview playing...") + "Add +" button
  - Add → inserts into sequence list
- **Reset button** (ghost, grey): "Recommended template pe wapas jayen" → confirm toast
- **Save:** Updates `state.ritual`, shows toast "Ritual save ho gaya. Kal se apply hoga."

**Verify:** Screenshot — track sequence, pin/block interactions working, add panel.

**Commit:**
```bash
git commit -m "feat: ritual builder — sequence editor, pin/block, add track panel"
```

---

## Task 11: Seasonal Companion Dashboard

**Files:**
- Modify: `devotion-prototype.html`

**What to build:**

`#screen-seasonal` — shows different content based on `state.faith`. Demo defaults to Navratri (Hindu).

**Shared structure:**
- Top bar: Season name + "Day X of Y" + back arrow
- Progress strip: filled/empty circles row

**Navratri (Hindu — active in demo, Day 5):**
- **Devi of the Day card:** "Skandamata" headline + significance text (2-3 lines) + "Kahani Sunein" audio button (stub → toast)
- **Colour of the day chip:** Saffron/yellow/red/etc. with coloured dot
- **Vrat rules accordion:** Tap to expand → "Sabudana, kuttu, singhara atta allowed. Grains, non-veg, alcohol avoided."
- **Puja steps accordion:** Tap to expand → numbered steps (1. Snan 2. Diya jalayein 3. Pushp arpit karein 4. Aarti)
- **Garba events card:** "Aaj raat ke Garba events" → 3 placeholder event cards (venue name, time, distance)

**Ramadan (Muslim):**
- **Sehri/Iftar timer:** Large countdown "Iftar mein: 6h 42m" (ticking JS timer)
- **Fasting status badge:** "Fast active since 4:48 AM" (success-50 bg)
- **Quran tracker:** "Juz 14 of 30" + progress bar + "On track" chip
- **Daily Hadith card:** Arabic text + Urdu translation

**Paryushana (Jain):**
- **Sutra of the day:** "Uttaradhyayana Sutra — Day 3" + text excerpt
- **Fasting tracker:** Type badge (Upvas / Ekasana / Ayambil) + "62 ghante se" counter
- **Reflection prompt card:** "Aaj ek vyakti ke saath kshamapana karein jisse aapne hurt kiya ho"

**Faith selector tabs** at top of seasonal screen (small): Navratri · Ramadan · Paryushana → switches content for demo.

**Verify:** Screenshot — Navratri Day 5 default, Skandamata card, Garba events, accordion working.

**Commit:**
```bash
git commit -m "feat: seasonal companion — Navratri/Ramadan/Paryushana dashboards"
```

---

## Task 12: Chat / AI Layer (Pre-Canned Responses)

**Files:**
- Modify: `devotion-prototype.html`

**What to build:**

The chat input bar (already in shell) now becomes fully functional.

**Chat area:** When a message is sent, a chat panel slides up from above the input bar (or an inline section on home scrolls to show messages). Messages stack as bubbles:
- User message: right-aligned, primary-50 bg, white text
- AI response: left-aligned, surface-ghost bg, grey-100 text, with JBIQ avatar dot (faith accent)

**Response flow:**
1. User types / taps chip → user bubble appears
2. Skeleton shimmer row (1.2s)
3. Streaming text response renders word by word

**Pre-canned response map (JS object — 20+ entries):**
```js
const AI_RESPONSES = {
  'rahu kaal': {
    title: 'Aaj Ka Rahu Kaal',
    body: 'Aaj ka Rahu Kaal <b>10:30 AM – 12:00 PM</b> hai. Is samay mahtvapurn kary shuru karne se bachein.',
  },
  'day 5': {
    title: 'Navratri Day 5 — Skandamata',
    body: 'Aaj ki Devi <b>Skandamata</b> hain — Kartikeya ki maa. Inhe <b>safed rang</b> priya hai. Puja mein lotus pushp arpit karein.',
  },
  'hanuman chalisa pin': {
    title: 'Hanuman Chalisa Pinned',
    body: 'Ho gaya! <b>Hanuman Chalisa</b> aapke har din ke ritual mein pin ho gayi hai. Festivals mein bhi yeh zaroor chalegi.',
    action: { label: 'Ritual dekhen', screen: 'ritual' }
  },
  'kal ka ritual': {
    title: 'Kal Ka Ritual — Shukla Shashthi',
    body: 'Kal Navratri Day 6 hai — <b>Katyayani Devi</b> ka din. Aapke ritual mein <b>Katyayani Stotra</b> seasonal slot mein add ho jayega.',
  },
  'navratri vrat': {
    title: 'Navratri Vrat Rules',
    body: 'Allowed: <b>Sabudana, Kuttu, Singhara atta, Fruits, Milk, Sendha namak</b>.\nAvoided: Grains, Non-veg, Alcohol, Regular salt.',
  },
  'fajr': {
    title: 'Aaj Ki Fajr Timing',
    body: 'Aaj Fajr <b>5:14 AM</b> hai (Mumbai). Sunrise 6:48 AM se pehle namaz ada karein.',
  },
  'sehri': {
    title: 'Ramadan Sehri Timing',
    body: 'Kal Sehri ka waqt <b>5:08 AM</b> hai. Abhi Iftar mein <b>6 ghante 42 minute</b> baaki hain.',
  },
  'hukamnama': {
    title: 'Aaj Ka Hukamnama',
    body: 'Ang <b>782</b> — "Har har naam dhiaaeeai. Gurmukh milai vadiaaee." — Sri Guru Granth Sahib Ji.',
  },
  'gurpurab': {
    title: 'Agle Gurpurab',
    body: 'Guru Gobind Singh Ji ka Gurpurab <b>12 din mein</b> hai (16 March). Keertan kirtan ki planning shuru karein.',
  },
  'lent': {
    title: 'Lent 2026',
    body: 'Ash Wednesday <b>4 din mein</b> hai (8 March). Lent 40 din chalti hai Easter (12 April) tak.',
  },
  'paryushana': {
    title: 'Paryushana Bhojan',
    body: 'Paryushana mein <b>Ayambil</b> (unseasoned food once daily) ka prachar hai. Roots (aloo, pyaz) avoid karein. Atthai mein poora upavas.',
  },
  'default': {
    body: 'JBIQ yahan hai! Aap apne ritual, Panchang, Navratri, ya kisi bhi dharmik sawal ke baare mein poochh sakte hain.',
  }
};
```

**sendChatMessage(text):**
```js
function sendChatMessage(text) {
  appendUserBubble(text);
  clearInput();
  showSkeletonBubble();
  setTimeout(() => {
    removeSkeletonBubble();
    const resp = findResponse(text.toLowerCase());
    appendAIBubble(resp);
  }, 1200);
}

function findResponse(text) {
  for (const key of Object.keys(AI_RESPONSES)) {
    if (text.includes(key)) return AI_RESPONSES[key];
  }
  return AI_RESPONSES['default'];
}
```

**Mic button:** Tap → shows voice overlay (reuse HelloJio listening state video pattern from previous prototype), simulates transcript, sends message.

**Camera button:** Tap → toast "Puja samagri identify karein — coming soon".

**Verify:** Type "Rahu Kaal" → user bubble → skeleton → AI response with title card. Tap chip → same flow.

**Commit:**
```bash
git commit -m "feat: chat AI layer — pre-canned responses, streaming text, voice stub, 20+ response keys"
```

---

## Task 13: You Tab + LocalStorage Reset

**Files:**
- Modify: `devotion-prototype.html`

**What to build:**

`#screen-you`:
- Profile hero (faith accent bg, 25% opacity): avatar circle (faith accent solid, initials "PA"), name "Priya Anjali", faith name + deity below
- Settings rows (tap → toast):
  - "Meri bhasha" → current language
  - "Duration badlein" → duration picker inline
  - "Ritual customize" → `showScreen('ritual')`
  - "Calendar display" → toggle list (tithi, nakshatra, rahu kaal toggles)
  - "Seasonal companion" → toggle on/off
  - "Notifications" → stub
- **"Dobara setup karein"** row (error-50 text, bottom): tap → confirm → `localStorage.clear()` → `showScreen('onboarding')` → resets all state

**Verify:** You tab shows profile, settings rows, reset works (returns to Step 1).

**Commit:**
```bash
git commit -m "feat: you tab — profile, settings rows, reset onboarding flow"
```

---

## Task 14: JDS Validation + Polish + Copy to /tmp

**Files:**
- Modify: `devotion-prototype.html` (final pass)

**What to do:**

1. Run JDS validate_prototype on the complete HTML
2. Fix any font-family fallbacks (remove `sans-serif`)
3. Fix any hardcoded hex values outside JDS palette
4. Replace any emoji used as icons with inline SVG paths
5. Verify all 5 faith paths work end-to-end:
   - Hindu: Onboarding → Home → Sacred Mode → Completion → Calendar → Ritual → Seasonal (Navratri)
   - Muslim: Onboarding → Home (green accent, prayer times in Panchang) → Seasonal (Ramadan)
   - Sikh, Jain, Christian: Onboarding → Home (correct accent, correct Panchang)
6. Verify localStorage persist: complete onboarding → refresh → lands on home
7. Copy final file: `cp /Users/shivali.wason/Documents/devotion/devotion-prototype.html /tmp/devotion-prototype.html`
8. Take final preview screenshots of: onboarding step 1, home (Hindu), Sacred Mode, Calendar, Seasonal (Navratri)

**Final commit:**
```bash
git add devotion-prototype.html
git commit -m "feat: JDS validation pass, polish, final prototype complete"
```

---

## Done Criteria

- [ ] All 5 faith paths selectable in onboarding
- [ ] Returning user lands on home (localStorage)
- [ ] Home shows greeting banner (faith accent), ritual card, Panchang, seasonal strip, JBIQ chips
- [ ] Sacred Mode: dark screen, track name, lyrics, progress, controls, 30s auto-advance
- [ ] Completion screen: faith greeting, wisdom, Done returns to home
- [ ] Calendar tab: today card, week strip, month grid, upcoming
- [ ] Ritual builder: sequence view, pin/block, add track
- [ ] Seasonal companion: Navratri Day 5 default, Ramadan, Paryushana switchable
- [ ] Chat: 20+ pre-canned responses, streaming text, skeleton
- [ ] You tab: profile, reset flow
- [ ] Zero JDS violations (validate_prototype passes)
- [ ] Preview server running, file accessible at localhost:8080
