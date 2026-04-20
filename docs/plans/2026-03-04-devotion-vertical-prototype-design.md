# Devotion Vertical — Prototype Design
**Date:** 2026-03-04
**Author:** Shivali Wason
**Status:** Approved — ready for implementation

---

## Overview

Interactive HTML prototype for the JioBharatIQ Devotion vertical. Chat-first interface with a persistent bottom input bar, multi-faith onboarding, and all three core modules: Daily Devotion Engine, Devotional Calendar, and Seasonal Companion.

**Approach:** JDS light mode home → dark Sacred Mode transition (Approach C). Single HTML file, pre-canned AI responses, no backend required.

---

## Screen Architecture

7 screens, single HTML file. Chat input + bottom nav persist across Home / Calendar / Ritual / Seasonal.

| # | Screen | Mode | Trigger |
|---|--------|------|---------|
| 1 | Onboarding (5 steps) | Light | First launch |
| 2 | Home | Light | Post-onboarding |
| 3 | Sacred Mode | Dark | Tap play |
| 4 | Completion Screen | Dark → Light fade | Ritual ends |
| 5 | Calendar Tab | Light | Bottom nav |
| 6 | Ritual Builder | Light | Customize CTA / Ritual tab |
| 7 | Seasonal Companion | Light + faith accent | Season strip tap |

### Navigation
```
Bottom Nav:  [ Home ]  [ Calendar ]  [ Ritual ]  [ You ]
Chat bar:    [ JBIQ se kuch bhi poochhein... ] [mic] [cam] [send]
```

---

## Faith → Theme Mapping

| Faith | Accent Hex | Usage |
|-------|-----------|-------|
| Hindu | `#FF6B35` (Saffron) | Greeting banner, chips, seasonal cards |
| Muslim | `#2D8A4E` (Green) | Same |
| Sikh | `#1A5FB4` (Royal Blue) | Same |
| Jain | `#8B1A1A` (Maroon) | Same |
| Christian | `#6B3FA0` (Purple) | Same |

Base palette: JDS tokens throughout (primary `#3535f3`, surface-ghost `#eeeeef`, white `#ffffff`).

---

## Screen 1 — Onboarding (5 Steps)

**Step 1 — Faith Selection**
- Full screen, JioBharatIQ logo top centre
- Headline: "Aapki aastha kya hai?"
- 5 faith cards (2+2+1 grid): icon + name (English + script) + accent tint
- Tap → card highlights, Continue button appears

**Step 2 — Region + Language**
- Hindu only: Purnimant (North) vs Amant (South/West) region toggle
- All faiths: Language chips (11 languages), auto-detected pre-selected
- Primary + optional secondary language

**Step 3 — Sect / Deity / Tradition**
- Hindu: Deity grid (Ganesh, Hanuman, Shiva, Vishnu, Devi, Krishna)
- Muslim: 3 Qari voice cards with 10-sec audio preview
- Sikh: Classical Raag vs Contemporary Kirtan
- Jain: Shvetambar vs Digambar
- Christian: Catholic / Protestant / Orthodox

**Step 4 — Duration**
- Quick (5–8 min) / Regular (10–18 min, pre-selected) / Extended (20–30 min)
- Toggle: "Weekend mein alag duration?" → second picker appears if ON

**Step 5 — Ritual Preview**
- System-generated track sequence card (Opener + 2 Main + Closer)
- Total time + track count
- CTA: "Shuru karein →" (primary) + "Customize karein" (ghost)
- Note: "Kal ka ritual auto-update hoga festival ke hisaab se"

**Transitions:** Slide left forward, slide right back. Progress dots top. Steps 2–4 have Skip link.

---

## Screen 2 — Home (Light Mode)

Scrollable, stacked sections:

1. **Greeting banner** — faith accent bg, personalised Hindi greeting, today's tithi + Gregorian date
2. **Aaj Ka Ritual card** — track sequence chips (active chip highlighted), Sacred Mode play button, duration + track count
3. **Aaj Ka Panchang widget** — collapsed 2-line default (tithi + next event), tap to expand full card with faith-specific fields (Rahu Kaal, Nakshatra, prayer times, etc.)
4. **Seasonal strip** — visible only during active season; progress dots (Day X of Y), tappable → Seasonal Companion dashboard
5. **JBIQ se Poochhein** — label + horizontal chip row of quick queries → triggers pre-canned AI response inline

**Pinned bottom:** Chat input bar + bottom nav.

---

## Screen 3 — Sacred Mode (Dark)

Triggered by play button tap. Full-screen overlay, black bg.

- Track title + artist name (white, large)
- Scrolling lyrics (white, centre-aligned, current line highlighted in faith accent)
- Waveform / progress bar (faith accent colour)
- Controls: Previous · Play/Pause · Next (white icons, large tap targets)
- Top right: X to exit Sacred Mode
- All phone notifications silenced during playback
- Auto-advances track-by-track
- On ritual completion → transitions to Completion Screen

---

## Screen 4 — Completion Screen (Dark → Light)

- Faith-appropriate greeting (e.g., "Shubh Panchami" / "Alhamdulillah" / "Waheguru Ji Ka Khalsa")
- One-line attributed wisdom quote
- Calendar note if relevant ("Ekadashi 3 din mein")
- Two CTAs: "Done" → returns to Home | "Aur Sunein →" → extended content placeholder

---

## Screen 5 — Calendar Tab (Light)

- Today card (expanded at top): full tithi/Panchang details, faith-specific fields
- Horizontal week strip: 7 days, coloured dots for events
- Swipeable month grid: festivals (saffron dot), vrats (gold dot), prayer times (teal dot)
- Upcoming section: next 5 notable dates with countdown

---

## Screen 6 — Ritual Builder (Light)

- Current sequence as draggable track cards (↕ handle)
- Per-track actions: Pin icon (max 3 pinned) · Block icon · 30-sec preview
- "+ Track todhein" button → Browse catalog by category / Search
- Save button (sticky bottom)
- Reset to recommended template option

---

## Screen 7 — Seasonal Companion (Light + Faith Accent)

**Navratri (active in demo, Day 5):**
- Day tracker: filled/empty circle progress row
- Devi of the day: name + significance card + 5-min audio
- Colour of the day chip
- Vrat rules (accordion)
- Puja steps (accordion)
- Garba events list (placeholder cards)

**Ramadan:**
- Sehri/Iftar countdown timer (live ticking)
- Quran tracker (Juz X of 30, progress bar)
- Fasting status badge

**Paryushana:**
- Sutra of the day (audio + text)
- Fasting tracker (type + hours counter)
- Reflection prompt

---

## Chat / AI Layer

Persistent bottom input. Pre-canned responses keyed to query text.

**Quick chips per faith (examples):**
- Hindu: "Aaj ka Rahu Kaal?", "Navratri Day 5 ki Devi?", "Hanuman Chalisa pin karein", "Kal ka ritual dikhao"
- Muslim: "Aaj ki Fajr timing?", "Ramadan mein sehri kab hai?", "Quran pace set karein"
- Sikh: "Aaj ka Hukamnama?", "Gurpurab kab hai?"
- Jain: "Paryushana mein kya khaana allowed hai?", "Navkar Mantra add karein"
- Christian: "Lent kab shuru hota hai?", "Aaj ka liturgical day?"

**Response pattern:** User types/taps chip → message appears in chat area above input → skeleton shimmer (1.2s) → streaming text response with rich card (title, body, action button where relevant).

**Camera input:** Tapping camera icon shows "Puja samagri identify karein" placeholder flow.

---

## Technical Constraints

- Single HTML file (no build step, no framework)
- JioType font from CDN
- HelloJio state videos from CDN (idle / listening / thinking / speaking)
- JDS tokens as CSS custom properties throughout
- Sacred Mode uses CSS class toggle on `<body>` (no separate page)
- All 5 faith paths share same HTML, JS switches content via `currentFaith` state variable
- Onboarding state persisted in `localStorage` so refreshing the page returns to Home
- Pre-canned responses in a JS object keyed by query substring

---

## What Is NOT in This Prototype

- Real audio playback (play button triggers Sacred Mode UI only)
- Real Panchang API (dates/tithi are hardcoded to 2026-03-04)
- Real AI / LLM calls
- Device calendar sync
- Garba / Taraweeh event maps (placeholder cards only)
- Multi-user profiles
- Phase 2 features (Japa counter, voice commands, commerce)
